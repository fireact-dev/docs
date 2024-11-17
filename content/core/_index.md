---
title: "Core Package"
linkTitle: "Core"
weight: 2
description: >
  Essential authentication and user management features
---

## Overview

The @fireact.dev/core package provides essential features for web applications built with Firebase and React. It handles user authentication, profile management, and account settings with a complete set of pre-built components and utilities.

## Features

### Authentication System
- Email/password authentication
- Social login providers support
- Password reset functionality
- Protected route handling

### User Profile Management
- User profile display and editing
- Name and email management
- Password change functionality
- Account deletion options

### Account Settings
- User preferences
- Account security options
- Profile customization
- Email verification

### Development Support
- Firebase emulator integration
- TypeScript support
- Internationalization (i18n)
- Responsive UI components

## Components

### Authentication Components
- `<SignIn />` - Sign in form with social options
- `<SignUp />` - Registration form
- `<ResetPassword />` - Password reset form
- `<PrivateRoute />` - Protected route wrapper

### Profile Components
- `<Profile />` - User profile display/edit
- `<EditName />` - Name editor
- `<EditEmail />` - Email update
- `<ChangePassword />` - Password change
- `<DeleteAccount />` - Account deletion

### Layout Components
- `<AuthenticatedLayout />` - Layout for protected pages
- `<PublicLayout />` - Layout for public pages
- `<Message />` - Notification display
- `<Dashboard />` - User dashboard

### Context Providers
- `<AuthProvider />` - Authentication context
- `<ConfigProvider />` - Configuration context
- `<LoadingProvider />` - Loading state management

## Tech Stack

- React with TypeScript
- Firebase Authentication
- Firebase Firestore
- TailwindCSS
- i18next for internationalization

## Getting Started

Visit our [Getting Started Guide]({{< ref "/core/getting-started" >}}) to begin using the core package.
