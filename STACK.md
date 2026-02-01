# SearcherHub Tech Stack

## Overview
A modern React SPA with real-time backend using Convex and BetterAuth for authentication.

## Core Stack

### Frontend
- **React 18.3** - UI library
- **TypeScript** - Type safety
- **Vite 6.4** - Build tool and dev server
- **TailwindCSS 4.1** - Styling
- **Lucide React** - Icon library
- **Recharts** - Data visualization

### Backend & Database
- **Convex** - Real-time backend and database
  - Version: 1.18.0
  - WebSocket-based queries/mutations
  - Serverless functions
  - File storage
  - HTTP actions for webhooks

### Authentication
- **BetterAuth 1.4.9** - Authentication library
- **@convex-dev/better-auth 0.10.10** - Convex integration
- **Features:**
  - Email/Password authentication
  - Google OAuth (configured)
  - Database sessions
  - Cross-domain support for SPAs
  - JWT tokens

### Additional Services
- **Polar.sh** - Payment and subscription management

## Project Structure

```
searcherhub/
├── src/
│   ├── components/        # React components
│   │   ├── AuthForm.tsx   # Authentication modal
│   │   ├── Layout.tsx     # Main layout with nav
│   │   └── Landing.tsx    # Landing page
│   ├── contexts/
│   │   └── AuthContext.tsx # Auth state management
│   ├── lib/
│   │   └── auth-client.ts  # BetterAuth client config
│   ├── App.tsx            # Routes and protected routes
│   └── index.tsx          # Entry point
├── convex/
│   ├── convex.config.ts   # Component registration
│   ├── auth.config.ts     # Auth provider config
│   ├── auth.ts            # BetterAuth server setup
│   ├── http.ts            # HTTP routes (auth + webhooks)
│   ├── schema.ts          # Database schema
│   ├── users.ts           # User queries/mutations
│   └── webhooks.ts        # Webhook handlers
├── .env.local             # Local environment variables
└── .env.example           # Environment template
```

## Key Configuration Files

### 1. Environment Variables
```bash
# Convex
CONVEX_DEPLOYMENT=dev:your-deployment
VITE_CONVEX_URL=https://your-deployment.convex.cloud
VITE_CONVEX_SITE_URL=https://your-deployment.convex.site

# Site URLs
SITE_URL=http://localhost:5173
VITE_SITE_URL=http://localhost:5173

# BetterAuth (set in Convex deployment)
BETTER_AUTH_SECRET=<generated-secret>

# Google OAuth (optional)
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# Other services
GEMINI_API_KEY=
POLAR_ACCESS_TOKEN=
```

### 2. BetterAuth Setup (`convex/auth.ts`)
```typescript
import { createClient, type GenericCtx } from "@convex-dev/better-auth";
import { convex, crossDomain } from "@convex-dev/better-auth/plugins";
import { betterAuth } from "better-auth/minimal";
import authConfig from "./auth.config";

const siteUrl = process.env.SITE_URL || "http://localhost:5173";

export const authComponent = createClient<DataModel>(components.betterAuth);

export const createAuth = (ctx: GenericCtx<DataModel>) => {
  return betterAuth({
    baseURL: siteUrl,
    trustedOrigins: [siteUrl], // Critical for CORS
    database: authComponent.adapter(ctx),
    emailAndPassword: {
      enabled: true,
      requireEmailVerification: false,
    },
    socialProviders: {
      google: {
        clientId: process.env.GOOGLE_CLIENT_ID || "",
        clientSecret: process.env.GOOGLE_CLIENT_SECRET || "",
        enabled: true,
      },
    },
    plugins: [
      convex({ authConfig }),
      crossDomain({ siteUrl }), // Critical for SPAs
    ],
  });
};
```

### 3. HTTP Routes (`convex/http.ts`)
```typescript
import { httpRouter } from "convex/server";
import { authComponent, createAuth } from "./auth";

const http = httpRouter();

// CORS required for client-side frameworks
authComponent.registerRoutes(http, createAuth, { cors: true });

// Add other routes (webhooks, etc.)
http.route({
  path: "/webhooks/polar",
  method: "POST",
  handler: polarWebhook,
});

export default http;
```

### 4. Auth Client (`lib/auth-client.ts`)
```typescript
import { createAuthClient } from "better-auth/react";
import { convexClient, crossDomainClient } from "@convex-dev/better-auth/client/plugins";

const convexSiteUrl = import.meta.env.VITE_CONVEX_SITE_URL as string;

export const authClient = createAuthClient({
  baseURL: convexSiteUrl, // Use .site domain, not .cloud
  plugins: [
    convexClient(),
    crossDomainClient(), // Required for SPAs
  ],
});

export const { signIn, signUp, signOut, useSession, $Infer } = authClient;
```

### 5. Entry Point (`index.tsx`)
```typescript
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

## Setup Instructions

### 1. Initialize Project
```bash
npm create vite@latest project-name -- --template react-ts
cd project-name
```

### 2. Install Dependencies
```bash
# Core
npm install react react-dom
npm install -D typescript @types/react @types/react-dom

# Convex
npm install convex

# BetterAuth
npm install better-auth@1.4.9 @convex-dev/better-auth@0.10.10 jose --legacy-peer-deps

# UI
npm install tailwindcss@next @tailwindcss/vite@next
npm install lucide-react recharts
npm install react-router-dom
```

### 3. Initialize Convex
```bash
npx convex dev
```

### 4. Setup BetterAuth Component
```bash
# Register component in convex.config.ts
# Add auth.config.ts for auth provider
# Create auth.ts with BetterAuth configuration
```

### 5. Set Environment Variables
```bash
# In Convex deployment
npx convex env set BETTER_AUTH_SECRET=$(openssl rand -base64 32)
npx convex env set SITE_URL=http://localhost:5173

# In .env.local (auto-generated by convex dev)
# Add VITE_CONVEX_SITE_URL=https://your-deployment.convex.site
# Add VITE_SITE_URL=http://localhost:5173
```

### 6. Run Development Servers
```bash
# Terminal 1: Convex backend
npx convex dev

# Terminal 2: Vite frontend
npm run dev
```

## Critical Configuration Notes

### For React SPAs with Convex + BetterAuth:

1. **Use `.site` domain in auth client**, not `.cloud`:
   ```typescript
   baseURL: import.meta.env.VITE_CONVEX_SITE_URL // .site, not .cloud
   ```

2. **Add `crossDomain` plugin** on both server and client:
   ```typescript
   // Server: convex/auth.ts
   plugins: [convex({ authConfig }), crossDomain({ siteUrl })]
   
   // Client: lib/auth-client.ts
   plugins: [convexClient(), crossDomainClient()]
   ```

3. **Set `trustedOrigins`** in BetterAuth config:
   ```typescript
   trustedOrigins: [siteUrl]
   ```

4. **Enable CORS** in route registration:
   ```typescript
   authComponent.registerRoutes(http, createAuth, { cors: true })
   ```

5. **Set both URL environment variables**:
   - `VITE_CONVEX_URL` (for Convex client)
   - `VITE_CONVEX_SITE_URL` (for auth client)

## Package Versions (Tested & Working)

```json
{
  "dependencies": {
    "better-auth": "1.4.9",
    "@convex-dev/better-auth": "0.10.10",
    "convex": "1.18.0",
    "jose": "^5.9.6",
    "lucide-react": "^0.468.0",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-router-dom": "^7.1.3",
    "recharts": "^2.15.0"
  },
  "devDependencies": {
    "@tailwindcss/vite": "^4.1.7",
    "@types/react": "^18.3.18",
    "@types/react-dom": "^18.3.5",
    "@vitejs/plugin-react": "^4.3.4",
    "tailwindcss": "^4.1.7",
    "typescript": "~5.6.2",
    "vite": "^6.4.1"
  }
}
```

## Common Issues & Solutions

### Issue: 404 on `/api/auth/*` routes
**Solution:** Add `crossDomain` plugin and `trustedOrigins` to BetterAuth server config

### Issue: CORS errors from browser
**Solution:** 
1. Use `.site` domain in auth client
2. Add `crossDomainClient()` plugin
3. Ensure `{ cors: true }` in `registerRoutes()`
4. Set `trustedOrigins: [siteUrl]`

### Issue: Dependencies conflict
**Solution:** Use `--legacy-peer-deps` when installing BetterAuth packages

### Issue: Session not persisting
**Solution:** Ensure using `convexClient()` plugin in auth client

## Deployment Checklist

- [ ] Set production `SITE_URL` in Convex deployment
- [ ] Set production `BETTER_AUTH_SECRET`
- [ ] Configure OAuth redirect URIs (use `.site` domain)
- [ ] Update CORS `trustedOrigins` with production domain
- [ ] Update environment variables in hosting platform
- [ ] Test authentication flows in production

## Resources

- [Convex Docs](https://docs.convex.dev)
- [BetterAuth Docs](https://www.better-auth.com)
- [Convex + BetterAuth Guide](https://labs.convex.dev/better-auth)
- [React Router Docs](https://reactrouter.com)
- [Vite Docs](https://vite.dev)
