---
title: "Testing Strategies"
linkTitle: "Testing"
type: "docs"
weight: 5
description: >
  Best practices for testing Fireact applications including unit, integration, and E2E tests.
---

## Overview

This guide covers comprehensive testing strategies for Fireact applications, including React component testing, Cloud Functions testing, Firestore security rules testing, and end-to-end testing.

## Testing Stack

- **Unit/Component Tests**: Vitest + React Testing Library
- **Cloud Functions Tests**: Jest + Firebase Test SDK
- **Security Rules Tests**: Firebase Emulator Suite
- **E2E Tests**: Playwright or Cypress
- **API Tests**: Supertest

## React Component Testing

### Setup Vitest

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'src/test/'],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach } from 'vitest';

afterEach(() => {
  cleanup();
});
```

### Component Testing Examples

#### Basic Component Test

```typescript
// src/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick handler when clicked', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByText('Click me')).toBeDisabled();
  });

  it('applies custom className', () => {
    render(<Button className="custom-class">Click me</Button>);
    expect(screen.getByText('Click me')).toHaveClass('custom-class');
  });
});
```

#### Testing with Context

```typescript
// src/contexts/__tests__/AuthContext.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { AuthProvider, useAuth } from '../AuthContext';
import { signInWithEmailAndPassword } from 'firebase/auth';

vi.mock('firebase/auth');

const TestComponent = () => {
  const { currentUser, signIn } = useAuth();
  return (
    <div>
      <div>{currentUser ? currentUser.email : 'Not logged in'}</div>
      <button onClick={() => signIn('test@example.com', 'password')}>
        Sign In
      </button>
    </div>
  );
};

describe('AuthContext', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('provides authentication state', async () => {
    const mockUser = { email: 'test@example.com', uid: '123' };
    vi.mocked(signInWithEmailAndPassword).mockResolvedValue({
      user: mockUser,
    } as any);

    render(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );

    expect(screen.getByText('Not logged in')).toBeInTheDocument();

    fireEvent.click(screen.getByText('Sign In'));

    await waitFor(() => {
      expect(screen.getByText('test@example.com')).toBeInTheDocument();
    });
  });
});
```

#### Testing Custom Hooks

```typescript
// src/hooks/__tests__/useSubscription.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { useSubscription } from '../useSubscription';
import { onSnapshot } from 'firebase/firestore';

vi.mock('firebase/firestore');

describe('useSubscription', () => {
  it('fetches subscription data', async () => {
    const mockSubscription = {
      id: 'sub_123',
      planId: 'pro',
      status: 'active',
    };

    vi.mocked(onSnapshot).mockImplementation((ref, onSuccess) => {
      onSuccess({
        exists: () => true,
        id: 'sub_123',
        data: () => mockSubscription,
      } as any);
      return vi.fn();
    });

    const { result } = renderHook(() => useSubscription('sub_123'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.subscription).toEqual({
      id: 'sub_123',
      ...mockSubscription,
    });
    expect(result.current.error).toBeNull();
  });

  it('handles errors', async () => {
    vi.mocked(onSnapshot).mockImplementation((ref, onSuccess, onError) => {
      onError?.(new Error('Permission denied') as any);
      return vi.fn();
    });

    const { result } = renderHook(() => useSubscription('sub_123'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.error).toBeTruthy();
    expect(result.current.subscription).toBeNull();
  });
});
```

#### Testing Forms

```typescript
// src/components/__tests__/LoginForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { LoginForm } from '../LoginForm';
import { signInWithEmailAndPassword } from 'firebase/auth';

vi.mock('firebase/auth');

describe('LoginForm', () => {
  it('validates email field', async () => {
    render(<LoginForm />);

    const submitButton = screen.getByText('Sign In');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByText('Email is required')).toBeInTheDocument();
    });
  });

  it('validates password field', async () => {
    render(<LoginForm />);

    const emailInput = screen.getByLabelText('Email');
    fireEvent.change(emailInput, { target: { value: 'test@example.com' } });

    const submitButton = screen.getByText('Sign In');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByText('Password is required')).toBeInTheDocument();
    });
  });

  it('submits form with valid data', async () => {
    const mockSignIn = vi.mocked(signInWithEmailAndPassword);
    mockSignIn.mockResolvedValue({ user: { uid: '123' } } as any);

    render(<LoginForm />);

    const emailInput = screen.getByLabelText('Email');
    const passwordInput = screen.getByLabelText('Password');

    fireEvent.change(emailInput, { target: { value: 'test@example.com' } });
    fireEvent.change(passwordInput, { target: { value: 'password123' } });

    const submitButton = screen.getByText('Sign In');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(mockSignIn).toHaveBeenCalledWith(
        expect.anything(),
        'test@example.com',
        'password123'
      );
    });
  });

  it('displays error on sign in failure', async () => {
    const mockSignIn = vi.mocked(signInWithEmailAndPassword);
    mockSignIn.mockRejectedValue({ code: 'auth/wrong-password' });

    render(<LoginForm />);

    const emailInput = screen.getByLabelText('Email');
    const passwordInput = screen.getByLabelText('Password');

    fireEvent.change(emailInput, { target: { value: 'test@example.com' } });
    fireEvent.change(passwordInput, { target: { value: 'wrongpassword' } });

    const submitButton = screen.getByText('Sign In');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByText('Incorrect password')).toBeInTheDocument();
    });
  });
});
```

## Cloud Functions Testing

### Setup Jest for Functions

```json
// functions/package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "devDependencies": {
    "@types/jest": "^29.5.0",
    "jest": "^29.5.0",
    "ts-jest": "^29.1.0"
  }
}
```

```javascript
// functions/jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  testMatch: ['**/__tests__/**/*.test.ts'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/index.ts',
  ],
};
```

### Testing Cloud Functions

```typescript
// functions/src/functions/__tests__/createSubscription.test.ts
import { createSubscription } from '../createSubscription';
import * as admin from 'firebase-admin';
import Stripe from 'stripe';

jest.mock('stripe');
jest.mock('firebase-admin', () => ({
  firestore: jest.fn(() => ({
    collection: jest.fn(() => ({
      doc: jest.fn(() => ({
        set: jest.fn(),
        get: jest.fn(),
      })),
    })),
  })),
}));

describe('createSubscription', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('creates subscription successfully', async () => {
    const mockStripe = {
      customers: {
        create: jest.fn().mockResolvedValue({ id: 'cus_123' }),
      },
      subscriptions: {
        create: jest.fn().mockResolvedValue({
          id: 'sub_123',
          status: 'active',
          current_period_start: 1234567890,
          current_period_end: 1234567890,
        }),
      },
    };

    (Stripe as any).mockImplementation(() => mockStripe);

    const context = {
      auth: { uid: 'user_123', token: { email: 'test@example.com' } },
    };

    const data = {
      planId: 'pro',
      paymentMethodId: 'pm_123',
    };

    const result = await createSubscription(data, context as any);

    expect(result.success).toBe(true);
    expect(result.subscriptionId).toBe('sub_123');
    expect(mockStripe.customers.create).toHaveBeenCalled();
    expect(mockStripe.subscriptions.create).toHaveBeenCalled();
  });

  it('throws error if unauthenticated', async () => {
    const context = { auth: null };
    const data = { planId: 'pro' };

    await expect(
      createSubscription(data, context as any)
    ).rejects.toThrow('unauthenticated');
  });
});
```

### Testing with Firebase Emulators

```typescript
// functions/src/__tests__/integration.test.ts
import * as admin from 'firebase-admin';
import { getFunctions } from 'firebase-admin/functions';

// Initialize Firebase Admin with emulator
process.env.FIRESTORE_EMULATOR_HOST = 'localhost:8080';
process.env.FIREBASE_AUTH_EMULATOR_HOST = 'localhost:9099';

if (!admin.apps.length) {
  admin.initializeApp({ projectId: 'test-project' });
}

describe('Integration Tests', () => {
  const db = admin.firestore();

  beforeEach(async () => {
    // Clear Firestore
    const collections = await db.listCollections();
    for (const collection of collections) {
      const docs = await collection.listDocuments();
      for (const doc of docs) {
        await doc.delete();
      }
    }
  });

  it('creates user document on authentication', async () => {
    const user = await admin.auth().createUser({
      email: 'test@example.com',
      password: 'password123',
    });

    // Wait for trigger
    await new Promise((resolve) => setTimeout(resolve, 1000));

    const userDoc = await db.collection('users').doc(user.uid).get();
    expect(userDoc.exists).toBe(true);
    expect(userDoc.data()?.email).toBe('test@example.com');
  });
});
```

## Firestore Security Rules Testing

```typescript
// firestore.rules.test.ts
import {
  assertFails,
  assertSucceeds,
  initializeTestEnvironment,
  RulesTestEnvironment,
} from '@firebase/rules-unit-testing';
import { setDoc, getDoc, doc } from 'firebase/firestore';

let testEnv: RulesTestEnvironment;

beforeAll(async () => {
  testEnv = await initializeTestEnvironment({
    projectId: 'test-project',
    firestore: {
      rules: fs.readFileSync('firestore.rules', 'utf8'),
    },
  });
});

afterAll(async () => {
  await testEnv.cleanup();
});

describe('Firestore Security Rules', () => {
  it('allows user to read their own data', async () => {
    const alice = testEnv.authenticatedContext('alice');
    const aliceDoc = doc(alice.firestore(), 'users/alice');

    await assertSucceeds(getDoc(aliceDoc));
  });

  it('denies user from reading others data', async () => {
    const alice = testEnv.authenticatedContext('alice');
    const bobDoc = doc(alice.firestore(), 'users/bob');

    await assertFails(getDoc(bobDoc));
  });

  it('allows subscription owner to write', async () => {
    const alice = testEnv.authenticatedContext('alice');
    const subDoc = doc(alice.firestore(), 'subscriptions/sub_123');

    await testEnv.withSecurityRulesDisabled(async (context) => {
      await setDoc(doc(context.firestore(), 'subscriptions/sub_123'), {
        ownerId: 'alice',
      });
    });

    await assertSucceeds(
      setDoc(subDoc, { name: 'Updated' }, { merge: true })
    );
  });

  it('denies non-owner from writing to subscription', async () => {
    const alice = testEnv.authenticatedContext('alice');
    const bobDoc = doc(alice.firestore(), 'subscriptions/sub_456');

    await testEnv.withSecurityRulesDisabled(async (context) => {
      await setDoc(doc(context.firestore(), 'subscriptions/sub_456'), {
        ownerId: 'bob',
      });
    });

    await assertFails(setDoc(bobDoc, { name: 'Hacked' }));
  });
});
```

## End-to-End Testing

### Playwright Setup

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:5173',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
});
```

### E2E Test Examples

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('user can sign up', async ({ page }) => {
    await page.goto('/signup');

    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.fill('input[name="confirmPassword"]', 'password123');

    await page.click('button[type="submit"]');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('text=Welcome')).toBeVisible();
  });

  test('user can sign in', async ({ page }) => {
    await page.goto('/login');

    await page.fill('input[name="email"]', 'existing@example.com');
    await page.fill('input[name="password"]', 'password123');

    await page.click('button[type="submit"]');

    await expect(page).toHaveURL('/dashboard');
  });

  test('shows error for invalid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.fill('input[name="email"]', 'wrong@example.com');
    await page.fill('input[name="password"]', 'wrongpassword');

    await page.click('button[type="submit"]');

    await expect(page.locator('text=Incorrect password')).toBeVisible();
  });
});
```

```typescript
// e2e/subscription.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Subscription Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Sign in before each test
    await page.goto('/login');
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    await page.waitForURL('/dashboard');
  });

  test('user can view subscription plans', async ({ page }) => {
    await page.goto('/pricing');

    await expect(page.locator('text=Starter')).toBeVisible();
    await expect(page.locator('text=Professional')).toBeVisible();
    await expect(page.locator('text=Enterprise')).toBeVisible();
  });

  test('user can subscribe to a plan', async ({ page }) => {
    await page.goto('/pricing');

    await page.click('button:has-text("Choose Professional")');

    // Fill payment details (using Stripe test card)
    const stripeFrame = page.frameLocator('iframe[name^="__privateStripeFrame"]');
    await stripeFrame.locator('input[name="cardnumber"]').fill('4242424242424242');
    await stripeFrame.locator('input[name="exp-date"]').fill('12/25');
    await stripeFrame.locator('input[name="cvc"]').fill('123');

    await page.click('button:has-text("Subscribe")');

    await expect(page).toHaveURL('/subscription/success');
    await expect(page.locator('text=Subscription Active')).toBeVisible();
  });
});
```

## Testing Best Practices

### 1. Follow AAA Pattern

```typescript
test('component behavior', () => {
  // Arrange
  const mockData = { name: 'Test' };
  const mockCallback = vi.fn();

  // Act
  render(<Component data={mockData} onClick={mockCallback} />);
  fireEvent.click(screen.getByText('Click me'));

  // Assert
  expect(mockCallback).toHaveBeenCalledWith(mockData);
});
```

### 2. Use Test IDs for Complex Selectors

```typescript
// Component
<button data-testid="submit-button">Submit</button>

// Test
const button = screen.getByTestId('submit-button');
```

### 3. Mock External Dependencies

```typescript
// Mock Stripe
vi.mock('stripe', () => ({
  default: vi.fn(() => ({
    customers: {
      create: vi.fn(),
    },
  })),
}));

// Mock Firebase
vi.mock('firebase/firestore', () => ({
  getDoc: vi.fn(),
  setDoc: vi.fn(),
}));
```

### 4. Test User Interactions, Not Implementation

```typescript
// ❌ Bad: Testing implementation
expect(component.state.count).toBe(1);

// ✅ Good: Testing user-visible behavior
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

## Coverage Goals

- **Unit Tests**: 80%+ coverage
- **Integration Tests**: Critical paths covered
- **E2E Tests**: Main user flows covered
- **Security Rules**: All rules tested

## Testing Checklist

- [ ] Unit tests for all components
- [ ] Unit tests for all hooks
- [ ] Unit tests for utility functions
- [ ] Integration tests for Cloud Functions
- [ ] Security rules tests
- [ ] E2E tests for critical user flows
- [ ] CI/CD pipeline runs tests
- [ ] Code coverage reports generated
- [ ] Mocks for external services
- [ ] Test data cleanup after tests

## See Also

- [Code Organization Best Practices](/best-practices/code-organization/)
- [Error Handling Best Practices](/best-practices/error-handling/)
- [Deployment Guide](../../DEPLOYMENT.md)
- [CI/CD Setup](/demo/#cicd)
