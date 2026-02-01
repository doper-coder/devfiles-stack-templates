# DevFiles Stack Templates

Reusable stack documentation and project templates for building modern web applications.

## 📚 Available Templates

### Convex + BetterAuth + React + TypeScript

A complete, production-ready stack for building real-time applications with authentication.

**Stack includes:**
- **Frontend:** React 18 + TypeScript + Vite + TailwindCSS
- **Backend:** Convex (real-time serverless backend)
- **Auth:** BetterAuth with Convex integration
- **Features:** Email/Password auth, Google OAuth, protected routes, session management

**Documentation:**
- **[STACK.md](./STACK.md)** - Complete technical documentation with all configurations
- **[PROJECT_TEMPLATE.md](./PROJECT_TEMPLATE.md)** - Step-by-step quick start guide

## 🚀 Quick Start

```bash
# 1. Create new Vite React TypeScript project
npm create vite@latest my-app -- --template react-ts
cd my-app

# 2. Install dependencies
npm install convex
npm install better-auth@1.4.9 @convex-dev/better-auth@0.10.10 jose --legacy-peer-deps
npm install tailwindcss@next @tailwindcss/vite@next
npm install lucide-react recharts react-router-dom

# 3. Follow PROJECT_TEMPLATE.md for complete setup
```

## ✨ Why This Stack?

- **Real-time by default** - Convex provides WebSocket-based queries with automatic reactivity
- **Type-safe** - End-to-end TypeScript across frontend, backend, and database
- **Modern auth** - BetterAuth with database sessions, OAuth, and cross-domain support
- **Developer experience** - Hot reload, automatic type generation, and instant deployments
- **Production ready** - Battle-tested configurations with proper CORS, security, and error handling

## 🔑 Key Features

✅ Email/Password authentication  
✅ Google OAuth (easily extend to other providers)  
✅ Protected routes  
✅ Session persistence  
✅ Cross-domain support for SPAs  
✅ Real-time database queries  
✅ Serverless HTTP actions  
✅ File storage capabilities  
✅ Type-safe database schema  

## 📖 Documentation

### STACK.md
Complete technical reference including:
- Full stack overview
- Project structure
- All configuration files with code
- Critical setup notes
- Common issues and solutions
- Deployment checklist

### PROJECT_TEMPLATE.md
Step-by-step guide with:
- Installation commands
- All code files needed
- Environment setup
- Testing checklist
- Production deployment steps

## 🛠️ Technologies

| Category | Technology | Version |
|----------|-----------|---------|
| Frontend Framework | React | 18.3 |
| Language | TypeScript | 5.6 |
| Build Tool | Vite | 6.4 |
| Styling | TailwindCSS | 4.1 |
| Backend | Convex | 1.18 |
| Authentication | BetterAuth | 1.4.9 |
| Auth Integration | @convex-dev/better-auth | 0.10.10 |
| Icons | Lucide React | Latest |
| Charts | Recharts | Latest |
| Routing | React Router | 7.1 |

## ⚠️ Critical Configuration Notes

When using Convex + BetterAuth in a React SPA, you **must**:

1. Use `.site` domain for auth client (not `.cloud`)
2. Add `crossDomain` plugin on both server and client
3. Set `trustedOrigins` in BetterAuth config
4. Enable CORS in route registration
5. Set both `VITE_CONVEX_URL` and `VITE_CONVEX_SITE_URL`

See STACK.md for detailed explanations.

## 🐛 Troubleshooting

Common issues are documented in STACK.md with solutions:
- 404 on `/api/auth/*` routes
- CORS errors
- Dependency conflicts
- Session persistence issues

## 🚢 Deployment

Ready to deploy? Check the deployment checklist in both documentation files.

**Recommended hosting:**
- Frontend: Vercel, Netlify, or Cloudflare Pages
- Backend: Convex (automatically deployed)

## 📝 License

MIT

## 🤝 Contributing

These templates are based on production experience. If you find issues or have improvements, feel free to open an issue or PR.

## 🔗 Resources

- [Convex Documentation](https://docs.convex.dev)
- [BetterAuth Documentation](https://www.better-auth.com)
- [Convex + BetterAuth Guide](https://labs.convex.dev/better-auth)
- [React Documentation](https://react.dev)
- [Vite Documentation](https://vite.dev)

---

**Created from the SearcherHub project** - A real-world application using this exact stack.
