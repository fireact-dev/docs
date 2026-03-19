---
title: "AI-Assisted Development with Claude Code"
linkTitle: "Claude Code"
weight: 1
description: >
  Use natural language to customize your Fireact app with the built-in Claude Code skill.
---

Every Fireact project scaffolded with `create-fireact-app` includes a built-in [Claude Code](https://claude.com/claude-code) skill that helps you add features, pages, components, and more using natural language. The skill understands Fireact's architecture and makes changes across all the right files — components, routes, config, i18n, security rules — so you don't have to remember the wiring.

### Getting Started

1. Open your Fireact project directory in Claude Code
2. The skill auto-detects Fireact projects (by checking for `@fireact.dev/app` in `package.json`)
3. Describe what you want, or type `/fireact-builder` to invoke the skill explicitly

### What You Can Do

The skill includes 8 customization playbooks:

| Playbook | Example Prompt |
|----------|---------------|
| **Add Subscription Page** | "Add a reports page to my subscription" |
| **Add Authenticated Page** | "Add an API keys page" |
| **Add Public Page** | "Add a landing page" |
| **Replace/Customize Component** | "Customize the sign-in page" |
| **Customize Navigation** | "Add a reports link to the sidebar" |
| **Customize Branding & Theme** | "Change the nav bar color to blue" |
| **Add Cloud Functions** | "Add a cloud function that sends a welcome email" |
| **Add Firestore Collections** | "Add a notes collection to subscriptions" |

### How It Works

When you ask the skill to make a change, it follows a structured process:

1. **Reads your project state** — examines `App.tsx`, `app.config.json`, `stripe.config.json`, `en.ts`, existing components, Cloud Functions, and Firestore rules to understand your current setup
2. **Follows the appropriate playbook** — each type of change has a defined sequence of steps that ensures nothing is missed
3. **Makes changes across all affected files** — for example, adding a subscription page involves creating the component, adding the route to config, adding the route to `App.tsx`, and adding i18n keys

### Example: Adding a Subscription Page

When you say "add a reports page", the skill will:

1. Create `src/components/Reports.tsx` with proper hooks (`useSubscription`, `useConfig`, `useTranslation`), loading/error handling, and TailwindCSS styling
2. Add `"reports": "/subscription/:id/reports"` to `src/config/app.config.json`
3. Add a `<Route>` inside the `SubscriptionProvider` block in `src/App.tsx` with `<ProtectedSubscriptionRoute>`
4. Add the component import to `src/App.tsx`
5. Add translation keys to `src/i18n/en.ts`

### Reference Documentation

The skill ships with detailed reference docs in `.claude/skills/fireact-builder/references/`:

| File | Contents |
|------|----------|
| `hooks-and-contexts-api.md` | All hooks (`useConfig`, `useSubscription`, `useAuth`, `useSubscriptionInvoices`), return types, and exported TypeScript types |
| `routing-patterns.md` | Three route groups, `ProtectedSubscriptionRoute`, config mapping |
| `component-patterns.md` | Templates for subscription, authenticated, public, and Firestore-connected components |
| `navigation-customization.md` | Menu patterns, `SubscriptionLayout` props, sidebar width classes |
| `cloud-functions-patterns.md` | Cloud Function templates, `global.saasConfig`, frontend calling, Firestore rules |

### Conventions the Skill Follows

The skill enforces Fireact best practices automatically:

- Uses `useTranslation()` with `t()` for all user-facing strings
- Handles loading and error states in subscription components
- Uses TailwindCSS exclusively (no inline styles)
- Adds route keys to `app.config.json` for every new page
- Wraps subscription routes in `<ProtectedSubscriptionRoute>`
- Follows the `/subscription/:id/<slug>` URL pattern for subscription pages
- Verifies changes compile with `npm run build`

### Requirements

- [Claude Code](https://claude.com/claude-code) CLI installed
- A Fireact project created with `create-fireact-app`
