# Quick Start Template
## Convex + BetterAuth + React + TypeScript

Use this as a reference when starting new projects with this stack.

## 1. Create New Project

```bash
# Create Vite React TypeScript project
npm create vite@latest my-project -- --template react-ts
cd my-project

# Install core dependencies
npm install react react-dom
npm install -D typescript @types/react @types/react-dom

# Install Convex
npm install convex

# Install BetterAuth (with legacy peer deps)
npm install better-auth@1.4.9 @convex-dev/better-auth@0.10.10 jose --legacy-peer-deps

# Install UI libraries
npm install tailwindcss@next @tailwindcss/vite@next
npm install lucide-react recharts react-router-dom

# Initialize Convex
npx convex dev
```

## 2. Create Convex Configuration Files

### `convex/convex.config.ts`
```typescript
import { defineApp } from "convex/server";
import betterAuth from "@convex-dev/better-auth/convex.config";

const app = defineApp();
app.use(betterAuth);

export default app;
```

### `convex/auth.config.ts`
```typescript
import { getAuthConfigProvider } from "@convex-dev/better-auth/auth-config";
import type { AuthConfig } from "convex/server";

export default {
  providers: [getAuthConfigProvider()],
} satisfies AuthConfig;
```

### `convex/auth.ts`
```typescript
import { createClient, type GenericCtx } from "@convex-dev/better-auth";
import { convex, crossDomain } from "@convex-dev/better-auth/plugins";
import { components } from "./_generated/api";
import { DataModel } from "./_generated/dataModel";
import { betterAuth } from "better-auth/minimal";
import authConfig from "./auth.config";

const siteUrl = process.env.SITE_URL || "http://localhost:5173";

export const authComponent = createClient<DataModel>(components.betterAuth);

export const createAuth = (ctx: GenericCtx<DataModel>) => {
  return betterAuth({
    baseURL: siteUrl,
    trustedOrigins: [siteUrl],
    database: authComponent.adapter(ctx),
    emailAndPassword: {
      enabled: true,
      requireEmailVerification: false,
    },
    socialProviders: {
      google: {
        clientId: process.env.GOOGLE_CLIENT_ID || "",
        clientSecret: process.env.GOOGLE_CLIENT_SECRET || "",
        enabled: !!process.env.GOOGLE_CLIENT_ID,
      },
    },
    plugins: [
      convex({ authConfig }),
      crossDomain({ siteUrl }),
    ],
  });
};
```

### `convex/http.ts`
```typescript
import { httpRouter } from "convex/server";
import { authComponent, createAuth } from "./auth";

const http = httpRouter();

authComponent.registerRoutes(http, createAuth, { cors: true });

export default http;
```

### `convex/schema.ts`
```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  // Add your tables here
  // BetterAuth manages its own tables via the component
});
```

## 3. Create Auth Client Files

### `lib/auth-client.ts`
```typescript
import { createAuthClient } from "better-auth/react";
import { convexClient, crossDomainClient } from "@convex-dev/better-auth/client/plugins";

const convexSiteUrl = import.meta.env.VITE_CONVEX_SITE_URL as string;

export const authClient = createAuthClient({
  baseURL: convexSiteUrl,
  plugins: [
    convexClient(),
    crossDomainClient(),
  ],
});

export const { signIn, signUp, signOut, useSession, $Infer } = authClient;
```

### `contexts/AuthContext.tsx`
```typescript
import { createContext, useContext, type ReactNode } from "react";
import { useSession } from "@/lib/auth-client";

interface AuthContextType {
  isAuthenticated: boolean;
  isLoading: boolean;
  user: any | null;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const { data: session, isPending } = useSession();
  
  const value: AuthContextType = {
    isAuthenticated: !!session?.user,
    isLoading: isPending,
    user: session?.user ?? null,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth must be used within AuthProvider");
  }
  return context;
}
```

## 4. Setup App Entry Point

### `src/index.tsx`
```typescript
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./index.css";
import { ConvexProvider, ConvexReactClient } from "convex/react";
import { AuthProvider } from "./contexts/AuthContext";

const convex = new ConvexReactClient(
  import.meta.env.VITE_CONVEX_URL as string
);

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <ConvexProvider client={convex}>
      <AuthProvider>
        <App />
      </AuthProvider>
    </ConvexProvider>
  </React.StrictMode>
);
```

### `src/App.tsx` (with protected routes)
```typescript
import { BrowserRouter, Routes, Route, Navigate } from "react-router-dom";
import { useAuth } from "./contexts/AuthContext";
import Landing from "./components/Landing";
import Dashboard from "./components/Dashboard";

function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) {
    return <div>Loading...</div>;
  }

  if (!isAuthenticated) {
    return <Navigate to="/" replace />;
  }

  return <>{children}</>;
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Landing />} />
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute>
              <Dashboard />
            </ProtectedRoute>
          }
        />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

## 5. Create Basic Auth Form Component

### `components/AuthForm.tsx`
```typescript
import { useState } from "react";
import { signIn, signUp } from "@/lib/auth-client";

export default function AuthForm({ onClose }: { onClose: () => void }) {
  const [isSignUp, setIsSignUp] = useState(false);
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");

    try {
      if (isSignUp) {
        await signUp.email({
          email,
          password,
          name: email.split("@")[0],
        });
      } else {
        await signIn.email({
          email,
          password,
        });
      }
      onClose();
    } catch (err: any) {
      setError(err.message || "Authentication failed");
    }
  };

  return (
    <div className="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
      <div className="bg-white p-8 rounded-lg max-w-md w-full">
        <h2 className="text-2xl font-bold mb-4">
          {isSignUp ? "Sign Up" : "Sign In"}
        </h2>
        
        {error && (
          <div className="bg-red-50 text-red-600 p-3 rounded mb-4">
            {error}
          </div>
        )}

        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <label className="block text-sm font-medium mb-1">Email</label>
            <input
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              className="w-full px-3 py-2 border rounded"
              required
            />
          </div>

          <div>
            <label className="block text-sm font-medium mb-1">Password</label>
            <input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              className="w-full px-3 py-2 border rounded"
              required
              minLength={8}
            />
          </div>

          <button
            type="submit"
            className="w-full bg-blue-600 text-white py-2 rounded hover:bg-blue-700"
          >
            {isSignUp ? "Create Account" : "Sign In"}
          </button>
        </form>

        <div className="mt-4 text-center">
          <button
            onClick={() => setIsSignUp(!isSignUp)}
            className="text-blue-600 hover:underline"
          >
            {isSignUp
              ? "Already have an account? Sign in"
              : "Need an account? Sign up"}
          </button>
        </div>

        <button
          onClick={onClose}
          className="absolute top-4 right-4 text-gray-500 hover:text-gray-700"
        >
          ✕
        </button>
      </div>
    </div>
  );
}
```

## 6. Environment Variables

### Set in Convex deployment:
```bash
npx convex env set BETTER_AUTH_SECRET=$(openssl rand -base64 32)
npx convex env set SITE_URL=http://localhost:5173
```

### Add to `.env.local`:
```bash
# Auto-generated by convex dev
CONVEX_DEPLOYMENT=dev:your-deployment
VITE_CONVEX_URL=https://your-deployment.convex.cloud

# Add these manually
VITE_CONVEX_SITE_URL=https://your-deployment.convex.site
VITE_SITE_URL=http://localhost:5173

# Optional: Google OAuth
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
```

### Create `.env.example`:
```bash
VITE_CONVEX_URL=
VITE_CONVEX_SITE_URL=
VITE_SITE_URL=http://localhost:5173
```

## 7. Configure TailwindCSS

### `vite.config.ts`
```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";
import path from "path";

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  server: {
    port: 5173,
  },
});
```

### `tailwind.config.js`
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

### `src/index.css`
```css
@import "tailwindcss";
```

## 8. TypeScript Configuration

### `tsconfig.json`
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

## 9. Run Development Servers

```bash
# Terminal 1: Convex backend
npx convex dev

# Terminal 2: Vite frontend  
npm run dev
```

## 10. Verify Setup

- [ ] Convex dev server running without errors
- [ ] Vite dev server running on port 5173
- [ ] Can access app at http://localhost:5173
- [ ] Auth form opens when clicking auth button
- [ ] Can create account with email/password
- [ ] Successfully redirects to dashboard after signup
- [ ] Session persists on page refresh
- [ ] Can sign out and sign back in

## Testing Checklist

```typescript
// Test authentication flow
✓ Sign up with new email
✓ Sign in with existing account
✓ Sign out
✓ Protected route redirects when not authenticated
✓ Protected route accessible when authenticated
✓ Session persists on page refresh
```

## Production Deployment

1. Deploy frontend to Vercel/Netlify/etc.
2. Update Convex environment variables:
   ```bash
   npx convex env set SITE_URL=https://your-domain.com --prod
   npx convex env set BETTER_AUTH_SECRET=$(openssl rand -base64 32) --prod
   ```
3. Update `trustedOrigins` to include production domain
4. Configure OAuth redirect URIs with `.site` domain
5. Test auth flows in production

## Additional Features to Add

- Email verification
- Password reset flow
- Multi-factor authentication
- Social OAuth (GitHub, Twitter, etc.)
- User profile management
- Role-based access control
- Session management UI

## Troubleshooting

If you encounter issues, refer to `STACK.md` for common problems and solutions.
