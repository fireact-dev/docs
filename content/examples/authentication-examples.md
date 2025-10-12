---
title: "Authentication Examples"
linkTitle: "Authentication"
type: "docs"
weight: 1
description: >
  Common authentication patterns and examples for Fireact.dev applications.
---

## Custom Sign-In Form

```typescript
import React, { useState } from 'react';
import { signInWithEmailAndPassword } from 'firebase/auth';
import { useNavigate } from 'react-router-dom';
import { useConfig } from '../hooks/useConfig';

export const CustomSignIn: React.FC = () => {
  const { firebaseApp } = useConfig();
  const navigate = useNavigate();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError(null);
    setLoading(true);

    try {
      const auth = firebaseApp.auth();
      await signInWithEmailAndPassword(auth, email, password);
      navigate('/dashboard');
    } catch (err: any) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Form implementation */}
    </form>
  );
};
```

## Social Login

### Google Sign-In

```typescript
import { signInWithPopup, GoogleAuthProvider } from 'firebase/auth';

export const GoogleSignIn: React.FC = () => {
  const { firebaseApp } = useConfig();

  const handleGoogleSignIn = async () => {
    try {
      const auth = firebaseApp.auth();
      const provider = new GoogleAuthProvider();

      // Optional: Add custom parameters
      provider.setCustomParameters({
        prompt: 'select_account'
      });

      const result = await signInWithPopup(auth, provider);
      const user = result.user;

      // Get additional info
      const credential = GoogleAuthProvider.credentialFromResult(result);
      const token = credential?.accessToken;

      console.log('Signed in user:', user);
    } catch (error: any) {
      console.error('Google sign-in error:', error);
    }
  };

  return (
    <button onClick={handleGoogleSignIn}>
      Sign in with Google
    </button>
  );
};
```

### GitHub Sign-In

```typescript
import { signInWithPopup, GithubAuthProvider } from 'firebase/auth';

export const handleGitHubSignIn = async () => {
  const auth = firebaseApp.auth();
  const provider = new GithubAuthProvider();

  provider.addScope('user:email');

  try {
    const result = await signInWithPopup(auth, provider);
    const user = result.user;
    return user;
  } catch (error: any) {
    if (error.code === 'auth/account-exists-with-different-credential') {
      // Handle account linking
    }
    throw error;
  }
};
```

## Email Verification

### Send Verification Email

```typescript
import { sendEmailVerification } from 'firebase/auth';

export const sendVerificationEmail = async (user: User) => {
  try {
    await sendEmailVerification(user, {
      url: 'https://yourapp.com/verify-email',
      handleCodeInApp: true
    });
    console.log('Verification email sent');
  } catch (error) {
    console.error('Failed to send verification email:', error);
  }
};
```

### Check Verification Status

```typescript
import { useAuth } from '../hooks/useAuth';

export const EmailVerificationGuard: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { user } = useAuth();

  if (!user?.emailVerified) {
    return (
      <div>
        <h2>Please verify your email</h2>
        <p>Check your inbox for verification link</p>
        <button onClick={() => sendVerificationEmail(user!)}>
          Resend Verification Email
        </button>
      </div>
    );
  }

  return <>{children}</>;
};
```

## Password Reset

```typescript
import { sendPasswordResetEmail } from 'firebase/auth';

export const PasswordReset: React.FC = () => {
  const { firebaseApp } = useConfig();
  const [email, setEmail] = useState('');
  const [sent, setSent] = useState(false);

  const handleReset = async (e: React.FormEvent) => {
    e.preventDefault();

    try {
      const auth = firebaseApp.auth();
      await sendPasswordResetEmail(auth, email, {
        url: 'https://yourapp.com/sign-in',
        handleCodeInApp: false
      });
      setSent(true);
    } catch (error) {
      console.error('Password reset error:', error);
    }
  };

  return (
    <form onSubmit={handleReset}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Enter your email"
        required
      />
      <button type="submit">Reset Password</button>
      {sent && <p>Password reset email sent!</p>}
    </form>
  );
};
```

## Protected Routes

```typescript
import { Navigate } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';

export const ProtectedRoute: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { user, loading } = useAuth();

  if (loading) {
    return <div>Loading...</div>;
  }

  if (!user) {
    return <Navigate to="/sign-in" replace />;
  }

  return <>{children}</>;
};
```

## Session Management

```typescript
import { setPersistence, browserLocalPersistence, browserSessionPersistence } from 'firebase/auth';

// Persistent session (remember me)
export const setRememberMe = async (auth: Auth, remember: boolean) => {
  const persistence = remember ? browserLocalPersistence : browserSessionPersistence;
  await setPersistence(auth, persistence);
};

// Auto logout on inactivity
export const useAutoLogout = (timeoutMinutes: number = 30) => {
  const { signOut } = useAuth();

  useEffect(() => {
    let timeout: NodeJS.Timeout;

    const resetTimer = () => {
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        signOut();
      }, timeoutMinutes * 60 * 1000);
    };

    // Reset on user activity
    window.addEventListener('mousemove', resetTimer);
    window.addEventListener('keypress', resetTimer);

    resetTimer();

    return () => {
      clearTimeout(timeout);
      window.removeEventListener('mousemove', resetTimer);
      window.removeEventListener('keypress', resetTimer);
    };
  }, [signOut, timeoutMinutes]);
};
```

For more authentication examples, see the [Firebase Auth Documentation](https://firebase.google.com/docs/auth).
