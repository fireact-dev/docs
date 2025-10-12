---
title: "Building Your First Custom Feature"
linkTitle: "First Custom Feature"
type: "docs"
weight: 1
description: >
  Learn how to build a custom feature end-to-end, from React component to Cloud Function to Firestore integration.
---

## Overview

In this tutorial, you'll build a **Task Management** feature that allows users to create, view, and manage tasks within their subscription. This will teach you:

- Creating custom React components
- Building Cloud Functions for backend logic
- Working with Firestore for data storage
- Implementing proper permissions and security
- Using Fireact.dev contexts and hooks

**What you'll build:**
- A tasks list page
- Task creation form
- Cloud Function to create tasks
- Firestore security rules for tasks

**Time to complete:** ~45 minutes

## Prerequisites

- Completed [Getting Started Guide](/getting-started/)
- Basic understanding of React, TypeScript, and Firebase
- Working Fireact.dev application running locally

## Step 1: Plan Your Feature

### Data Model

First, define the Firestore data structure:

```typescript
// Task document structure
interface Task {
  id: string;
  title: string;
  description: string;
  status: 'pending' | 'in-progress' | 'completed';
  subscriptionId: string;
  createdBy: string;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}
```

### Firestore Collection Structure

```
firestore/
└── subscriptions/
    └── {subscriptionId}/
        └── tasks/              # Subcollection
            └── {taskId}/
                ├── title
                ├── description
                ├── status
                ├── createdBy
                ├── createdAt
                └── updatedAt
```

## Step 2: Create the Task Type Definition

Add TypeScript types to your application:

**Location:** `src/types.ts`

```typescript
export interface Task {
  id: string;
  title: string;
  description: string;
  status: 'pending' | 'in-progress' | 'completed';
  subscriptionId: string;
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface CreateTaskData {
  title: string;
  description: string;
  subscriptionId: string;
}
```

## Step 3: Create the Cloud Function

### 3.1 Create Function File

**Location:** `functions/src/functions/tasks/createTask.ts`

```typescript
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

/**
 * Creates a new task within a subscription
 *
 * @param {Object} data - Task data
 * @param {string} data.title - Task title
 * @param {string} data.description - Task description
 * @param {string} data.subscriptionId - Subscription ID
 * @param {CallableContext} context - Function context with auth info
 * @returns {Promise<{taskId: string}>} Created task ID
 */
export const createTask = functions.https.onCall(
  async (data: {
    title: string;
    description: string;
    subscriptionId: string;
  }, context) => {
    // Validate authentication
    if (!context.auth) {
      throw new functions.https.HttpsError(
        'unauthenticated',
        'User must be authenticated'
      );
    }

    const userId = context.auth.uid;
    const { title, description, subscriptionId } = data;

    // Validate input
    if (!title || !description || !subscriptionId) {
      throw new functions.https.HttpsError(
        'invalid-argument',
        'Title, description, and subscriptionId are required'
      );
    }

    // Verify user has access to subscription
    const subscriptionRef = admin.firestore()
      .collection('subscriptions')
      .doc(subscriptionId);

    const subscriptionDoc = await subscriptionRef.get();

    if (!subscriptionDoc.exists) {
      throw new functions.https.HttpsError(
        'not-found',
        'Subscription not found'
      );
    }

    // Check user is member of subscription
    const userSubscriptionRef = admin.firestore()
      .collection('users')
      .doc(userId)
      .collection('subscriptions')
      .doc(subscriptionId);

    const userSubscriptionDoc = await userSubscriptionRef.get();

    if (!userSubscriptionDoc.exists) {
      throw new functions.https.HttpsError(
        'permission-denied',
        'User does not have access to this subscription'
      );
    }

    // Create task
    const taskRef = subscriptionRef
      .collection('tasks')
      .doc();

    const taskData = {
      id: taskRef.id,
      title,
      description,
      status: 'pending',
      subscriptionId,
      createdBy: userId,
      createdAt: admin.firestore.FieldValue.serverTimestamp(),
      updatedAt: admin.firestore.FieldValue.serverTimestamp(),
    };

    await taskRef.set(taskData);

    return {
      taskId: taskRef.id,
      message: 'Task created successfully'
    };
  }
);
```

### 3.2 Export the Function

**Location:** `functions/src/index.ts`

Add the export:

```typescript
export { createTask } from './functions/tasks/createTask';
```

### 3.3 Build Functions

```bash
cd functions
npm run build
cd ..
```

## Step 4: Update Firestore Security Rules

**Location:** `firestore.rules`

Add rules for tasks subcollection:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Helper function to check subscription membership
    function isSubscriptionMember(subscriptionId) {
      return exists(/databases/$(database)/documents/users/$(request.auth.uid)/subscriptions/$(subscriptionId));
    }

    // Existing rules...

    // Tasks subcollection rules
    match /subscriptions/{subscriptionId}/tasks/{taskId} {
      // Members can read tasks
      allow read: if isSubscriptionMember(subscriptionId);

      // Members can create tasks (through Cloud Function)
      allow create: if isSubscriptionMember(subscriptionId);

      // Members can update tasks they created
      allow update: if isSubscriptionMember(subscriptionId)
                    && resource.data.createdBy == request.auth.uid;

      // Members can delete tasks they created
      allow delete: if isSubscriptionMember(subscriptionId)
                    && resource.data.createdBy == request.auth.uid;
    }
  }
}
```

Deploy rules (if using production):
```bash
firebase deploy --only firestore:rules
```

## Step 5: Create React Components

### 5.1 Create Task List Component

**Location:** `src/components/TaskList.tsx`

```typescript
import React, { useState, useEffect } from 'react';
import { collection, query, onSnapshot, orderBy } from 'firebase/firestore';
import { useParams } from 'react-router-dom';
import { useConfig } from '../hooks/useConfig';
import { Task } from '../types';

export const TaskList: React.FC = () => {
  const { subscriptionId } = useParams<{ subscriptionId: string }>();
  const { firebaseApp } = useConfig();
  const [tasks, setTasks] = useState<Task[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!subscriptionId) return;

    const db = firebaseApp.firestore();
    const tasksRef = collection(
      db,
      'subscriptions',
      subscriptionId,
      'tasks'
    );
    const q = query(tasksRef, orderBy('createdAt', 'desc'));

    const unsubscribe = onSnapshot(q, (snapshot) => {
      const tasksData: Task[] = [];
      snapshot.forEach((doc) => {
        tasksData.push({
          ...doc.data(),
          createdAt: doc.data().createdAt?.toDate(),
          updatedAt: doc.data().updatedAt?.toDate(),
        } as Task);
      });
      setTasks(tasksData);
      setLoading(false);
    });

    return () => unsubscribe();
  }, [subscriptionId, firebaseApp]);

  if (loading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="text-gray-600">Loading tasks...</div>
      </div>
    );
  }

  if (tasks.length === 0) {
    return (
      <div className="text-center py-12">
        <h3 className="text-lg font-medium text-gray-900 mb-2">
          No tasks yet
        </h3>
        <p className="text-gray-600">
          Create your first task to get started
        </p>
      </div>
    );
  }

  return (
    <div className="space-y-4">
      {tasks.map((task) => (
        <div
          key={task.id}
          className="bg-white shadow rounded-lg p-6 border border-gray-200"
        >
          <div className="flex items-start justify-between">
            <div className="flex-1">
              <h3 className="text-lg font-medium text-gray-900">
                {task.title}
              </h3>
              <p className="mt-1 text-sm text-gray-600">
                {task.description}
              </p>
              <div className="mt-2">
                <span
                  className={`inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium
                    ${task.status === 'completed' ? 'bg-green-100 text-green-800' : ''}
                    ${task.status === 'in-progress' ? 'bg-blue-100 text-blue-800' : ''}
                    ${task.status === 'pending' ? 'bg-gray-100 text-gray-800' : ''}
                  `}
                >
                  {task.status}
                </span>
              </div>
            </div>
          </div>
          <div className="mt-4 text-xs text-gray-500">
            Created {task.createdAt.toLocaleDateString()}
          </div>
        </div>
      ))}
    </div>
  );
};
```

### 5.2 Create Task Form Component

**Location:** `src/components/CreateTaskForm.tsx`

```typescript
import React, { useState } from 'react';
import { useParams, useNavigate } from 'react-router-dom';
import { httpsCallable } from 'firebase/functions';
import { useConfig } from '../hooks/useConfig';
import { useLoading } from '../hooks/useLoading';

export const CreateTaskForm: React.FC = () => {
  const { subscriptionId } = useParams<{ subscriptionId: string }>();
  const navigate = useNavigate();
  const { firebaseApp } = useConfig();
  const { setLoading } = useLoading();

  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError(null);
    setLoading(true);

    try {
      const functions = firebaseApp.functions();
      const createTask = httpsCallable(functions, 'createTask');

      await createTask({
        title,
        description,
        subscriptionId,
      });

      // Reset form
      setTitle('');
      setDescription('');

      // Navigate back to tasks list
      navigate(`/${subscriptionId}/tasks`);
    } catch (err: any) {
      setError(err.message || 'Failed to create task');
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-6">
      {error && (
        <div className="rounded-md bg-red-50 p-4">
          <div className="text-sm text-red-800">{error}</div>
        </div>
      )}

      <div>
        <label
          htmlFor="title"
          className="block text-sm font-medium text-gray-700"
        >
          Task Title
        </label>
        <input
          type="text"
          id="title"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          required
          className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500"
        />
      </div>

      <div>
        <label
          htmlFor="description"
          className="block text-sm font-medium text-gray-700"
        >
          Description
        </label>
        <textarea
          id="description"
          rows={4}
          value={description}
          onChange={(e) => setDescription(e.target.value)}
          required
          className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500"
        />
      </div>

      <div className="flex justify-end space-x-3">
        <button
          type="button"
          onClick={() => navigate(`/${subscriptionId}/tasks`)}
          className="px-4 py-2 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50"
        >
          Cancel
        </button>
        <button
          type="submit"
          className="px-4 py-2 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-blue-600 hover:bg-blue-700"
        >
          Create Task
        </button>
      </div>
    </form>
  );
};
```

### 5.3 Create Tasks Page

**Location:** `src/components/TasksPage.tsx`

```typescript
import React from 'react';
import { Link, useParams } from 'react-router-dom';
import { TaskList } from './TaskList';

export const TasksPage: React.FC = () => {
  const { subscriptionId } = useParams<{ subscriptionId: string }>();

  return (
    <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold text-gray-900">Tasks</h1>
        <Link
          to={`/${subscriptionId}/tasks/new`}
          className="inline-flex items-center px-4 py-2 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-blue-600 hover:bg-blue-700"
        >
          Create Task
        </Link>
      </div>
      <TaskList />
    </div>
  );
};
```

## Step 6: Add Routes

**Location:** `src/App.tsx`

Add routes for the tasks feature:

```typescript
import { TasksPage } from './components/TasksPage';
import { CreateTaskForm } from './components/CreateTaskForm';

// Inside your routes configuration:
<Route element={<ProtectedSubscriptionRoute />}>
  <Route element={<SubscriptionLayout />}>
    {/* Existing routes... */}

    {/* Task routes */}
    <Route path="/:subscriptionId/tasks" element={<TasksPage />} />
    <Route
      path="/:subscriptionId/tasks/new"
      element={
        <div className="max-w-2xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
          <h1 className="text-2xl font-bold text-gray-900 mb-6">
            Create New Task
          </h1>
          <CreateTaskForm />
        </div>
      }
    />
  </Route>
</Route>
```

## Step 7: Add Navigation Link

Update your subscription navigation menu to include a link to tasks.

**Location:** Update your navigation component

```typescript
<nav className="space-y-1">
  {/* Existing menu items */}

  <Link
    to={`/${subscriptionId}/tasks`}
    className="group flex items-center px-3 py-2 text-sm font-medium rounded-md hover:bg-gray-50"
  >
    Tasks
  </Link>
</nav>
```

## Step 8: Test Your Feature

### 8.1 Start Development Environment

```bash
# Rebuild functions
cd functions
npm run build
cd ..

# Start emulators
firebase emulators:start
```

### 8.2 Test the Feature

1. **Navigate to Tasks Page**:
   - Sign in to your application
   - Select a subscription
   - Click "Tasks" in the navigation

2. **Create a Task**:
   - Click "Create Task" button
   - Fill in title and description
   - Click "Create Task"

3. **Verify Task Appears**:
   - Should redirect to tasks list
   - New task should appear in the list

4. **Check Firestore**:
   - Open emulator UI: http://localhost:4000
   - Navigate to Firestore
   - Verify task document exists in correct location

### 8.3 Test Error Scenarios

1. **Unauthorized Access**:
   - Try accessing a subscription you're not a member of
   - Should see permission error

2. **Invalid Data**:
   - Try creating task with empty fields
   - Should see validation error

## Step 9: Add Internationalization (Optional)

Add translations for your new feature to all language files:

**Location:** `src/i18n/en.ts`

```typescript
export default {
  // ... existing translations (auth, profile, subscription, etc.)

  // Add new tasks category
  tasks: {
    title: "Tasks",
    create: "Create Task",
    noTasks: "No tasks yet",
    createFirst: "Create your first task to get started",
    taskTitle: "Task Title",
    description: "Description",
    status: {
      pending: "Pending",
      inProgress: "In Progress",
      completed: "Completed"
    }
  },

  // ... other translations
};
```

**Location:** `src/i18n/zh.ts` (Chinese translation)

```typescript
export default {
  // ... existing translations

  tasks: {
    title: "任务",
    create: "创建任务",
    noTasks: "暂无任务",
    createFirst: "创建您的第一个任务开始使用",
    taskTitle: "任务标题",
    description: "描述",
    status: {
      pending: "待处理",
      inProgress: "进行中",
      completed: "已完成"
    }
  },

  // ... other translations
};
```

**Note:** Fireact uses TypeScript (`.ts`) files, not JSON. Add the same structure to all language files (de.ts, fr.ts, es.ts, etc.).

Use in components:

```typescript
import { useTranslation } from 'react-i18next';

function TaskList() {
  const { t } = useTranslation();

  return (
    <div>
      <h1>{t('tasks.title')}</h1>
      <button>{t('tasks.create')}</button>
      <p>{t('tasks.noTasks')}</p>
      <span>{t('tasks.status.pending')}</span>
    </div>
  );
}
```

## Next Steps

Now that you've built your first feature, you can:

1. **Add More Functionality**:
   - Update task status
   - Delete tasks
   - Filter tasks by status
   - Assign tasks to team members

2. **Improve UI**:
   - Add loading states
   - Add animations
   - Improve mobile responsiveness

3. **Add More Features**:
   - Task comments
   - Due dates and reminders
   - File attachments
   - Task categories/tags

4. **Learn More**:
   - [Customizing the UI Theme](customizing-ui-theme/)
   - [Implementing Custom Subscription Plans](custom-subscription-plans/)
   - [Adding Third-Party Integrations](third-party-integrations/)

## Key Takeaways

✅ Cloud Functions handle backend logic and security
✅ Firestore security rules provide additional protection
✅ Use Fireact.dev hooks and contexts for consistent UX
✅ Real-time updates with Firestore listeners
✅ TypeScript ensures type safety throughout the stack

## Troubleshooting

### Function Not Found

If you get "function not found" error:
```bash
cd functions
npm run build
cd ..
# Restart emulators
```

### Permission Denied

Check:
- User is authenticated
- User is member of subscription
- Firestore rules are correct

### Task Not Appearing

- Check browser console for errors
- Verify Firestore emulator UI shows the task
- Check component is subscribed to real-time updates

For more help, see the [Troubleshooting Guide](https://github.com/fireact-dev/fireact.dev/blob/main/TROUBLESHOOTING.md).
