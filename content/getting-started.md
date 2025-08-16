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
