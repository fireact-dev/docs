---
title: "Getting Started"
linkTitle: "Getting Started"
type: "docs"
no_list: true
weight: 5
cascade:
  type: "docs"
---

This guide will walk you through the process of setting up your first Fireact application.

### Prerequisites

Before you begin, ensure you have the following installed and configured:

1.  **Node.js (version 22 or higher)**: Fireact.dev uses Node.js for its development environment and Cloud Functions.
    *   You can download it from [nodejs.org](https://nodejs.org/).
    *   Verify your installation: `node -v`
2.  **npm (Node Package Manager)**: npm is installed automatically with Node.js.
    *   Verify your installation: `npm -v`
3.  **Firebase CLI**: The Firebase Command Line Interface is essential for interacting with your Firebase project, deploying functions, and running emulators.
    *   Install it globally: `npm install -g firebase-tools`
    *   Log in to Firebase: `firebase login`
4.  **Stripe CLI**: If you plan to use Stripe for subscriptions, the Stripe CLI is necessary for testing webhooks locally.
    *   Follow the installation instructions on the [Stripe documentation](https://stripe.com/docs/stripe-cli).
    *   Log in to Stripe CLI: `stripe login`
5.  **Firebase Project**: You need an existing Firebase project.
    *   Create one at [Firebase Console](https://console.firebase.google.com/).
    *   Ensure you have at least one **Web App** configured within your Firebase project. Fireact.dev will fetch the SDK configuration for this web app during setup.

### Creating a New Fireact Application

To create a new Fireact application, use the `create-fireact-app` CLI tool. This tool will guide you through the setup process, including Firebase and Stripe configurations.

1.  **Install the CLI (if you haven't already)**:
    ```bash
    npm install -g create-fireact-app
    ```

2.  **Create a new project**:
    ```bash
    create-fireact-app <your-project-name>
    ```
    Replace `<your-project-name>` with the desired name for your new application.

3.  **Follow the prompts**: The CLI will guide you through selecting your Firebase project and configuring Stripe.

### After Creation: Running Your Application

Once the `create-fireact-app` command completes, follow these steps to run your new Fireact application locally:

1.  **Navigate to your project directory**:
    ```bash
    cd <your-project-name>
    ```

2.  **Build Cloud Functions**:
    ```bash
    cd functions && npm run build
    ```
    This compiles your Firebase Cloud Functions from TypeScript to JavaScript.

3.  **Build the React Application**:
    ```bash
    cd .. && npm run build
    ```
    This builds the React frontend application.

4.  **Start Firebase Emulators**:
    ```bash
    firebase emulators:start
    ```
    This command starts the Firebase emulators for Authentication, Firestore, and Cloud Functions. Your frontend application will also be served on `http://localhost:5002`.

5.  **Set up Stripe Webhook (in a separate terminal)**:
    If you are using Stripe, you need to forward Stripe events to your local Cloud Functions emulator. Open a **new terminal** and run:
    ```bash
    stripe listen --forward-to http://127.0.0.1:5001/<your-firebase-project-id>/us-central1/stripeWebhook
    ```
    *Replace `<your-firebase-project-id>` with your actual Firebase project ID.*

6.  **Update Stripe Webhook Secret**:
    The `stripe listen` command will output a new webhook endpoint secret (e.g., `whsec_...`). You **must** copy this secret and update it in your functions configuration:
    *   Open `functions/src/config/stripe.config.json` in your project.
    *   Replace the existing `endpointSecret` value with the new one.

7.  **Rebuild Cloud Functions (after updating secret)**:
    ```bash
    cd functions && npm run build
    ```
    This ensures your Cloud Functions use the updated Stripe webhook secret.

8.  **Access your application**:
    Your Fireact application will be available in your browser at `http://localhost:5002`.

### Next Steps: Customizing Your SaaS Application

The `create-fireact-app` CLI provides a solid foundation with essential SaaS functionalities like authentication, user management, and subscription handling. However, the generated application is a starting point. To build the specific features and unique user experience for your SaaS product, you will need to engage in custom development.

The **[Custom Development](/custom-development/)** section of this documentation provides comprehensive guides on how to:

*   Develop your own React components and pages.
*   Understand and manage data fetching within the framework.
*   Implement and extend localization capabilities.
*   Configure routing and enforce permission control.

By leveraging the information in the Custom Development section, you can effectively extend the Fireact.dev framework to build the unique features of your SaaS application.

## Troubleshooting

### Common Setup Issues

#### CLI Command Not Found

If `create-fireact-app` is not recognized after installation:

```bash
# Check global npm path
npm config get prefix

# Use npx instead (no installation needed)
npx create-fireact-app my-app
```

#### Firebase Login Issues

If `firebase login` fails or hangs:

```bash
# Try CI mode (opens browser authentication)
firebase login --no-localhost

# Or clear cache and retry
rm -rf ~/.config/firebase
firebase login
```

#### Node.js Version Error

Fireact.dev requires Node.js v18 or higher:

```bash
# Check your version
node -v

# Upgrade using nvm (recommended)
nvm install 18
nvm use 18
```

### Build and Emulator Issues

#### Emulators Won't Start

If Firebase emulators fail to start:

1. **Check port availability**:
   ```bash
   # Common ports used by emulators
   lsof -i :5173  # Vite dev server
   lsof -i :9099  # Auth emulator
   lsof -i :8080  # Firestore emulator
   lsof -i :5001  # Functions emulator
   lsof -i :5002  # Hosting emulator
   ```

2. **Ensure functions are built**:
   ```bash
   cd functions
   npm run build
   cd ..
   ```

3. **Check Java installation** (required for Firestore emulator):
   ```bash
   java -version
   # Install Java if missing
   ```

#### Build Errors

If you encounter build errors:

```bash
# Clean and rebuild
rm -rf node_modules dist
npm install
npm run build

# For functions
cd functions
rm -rf node_modules lib
npm install
npm run build
cd ..
```

### Stripe Integration Issues

#### Webhook Secret Mismatch

If Stripe webhooks fail with signature verification errors:

1. Get new webhook secret from Stripe CLI:
   ```bash
   stripe listen --forward-to http://127.0.0.1:5001/YOUR_PROJECT_ID/us-central1/stripeWebhook
   ```

2. Copy the webhook signing secret (starts with `whsec_`)

3. Update `functions/src/config/stripe.config.json`:
   ```json
   {
     "endpointSecret": "whsec_YOUR_NEW_SECRET_HERE"
   }
   ```

4. Rebuild functions:
   ```bash
   cd functions && npm run build && cd ..
   ```

5. Restart emulators

#### Test Card Not Working

Use these Stripe test card numbers:

- **Success**: `4242 4242 4242 4242`
- **Requires Authentication**: `4000 0025 0000 3155`
- **Declined**: `4000 0000 0000 9995`

**For all test cards:**
- Expiry: Any future date
- CVC: Any 3 digits
- ZIP: Any valid ZIP code

### Authentication Issues

#### User Not Redirected After Sign-In

If the user is stuck on the sign-in page after authentication:

1. Clear browser cache (Cmd+Shift+R or Ctrl+Shift+R)
2. Check browser console for errors
3. Verify routing configuration in your app
4. Ensure emulators are running

#### Password Reset Email Not Received

In **emulator mode**:
- Emails are not sent
- Check the Auth emulator UI at http://localhost:4000
- Look for reset links in the Auth tab

In **production**:
- Check spam folder
- Verify email templates in Firebase Console
- Check email quota limits

### Firestore Issues

#### Permission Denied Errors

If you see "Permission denied" errors when accessing Firestore:

1. **In emulator mode**: Check `firestore.rules` file
2. **Restart emulators** after rule changes
3. **Verify user is authenticated**
4. **Check Firestore rules** in Firebase Console (production)

#### Data Not Appearing

If data doesn't appear in your application:

1. Check Firestore emulator UI: http://localhost:4000
2. Verify collection and document names
3. Check query filters and where clauses
4. Look for errors in browser console

### Getting Help

If you're still experiencing issues:

1. **Check the comprehensive troubleshooting guide**: [Troubleshooting Documentation](https://github.com/fireact-dev/fireact.dev/blob/main/TROUBLESHOOTING.md)

2. **Search existing issues**: [GitHub Issues](https://github.com/fireact-dev/fireact.dev/issues)

3. **Ask the community**: GitHub Discussions

4. **Report a bug**: Create a new issue with:
   - Clear description of the problem
   - Steps to reproduce
   - Expected vs actual behavior
   - Your environment (OS, Node version, browser)
   - Error messages and logs

### Quick Reference

```bash
# Common commands
create-fireact-app my-app          # Create new app
firebase emulators:start           # Start emulators
firebase emulators:start --export-on-exit  # Save data on exit
stripe listen --forward-to ...     # Start Stripe webhooks
npm run build                      # Build React app
cd functions && npm run build      # Build functions

# Troubleshooting commands
firebase --version                 # Check Firebase CLI version
node --version                     # Check Node version
npm cache clean --force           # Clear npm cache
rm -rf node_modules               # Remove dependencies
git submodule update --init       # Initialize submodules
```

## What's Next?

Congratulations! You now have a working Fireact.dev application. Here's your recommended learning path:

### 1. Build Your First Feature (Recommended)
Start with the **[Building Your First Custom Feature](/tutorials/building-first-feature/)** tutorial. This hands-on guide walks you through creating a complete feature from scratch, including:
- Frontend components with React and TailwindCSS
- Cloud Functions for backend logic
- Firestore database integration
- User permissions and access control

**Time**: ~45 minutes | **Difficulty**: Beginner

### 2. Customize the Look and Feel
Follow **[Customizing the UI Theme](/tutorials/customizing-ui-theme/)** to brand your application:
- Configure TailwindCSS colors and fonts
- Implement dark mode
- Create custom components
- Apply your brand identity

**Time**: ~30 minutes | **Difficulty**: Beginner

### 3. Explore Code Examples
Browse the **[Examples](/examples/)** section for ready-to-use code snippets:
- [Authentication Examples](/examples/authentication-examples/) - Social login, email verification
- [Data Management Examples](/examples/data-management-examples/) - CRUD operations, pagination
- [UI Components Examples](/examples/ui-components-examples/) - Forms, modals, tables
- [Subscription Examples](/examples/subscription-examples/) - Payment flows, billing

### 4. Advanced Topics
Once comfortable with the basics, explore advanced tutorials:
- **[Custom Subscription Plans](/tutorials/custom-subscription-plans/)** - Implement tiered pricing, usage-based billing, and add-ons
- **[Third-Party Integrations](/tutorials/third-party-integrations/)** - Integrate SendGrid, Google Analytics, external APIs

### 5. Learn Custom Development Patterns
Deep dive into **[Custom Development](/custom-development/)** to understand:
- Application architecture and routing
- Data fetching patterns
- Localization and i18n
- Permission control systems

### 6. Production Readiness
Before deploying to production, review:
- **[Best Practices](/best-practices/)** - Code organization, performance, error handling, testing
- **[Security Guide](https://github.com/fireact-dev/fireact.dev/blob/main/SECURITY.md)** - Authentication, authorization, data protection
- **[Deployment Guide](https://github.com/fireact-dev/fireact.dev/blob/main/DEPLOYMENT.md)** - Deploy to Firebase, Vercel, AWS, or Docker

## Additional Resources

- **[API Reference](/app/)** - Complete documentation of all components and hooks
- **[Functions Reference](/functions/)** - Cloud Functions API documentation
- **[Architecture Overview](https://github.com/fireact-dev/fireact.dev/blob/main/ARCHITECTURE.md)** - System design and architecture
- **[Contributing Guide](https://github.com/fireact-dev/fireact.dev/blob/main/CONTRIBUTING.md)** - Contribute to Fireact.dev
- **[GitHub Discussions](https://github.com/fireact-dev/fireact.dev/discussions)** - Ask questions and share ideas

## Quick Tips for Success

1. **Start Small**: Begin with simple features before tackling complex ones
2. **Use Examples**: Copy and adapt code from the Examples section
3. **Test Locally**: Always test with emulators before deploying
4. **Read Error Messages**: Firebase and React provide helpful error messages
5. **Check the Console**: Browser developer console shows runtime errors
6. **Version Control**: Commit your changes frequently
7. **Join the Community**: Don't hesitate to ask questions in GitHub Discussions

Happy building! 🚀
