---
title: "Getting Started"
linkTitle: "Getting Started"
weight: 2
description: >
  What you need to know to get started with Fireact
---

## Prerequisites

Before you begin, make sure you have the following installed:

- Node.js (version 14 or higher)
- npm or yarn
- Git

## Installation

You can create a new Fireact project using our CLI tool:

```bash
npx create-fireact-app my-app
cd my-app
```

Or clone the repository directly:

```bash
git clone https://github.com/username/fireact.git
cd fireact
npm install
```

## Configuration

1. Create a new Firebase project at [Firebase Console](https://console.firebase.google.com)
2. Copy your Firebase configuration
3. Create a `.env` file in your project root:

```env
REACT_APP_FIREBASE_API_KEY=your_api_key
REACT_APP_FIREBASE_AUTH_DOMAIN=your_auth_domain
REACT_APP_FIREBASE_PROJECT_ID=your_project_id
REACT_APP_FIREBASE_STORAGE_BUCKET=your_storage_bucket
REACT_APP_FIREBASE_MESSAGING_SENDER_ID=your_messaging_sender_id
REACT_APP_FIREBASE_APP_ID=your_app_id
```

## Running the Application

Start the development server:

```bash
npm start
```

Your application will be available at `http://localhost:3000`.

## Next Steps

- Explore the Core Concepts section to understand Fireact's architecture
- Check out our API documentation for detailed reference
- Read through our guides for specific use cases and examples
