# Phase 1: Shared Infrastructure - Detailed Implementation Guide

## 🎯 Phase Overview

**Objective**: Build the common foundation that all microfrontends will use for communication, data management, and development tooling.

**Duration**: 3 days

**Dependencies**: Phase 0 (Project Foundation Setup) must be complete

## 📋 Phase 1 Deliverables

- ✅ NPM package: `@todo-mfe/shared` with full test coverage
- ✅ Event Bus system for inter-MFE communication
- ✅ Local Storage service wrapper
- ✅ TypeScript interfaces and types
- ✅ Webpack/Vite configuration templates
- ✅ Module Federation plugin setup
- ✅ Docker base images
- ✅ Development orchestration scripts

---

## 🚀 Step 1.1: Create Shared Utilities Package

### 1.1.1: Initialize Shared Package Structure

```bash
# Create the shared package structure
mkdir -p packages/shared/src/{events,storage,types,utils,constants,__tests__}
mkdir -p packages/shared/src/{events,storage,types,utils,constants}/__tests__

# Create package.json
cd packages/shared
```

**File**: `packages/shared/package.json`
```json
{
  "name": "@todo-mfe/shared",
  "version": "1.0.0",
  "description": "Shared utilities for Todo Microfrontends",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/**/*.ts",
    "clean": "rm -rf dist"
  },
  "files": [
    "dist/**/*"
  ],
  "keywords": ["microfrontend", "shared", "todo"],
  "author": "Todo MFE Team",
  "license": "MIT",
  "devDependencies": {
    "@types/jest": "^29.5.11",
    "jest": "^29.7.0",
    "ts-jest": "^29.1.1",
    "typescript": "^5.3.3"
  },
  "peerDependencies": {
    "typescript": "^5.0.0"
  }
}
```

**File**: `packages/shared/tsconfig.json`
```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "rootDir": "./src",
    "module": "commonjs",
    "target": "ES2020"
  },
  "include": ["src/**/*"],
  "exclude": ["dist", "node_modules", "**/*.test.ts", "**/*.spec.ts"]
}
```

**File**: `packages/shared/jest.config.js`
```javascript
module.exports = {
  preset: '../../jest.preset.js',
  displayName: 'shared',
  testEnvironment: 'jsdom',
  transform: {
    '^.+\\.[tj]s$': ['ts-jest', { tsconfig: '<rootDir>/tsconfig.json' }],
  },
  moduleFileExtensions: ['ts', 'js', 'html'],
  coverageDirectory: '../../coverage/packages/shared',
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/index.ts',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

### 1.1.2: Implement Event Bus System

**File**: `packages/shared/src/events/types.ts`
```typescript
// Event types for inter-MFE communication
export interface TodoItem {
  id: string;
  title: string;
  description?: string;
  completed: boolean;
  category?: string;
  priority: 'low' | 'medium' | 'high';
  dueDate?: Date;
  createdAt: Date;
  updatedAt: Date;
}

export interface TodoEvent {
  type: string;
  payload: any;
  timestamp: number;
  source: string;
}

// Specific event types
export interface TaskCreatedEvent extends TodoEvent {
  type: 'task:created';
  payload: TodoItem;
}

export interface TaskUpdatedEvent extends TodoEvent {
  type: 'task:updated';
  payload: { id: string } & Partial<TodoItem>;
}

export interface TaskDeletedEvent extends TodoEvent {
  type: 'task:deleted';
  payload: { id: string };
}

export interface ThemeChangedEvent extends TodoEvent {
  type: 'theme:changed';
  payload: { theme: 'light' | 'dark' };
}

export interface CategoryModifiedEvent extends TodoEvent {
  type: 'category:modified';
  payload: { action: 'add' | 'remove' | 'update'; category: string };
}

export interface DataSyncEvent extends TodoEvent {
  type: 'data:sync';
  payload: { timestamp: number };
}

export type MFEEvent =
  | TaskCreatedEvent
  | TaskUpdatedEvent
  | TaskDeletedEvent
  | ThemeChangedEvent
  | CategoryModifiedEvent
  | DataSyncEvent;

export type EventHandler<T extends TodoEvent = TodoEvent> = (event: T) => void;
export type EventType = MFEEvent['type'];
```

**File**: `packages/shared/src/events/EventBus.ts`
```typescript
import { TodoEvent, EventHandler, EventType } from './types';

export class EventBus {
  private listeners: Map<string, Set<EventHandler>> = new Map();
  private static instance: EventBus;

  private constructor() {}

  public static getInstance(): EventBus {
    if (!EventBus.instance) {
      EventBus.instance = new EventBus();
    }
    return EventBus.instance;
  }

  /**
   * Subscribe to an event type
   */
  public on<T extends TodoEvent>(eventType: EventType, handler: EventHandler<T>): () => void {
    if (!this.listeners.has(eventType)) {
      this.listeners.set(eventType, new Set());
    }

    const handlers = this.listeners.get(eventType)!;
    handlers.add(handler as EventHandler);

    // Return unsubscribe function
    return () => {
      handlers.delete(handler as EventHandler);
      if (handlers.size === 0) {
        this.listeners.delete(eventType);
      }
    };
  }

  /**
   * Emit an event to all subscribers
   */
  public emit<T extends TodoEvent>(event: T): void {
    const handlers = this.listeners.get(event.type);
    if (handlers) {
      handlers.forEach(handler => {
        try {
          handler(event);
        } catch (error) {
          console.error(`Error in event handler for ${event.type}:`, error);
        }
      });
    }

    // Log event for debugging in development
    if (process.env.NODE_ENV === 'development') {
      console.log(`[EventBus] ${event.type}:`, event.payload);
    }
  }

  /**
   * Remove all listeners for an event type
   */
  public off(eventType: EventType): void {
    this.listeners.delete(eventType);
  }

  /**
   * Remove all listeners
   */
  public clear(): void {
    this.listeners.clear();
  }

  /**
   * Get list of active event types
   */
  public getActiveEventTypes(): string[] {
    return Array.from(this.listeners.keys());
  }

  /**
   * Get listener count for an event type
   */
  public getListenerCount(eventType: EventType): number {
    return this.listeners.get(eventType)?.size || 0;
  }
}

// Export singleton instance
export const eventBus = EventBus.getInstance();
```

### 1.1.3: Implement Local Storage Service

**File**: `packages/shared/src/storage/types.ts`
```typescript
export interface StorageOptions {
  prefix?: string;
  serialize?: (value: any) => string;
  deserialize?: (value: string) => any;
}

export interface StorageService {
  get<T>(key: string): T | null;
  set<T>(key: string, value: T): void;
  remove(key: string): void;
  clear(): void;
  exists(key: string): boolean;
  getAllKeys(): string[];
  getSize(): number;
}
```

**File**: `packages/shared/src/storage/LocalStorageService.ts`
```typescript
import { StorageService, StorageOptions } from './types';
import { eventBus } from '../events/EventBus';

export class LocalStorageService implements StorageService {
  private prefix: string;
  private serialize: (value: any) => string;
  private deserialize: (value: string) => any;

  constructor(options: StorageOptions = {}) {
    this.prefix = options.prefix || 'todo-mfe:';
    this.serialize = options.serialize || JSON.stringify;
    this.deserialize = options.deserialize || JSON.parse;
  }

  private getKey(key: string): string {
    return `${this.prefix}${key}`;
  }

  public get<T>(key: string): T | null {
    try {
      const item = localStorage.getItem(this.getKey(key));
      if (item === null) {
        return null;
      }
      return this.deserialize(item) as T;
    } catch (error) {
      console.error(`Error getting item from localStorage: ${key}`, error);
      return null;
    }
  }

  public set<T>(key: string, value: T): void {
    try {
      const serializedValue = this.serialize(value);
      localStorage.setItem(this.getKey(key), serializedValue);

      // Emit storage change event
      eventBus.emit({
        type: 'storage:changed',
        payload: { key, value },
        timestamp: Date.now(),
        source: 'LocalStorageService'
      } as any);
    } catch (error) {
      console.error(`Error setting item in localStorage: ${key}`, error);
    }
  }

  public remove(key: string): void {
    try {
      localStorage.removeItem(this.getKey(key));

      // Emit storage change event
      eventBus.emit({
        type: 'storage:removed',
        payload: { key },
        timestamp: Date.now(),
        source: 'LocalStorageService'
      } as any);
    } catch (error) {
      console.error(`Error removing item from localStorage: ${key}`, error);
    }
  }

  public clear(): void {
    try {
      const keys = this.getAllKeys();
      keys.forEach(key => localStorage.removeItem(key));

      // Emit storage clear event
      eventBus.emit({
        type: 'storage:cleared',
        payload: { clearedKeys: keys.length },
        timestamp: Date.now(),
        source: 'LocalStorageService'
      } as any);
    } catch (error) {
      console.error('Error clearing localStorage', error);
    }
  }

  public exists(key: string): boolean {
    return localStorage.getItem(this.getKey(key)) !== null;
  }

  public getAllKeys(): string[] {
    const keys: string[] = [];
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      if (key && key.startsWith(this.prefix)) {
        keys.push(key);
      }
    }
    return keys;
  }

  public getSize(): number {
    return this.getAllKeys().length;
  }

  public exportData(): Record<string, any> {
    const data: Record<string, any> = {};
    const keys = this.getAllKeys();

    keys.forEach(fullKey => {
      const key = fullKey.replace(this.prefix, '');
      data[key] = this.get(key);
    });

    return data;
  }

  public importData(data: Record<string, any>): void {
    Object.entries(data).forEach(([key, value]) => {
      this.set(key, value);
    });
  }
}

// Export singleton instance
export const storageService = new LocalStorageService();
```

### 1.1.4: Define TypeScript Interfaces

**File**: `packages/shared/src/types/index.ts`
```typescript
// Re-export all types
export * from '../events/types';
export * from '../storage/types';

// Additional shared types
export interface User {
  id: string;
  name: string;
  email?: string;
  preferences: UserPreferences;
  createdAt: Date;
}

export interface UserPreferences {
  theme: 'light' | 'dark';
  language: string;
  notifications: NotificationSettings;
  categories: string[];
  defaultPriority: 'low' | 'medium' | 'high';
}

export interface NotificationSettings {
  enabled: boolean;
  emailNotifications: boolean;
  pushNotifications: boolean;
  dueDateReminders: boolean;
  weeklyDigest: boolean;
}

export interface AppConfig {
  version: string;
  environment: 'development' | 'staging' | 'production';
  features: FeatureFlags;
  api: ApiConfig;
}

export interface FeatureFlags {
  analytics: boolean;
  export: boolean;
  collaboration: boolean;
  darkMode: boolean;
  offlineMode: boolean;
}

export interface ApiConfig {
  baseUrl: string;
  timeout: number;
  retryAttempts: number;
}

export interface MFEManifest {
  name: string;
  version: string;
  url: string;
  scope: string;
  module: string;
  dependencies?: string[];
  fallback?: string;
}

// Utility types
export type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

export type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
export type Required<T, K extends keyof T> = T & Pick<Required<T>, K>;
```

### 1.1.5: Create Shared Constants

**File**: `packages/shared/src/constants/index.ts`
```typescript
// Application constants
export const APP_CONFIG = {
  NAME: 'Todo Microfrontends',
  VERSION: '1.0.0',
  DESCRIPTION: 'A microfrontend TODO application',
} as const;

// Storage keys
export const STORAGE_KEYS = {
  TODOS: 'todos',
  USER: 'user',
  PREFERENCES: 'preferences',
  CATEGORIES: 'categories',
  THEME: 'theme',
  LAST_SYNC: 'lastSync',
} as const;

// Event types (for type safety)
export const EVENT_TYPES = {
  TASK_CREATED: 'task:created',
  TASK_UPDATED: 'task:updated',
  TASK_DELETED: 'task:deleted',
  THEME_CHANGED: 'theme:changed',
  CATEGORY_MODIFIED: 'category:modified',
  DATA_SYNC: 'data:sync',
  STORAGE_CHANGED: 'storage:changed',
  STORAGE_REMOVED: 'storage:removed',
  STORAGE_CLEARED: 'storage:cleared',
} as const;

// Default values
export const DEFAULTS = {
  THEME: 'light' as const,
  PRIORITY: 'medium' as const,
  CATEGORIES: ['Personal', 'Work', 'Shopping', 'Health'],
  USER_PREFERENCES: {
    theme: 'light' as const,
    language: 'en',
    notifications: {
      enabled: true,
      emailNotifications: false,
      pushNotifications: true,
      dueDateReminders: true,
      weeklyDigest: false,
    },
    categories: ['Personal', 'Work', 'Shopping', 'Health'],
    defaultPriority: 'medium' as const,
  },
} as const;

// MFE Configuration
export const MFE_CONFIG = {
  SHELL: {
    NAME: 'shell',
    PORT: 9000,
    SCOPE: 'shell',
  },
  TASK_MANAGER: {
    NAME: 'task-manager',
    PORT: 9001,
    SCOPE: 'taskManager',
    MODULE: './TaskManager',
  },
  ANALYTICS: {
    NAME: 'analytics',
    PORT: 9002,
    SCOPE: 'analytics',
    MODULE: './Analytics',
  },
  USER_PROFILE: {
    NAME: 'user-profile',
    PORT: 9003,
    SCOPE: 'userProfile',
    MODULE: './UserProfile',
  },
} as const;

// API Configuration
export const API_CONFIG = {
  TIMEOUT: 10000,
  RETRY_ATTEMPTS: 3,
  RETRY_DELAY: 1000,
} as const;

// Validation rules
export const VALIDATION = {
  TODO_TITLE_MAX_LENGTH: 100,
  TODO_DESCRIPTION_MAX_LENGTH: 500,
  CATEGORY_NAME_MAX_LENGTH: 50,
  USER_NAME_MAX_LENGTH: 100,
} as const;
```

### 1.1.6: Create Utility Functions

**File**: `packages/shared/src/utils/index.ts`
```typescript
import { TodoItem } from '../types';

// UUID generator
export function generateId(): string {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
    const r = Math.random() * 16 | 0;
    const v = c == 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
}

// Date utilities
export function formatDate(date: Date): string {
  return date.toISOString().split('T')[0];
}

export function isDatePast(date: Date): boolean {
  return date < new Date();
}

export function getDaysUntil(date: Date): number {
  const today = new Date();
  const diffTime = date.getTime() - today.getTime();
  return Math.ceil(diffTime / (1000 * 60 * 60 * 24));
}

// Todo utilities
export function createTodoItem(
  title: string,
  options: Partial<Omit<TodoItem, 'id' | 'createdAt' | 'updatedAt'>> = {}
): TodoItem {
  const now = new Date();
  return {
    id: generateId(),
    title,
    description: options.description || '',
    completed: options.completed || false,
    category: options.category,
    priority: options.priority || 'medium',
    dueDate: options.dueDate,
    createdAt: now,
    updatedAt: now,
  };
}

export function updateTodoItem(
  todo: TodoItem,
  updates: Partial<Omit<TodoItem, 'id' | 'createdAt'>>
): TodoItem {
  return {
    ...todo,
    ...updates,
    updatedAt: new Date(),
  };
}

export function sortTodosByPriority(todos: TodoItem[]): TodoItem[] {
  const priorityOrder = { high: 3, medium: 2, low: 1 };
  return [...todos].sort((a, b) => priorityOrder[b.priority] - priorityOrder[a.priority]);
}

export function sortTodosByDueDate(todos: TodoItem[]): TodoItem[] {
  return [...todos].sort((a, b) => {
    if (!a.dueDate && !b.dueDate) return 0;
    if (!a.dueDate) return 1;
    if (!b.dueDate) return -1;
    return a.dueDate.getTime() - b.dueDate.getTime();
  });
}

export function filterTodosByStatus(todos: TodoItem[], completed: boolean): TodoItem[] {
  return todos.filter(todo => todo.completed === completed);
}

export function filterTodosByCategory(todos: TodoItem[], category: string): TodoItem[] {
  return todos.filter(todo => todo.category === category);
}

export function searchTodos(todos: TodoItem[], query: string): TodoItem[] {
  const lowercaseQuery = query.toLowerCase();
  return todos.filter(todo =>
    todo.title.toLowerCase().includes(lowercaseQuery) ||
    (todo.description && todo.description.toLowerCase().includes(lowercaseQuery))
  );
}

// Validation utilities
export function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

export function validateTodoTitle(title: string): boolean {
  return title.trim().length > 0 && title.length <= 100;
}

export function validateCategory(category: string): boolean {
  return category.trim().length > 0 && category.length <= 50;
}

// Performance utilities
export function debounce<T extends (...args: any[]) => void>(
  func: T,
  wait: number
): (...args: Parameters<T>) => void {
  let timeout: NodeJS.Timeout;
  return (...args: Parameters<T>) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(null, args), wait);
  };
}

export function throttle<T extends (...args: any[]) => void>(
  func: T,
  limit: number
): (...args: Parameters<T>) => void {
  let inThrottle: boolean;
  return (...args: Parameters<T>) => {
    if (!inThrottle) {
      func.apply(null, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Error handling utilities
export function createError(message: string, code?: string): Error {
  const error = new Error(message);
  if (code) {
    (error as any).code = code;
  }
  return error;
}

export function isNetworkError(error: any): boolean {
  return error instanceof TypeError && error.message.includes('fetch');
}

// Local Storage utilities
export function getStorageUsage(): { used: number; available: number } {
  let used = 0;
  for (const key in localStorage) {
    if (localStorage.hasOwnProperty(key)) {
      used += localStorage[key].length + key.length;
    }
  }

  const available = 5 * 1024 * 1024; // 5MB typical limit
  return { used, available };
}

export function clearOldData(maxAge: number = 30 * 24 * 60 * 60 * 1000): void {
  const now = Date.now();
  for (const key in localStorage) {
    try {
      const item = JSON.parse(localStorage[key]);
      if (item.timestamp && (now - item.timestamp > maxAge)) {
        localStorage.removeItem(key);
      }
    } catch {
      // Ignore items that aren't JSON
    }
  }
}
```

### 1.1.7: Create Package Index

**File**: `packages/shared/src/index.ts`
```typescript
// Event system
export { EventBus, eventBus } from './events/EventBus';
export * from './events/types';

// Storage system
export { LocalStorageService, storageService } from './storage/LocalStorageService';
export * from './storage/types';

// Types
export * from './types';

// Constants
export * from './constants';

// Utilities
export * from './utils';

// Package info
export const PACKAGE_VERSION = '1.0.0';
export const PACKAGE_NAME = '@todo-mfe/shared';
```

### 1.1.8: Write Tests

**File**: `packages/shared/src/__tests__/EventBus.test.ts`
```typescript
import { EventBus } from '../events/EventBus';
import { TaskCreatedEvent } from '../events/types';

describe('EventBus', () => {
  let eventBus: EventBus;

  beforeEach(() => {
    eventBus = EventBus.getInstance();
    eventBus.clear();
  });

  it('should be a singleton', () => {
    const instance1 = EventBus.getInstance();
    const instance2 = EventBus.getInstance();
    expect(instance1).toBe(instance2);
  });

  it('should emit and receive events', () => {
    const handler = jest.fn();
    eventBus.on('task:created', handler);

    const event: TaskCreatedEvent = {
      type: 'task:created',
      payload: {
        id: '1',
        title: 'Test Task',
        completed: false,
        priority: 'medium',
        createdAt: new Date(),
        updatedAt: new Date(),
      },
      timestamp: Date.now(),
      source: 'test',
    };

    eventBus.emit(event);
    expect(handler).toHaveBeenCalledWith(event);
  });

  it('should unsubscribe correctly', () => {
    const handler = jest.fn();
    const unsubscribe = eventBus.on('task:created', handler);

    unsubscribe();

    const event: TaskCreatedEvent = {
      type: 'task:created',
      payload: {
        id: '1',
        title: 'Test Task',
        completed: false,
        priority: 'medium',
        createdAt: new Date(),
        updatedAt: new Date(),
      },
      timestamp: Date.now(),
      source: 'test',
    };

    eventBus.emit(event);
    expect(handler).not.toHaveBeenCalled();
  });

  it('should handle multiple listeners', () => {
    const handler1 = jest.fn();
    const handler2 = jest.fn();

    eventBus.on('task:created', handler1);
    eventBus.on('task:created', handler2);

    const event: TaskCreatedEvent = {
      type: 'task:created',
      payload: {
        id: '1',
        title: 'Test Task',
        completed: false,
        priority: 'medium',
        createdAt: new Date(),
        updatedAt: new Date(),
      },
      timestamp: Date.now(),
      source: 'test',
    };

    eventBus.emit(event);
    expect(handler1).toHaveBeenCalledWith(event);
    expect(handler2).toHaveBeenCalledWith(event);
  });

  it('should get listener count', () => {
    const handler1 = jest.fn();
    const handler2 = jest.fn();

    expect(eventBus.getListenerCount('task:created')).toBe(0);

    eventBus.on('task:created', handler1);
    expect(eventBus.getListenerCount('task:created')).toBe(1);

    eventBus.on('task:created', handler2);
    expect(eventBus.getListenerCount('task:created')).toBe(2);
  });
});
```

**File**: `packages/shared/src/__tests__/LocalStorageService.test.ts`
```typescript
import { LocalStorageService } from '../storage/LocalStorageService';

// Mock localStorage
const localStorageMock = (() => {
  let store: Record<string, string> = {};
  return {
    getItem: jest.fn((key: string) => store[key] || null),
    setItem: jest.fn((key: string, value: string) => {
      store[key] = value;
    }),
    removeItem: jest.fn((key: string) => {
      delete store[key];
    }),
    clear: jest.fn(() => {
      store = {};
    }),
    get length() {
      return Object.keys(store).length;
    },
    key: jest.fn((index: number) => Object.keys(store)[index] || null),
  };
})();

Object.defineProperty(window, 'localStorage', {
  value: localStorageMock,
});

describe('LocalStorageService', () => {
  let storageService: LocalStorageService;

  beforeEach(() => {
    storageService = new LocalStorageService({ prefix: 'test:' });
    localStorageMock.clear();
    jest.clearAllMocks();
  });

  it('should set and get items', () => {
    const testData = { name: 'test', value: 123 };
    storageService.set('testKey', testData);

    expect(localStorageMock.setItem).toHaveBeenCalledWith('test:testKey', JSON.stringify(testData));

    const retrieved = storageService.get('testKey');
    expect(retrieved).toEqual(testData);
  });

  it('should return null for non-existent items', () => {
    const result = storageService.get('nonExistent');
    expect(result).toBeNull();
  });

  it('should remove items', () => {
    storageService.set('testKey', 'testValue');
    storageService.remove('testKey');

    expect(localStorageMock.removeItem).toHaveBeenCalledWith('test:testKey');
    expect(storageService.get('testKey')).toBeNull();
  });

  it('should check if items exist', () => {
    expect(storageService.exists('testKey')).toBe(false);

    storageService.set('testKey', 'testValue');
    expect(storageService.exists('testKey')).toBe(true);
  });

  it('should export and import data', () => {
    const data = {
      key1: 'value1',
      key2: { nested: 'value2' },
      key3: [1, 2, 3],
    };

    Object.entries(data).forEach(([key, value]) => {
      storageService.set(key, value);
    });

    const exported = storageService.exportData();
    expect(exported).toEqual(data);

    storageService.clear();
    storageService.importData(exported);

    Object.entries(data).forEach(([key, value]) => {
      expect(storageService.get(key)).toEqual(value);
    });
  });
});
```

### 1.1.9: Validation for Step 1.1

```bash
# Navigate to shared package
cd packages/shared

# Install dependencies
npm install

# Run tests
npm test

# Build package
npm run build

# Verify TypeScript compilation
npx tsc --noEmit

# Run linting
npm run lint
```

**Expected Results:**
- ✅ All tests pass with >80% coverage
- ✅ TypeScript compilation successful
- ✅ No linting errors
- ✅ `dist/` folder created with compiled JS and type definitions

---

## 🚀 Step 1.2: Setup Development Infrastructure

### 1.2.1: Create Webpack Configuration Templates

**File**: `tools/webpack/webpack.base.js`
```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = (projectRoot, options = {}) => ({
  mode: process.env.NODE_ENV || 'development',
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx'],
    alias: {
      '@shared': path.resolve(__dirname, '../../packages/shared/src'),
    },
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.(png|jpe?g|gif|svg)$/i,
        type: 'asset/resource',
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(projectRoot, 'src/index.html'),
      title: options.title || 'Microfrontend App',
    }),
  ],
  devServer: {
    port: options.port || 3000,
    historyApiFallback: true,
    hot: true,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, PATCH, OPTIONS',
      'Access-Control-Allow-Headers': 'X-Requested-With, content-type, Authorization',
    },
  },
});
```

**File**: `tools/webpack/webpack.mf.js`
```javascript
const ModuleFederationPlugin = require('@module-federation/webpack');
const baseConfig = require('./webpack.base');

module.exports = (projectRoot, federationConfig) => {
  const base = baseConfig(projectRoot, federationConfig);

  return {
    ...base,
    mode: process.env.NODE_ENV || 'development',
    plugins: [
      ...base.plugins,
      new ModuleFederationPlugin({
        ...federationConfig,
        shared: {
          react: {
            singleton: true,
            requiredVersion: '^18.0.0',
          },
          'react-dom': {
            singleton: true,
            requiredVersion: '^18.0.0',
          },
          '@todo-mfe/shared': {
            singleton: true,
          },
          ...federationConfig.shared,
        },
      }),
    ],
  };
};
```

### 1.2.2: Create Development Scripts

**File**: `tools/scripts/dev-setup.js`
```javascript
#!/usr/bin/env node

const { execSync } = require('child_process');
const fs = require('fs');
const path = require('path');

const MFE_CONFIGS = [
  { name: 'shell', port: 9000 },
  { name: 'task-manager', port: 9001 },
  { name: 'analytics', port: 9002 },
  { name: 'user-profile', port: 9003 },
];

function installDependencies() {
  console.log('📦 Installing dependencies...');

  // Install root dependencies
  execSync('npm install', { stdio: 'inherit' });

  // Install shared package dependencies
  execSync('npm install', {
    cwd: 'packages/shared',
    stdio: 'inherit'
  });

  console.log('✅ Dependencies installed');
}

function buildSharedPackage() {
  console.log('🔨 Building shared package...');
  execSync('npm run build', {
    cwd: 'packages/shared',
    stdio: 'inherit'
  });
  console.log('✅ Shared package built');
}

function checkPorts() {
  console.log('🔍 Checking port availability...');

  const { execSync } = require('child_process');

  MFE_CONFIGS.forEach(({ name, port }) => {
    try {
      execSync(`netstat -an | grep ${port}`, { stdio: 'pipe' });
      console.warn(`⚠️  Port ${port} (${name}) is already in use`);
    } catch {
      console.log(`✅ Port ${port} (${name}) is available`);
    }
  });
}

function createEnvFiles() {
  console.log('⚙️  Creating environment files...');

  const envContent = `NODE_ENV=development
MFE_SHELL_URL=http://localhost:9000
MFE_TASK_MANAGER_URL=http://localhost:9001
MFE_ANALYTICS_URL=http://localhost:9002
MFE_USER_PROFILE_URL=http://localhost:9003
`;

  MFE_CONFIGS.forEach(({ name }) => {
    const packagePath = `packages/${name}`;
    if (fs.existsSync(packagePath)) {
      fs.writeFileSync(path.join(packagePath, '.env'), envContent);
      console.log(`✅ Created .env for ${name}`);
    }
  });
}

function main() {
  console.log('🚀 Setting up development environment...\n');

  try {
    installDependencies();
    buildSharedPackage();
    checkPorts();
    createEnvFiles();

    console.log('\n🎉 Development environment setup complete!');
    console.log('\nNext steps:');
    console.log('1. Run: npm run dev:all');
    console.log('2. Open: http://localhost:9000');

  } catch (error) {
    console.error('❌ Setup failed:', error.message);
    process.exit(1);
  }
}

if (require.main === module) {
  main();
}

module.exports = { installDependencies, buildSharedPackage, checkPorts, createEnvFiles };
```

**File**: `tools/scripts/start-all.js`
```javascript
#!/usr/bin/env node

const { spawn } = require('child_process');
const fs = require('fs');

const MFE_CONFIGS = [
  { name: 'shell', port: 9000, color: '\x1b[36m' }, // Cyan
  { name: 'task-manager', port: 9001, color: '\x1b[32m' }, // Green
  { name: 'analytics', port: 9002, color: '\x1b[33m' }, // Yellow
  { name: 'user-profile', port: 9003, color: '\x1b[35m' }, // Magenta
];

const RESET_COLOR = '\x1b[0m';

function startMFE({ name, port, color }) {
  const packagePath = `packages/${name}`;

  if (!fs.existsSync(packagePath)) {
    console.log(`${color}⚠️  Package ${name} not found, skipping...${RESET_COLOR}`);
    return null;
  }

  console.log(`${color}🚀 Starting ${name} on port ${port}...${RESET_COLOR}`);

  const process = spawn('npm', ['run', 'dev'], {
    cwd: packagePath,
    stdio: 'pipe',
  });

  process.stdout.on('data', (data) => {
    const output = data.toString().trim();
    if (output) {
      console.log(`${color}[${name}]${RESET_COLOR} ${output}`);
    }
  });

  process.stderr.on('data', (data) => {
    const output = data.toString().trim();
    if (output) {
      console.error(`${color}[${name} ERROR]${RESET_COLOR} ${output}`);
    }
  });

  process.on('close', (code) => {
    if (code !== 0) {
      console.error(`${color}❌ ${name} exited with code ${code}${RESET_COLOR}`);
    } else {
      console.log(`${color}✅ ${name} stopped gracefully${RESET_COLOR}`);
    }
  });

  return process;
}

function main() {
  console.log('🎯 Starting all microfrontends...\n');

  const processes = MFE_CONFIGS.map(startMFE).filter(Boolean);

  if (processes.length === 0) {
    console.log('❌ No microfrontends found to start');
    process.exit(1);
  }

  console.log(`\n✨ Started ${processes.length} microfrontends`);
  console.log('📊 Access the application at: http://localhost:9000');
  console.log('🛑 Press Ctrl+C to stop all services\n');

  // Handle shutdown
  process.on('SIGINT', () => {
    console.log('\n🛑 Shutting down all services...');
    processes.forEach((proc, index) => {
      proc.kill('SIGINT');
      console.log(`✅ Stopped ${MFE_CONFIGS[index].name}`);
    });
    process.exit(0);
  });
}

if (require.main === module) {
  main();
}

module.exports = { startMFE };
```

### 1.2.3: Create Docker Configuration

**File**: `docker/base.Dockerfile`
```dockerfile
# Base Node.js image for all microfrontends
FROM node:18-alpine AS base

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Development stage
FROM base AS development

# Install all dependencies including dev
RUN npm ci

# Copy source code
COPY . .

# Expose port (will be overridden)
EXPOSE 3000

# Start development server
CMD ["npm", "run", "dev"]

# Build stage
FROM base AS build

# Install all dependencies
RUN npm ci

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Production stage
FROM nginx:alpine AS production

# Copy nginx configuration
COPY docker/nginx.conf /etc/nginx/nginx.conf

# Copy built application
COPY --from=build /app/dist /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

**File**: `docker/nginx.conf`
```nginx
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;

    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;
        index index.html;

        # SPA routing
        location / {
            try_files $uri $uri/ /index.html;
        }

        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }

        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Referrer-Policy "no-referrer-when-downgrade" always;
    }
}
```

**File**: `docker-compose.yml`
```yaml
version: '3.8'

services:
  shell:
    build:
      context: .
      dockerfile: docker/base.Dockerfile
      target: development
      args:
        - PACKAGE_PATH=packages/shell
    ports:
      - "9000:9000"
    volumes:
      - ./packages/shell:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - PORT=9000
    depends_on:
      - shared

  task-manager:
    build:
      context: .
      dockerfile: docker/base.Dockerfile
      target: development
      args:
        - PACKAGE_PATH=packages/task-manager
    ports:
      - "9001:9001"
    volumes:
      - ./packages/task-manager:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - PORT=9001
    depends_on:
      - shared

  analytics:
    build:
      context: .
      dockerfile: docker/base.Dockerfile
      target: development
      args:
        - PACKAGE_PATH=packages/analytics
    ports:
      - "9002:9002"
    volumes:
      - ./packages/analytics:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - PORT=9002
    depends_on:
      - shared

  user-profile:
    build:
      context: .
      dockerfile: docker/base.Dockerfile
      target: development
      args:
        - PACKAGE_PATH=packages/user-profile
    ports:
      - "9003:9003"
    volumes:
      - ./packages/user-profile:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - PORT=9003
    depends_on:
      - shared

  shared:
    build:
      context: .
      dockerfile: docker/base.Dockerfile
      target: development
      args:
        - PACKAGE_PATH=packages/shared
    volumes:
      - ./packages/shared:/app
      - /app/node_modules
    command: ["npm", "run", "dev"]

networks:
  default:
    name: todo-mfe-network
```

### 1.2.4: Update Root Package Scripts

**Update**: `package.json` (add to scripts section)
```json
{
  "scripts": {
    "dev:setup": "node tools/scripts/dev-setup.js",
    "dev:all": "node tools/scripts/start-all.js",
    "dev:shell": "npm run dev --workspace=packages/shell",
    "dev:task-manager": "npm run dev --workspace=packages/task-manager",
    "dev:analytics": "npm run dev --workspace=packages/analytics",
    "dev:user-profile": "npm run dev --workspace=packages/user-profile",
    "build:shared": "npm run build --workspace=packages/shared",
    "build:shell": "npm run build --workspace=packages/shell",
    "build:task-manager": "npm run build --workspace=packages/task-manager",
    "build:analytics": "npm run build --workspace=packages/analytics",
    "build:user-profile": "npm run build --workspace=packages/user-profile",
    "docker:build": "docker-compose build",
    "docker:up": "docker-compose up",
    "docker:down": "docker-compose down",
    "docker:logs": "docker-compose logs -f"
  }
}
```

### 1.2.5: Create Environment Configuration

**File**: `.env.example`
```bash
# Development Environment Configuration
NODE_ENV=development

# Microfrontend URLs
MFE_SHELL_URL=http://localhost:9000
MFE_TASK_MANAGER_URL=http://localhost:9001
MFE_ANALYTICS_URL=http://localhost:9002
MFE_USER_PROFILE_URL=http://localhost:9003

# Feature Flags
FEATURE_ANALYTICS=true
FEATURE_EXPORT=true
FEATURE_COLLABORATION=false
FEATURE_DARK_MODE=true
FEATURE_OFFLINE_MODE=false

# API Configuration
API_BASE_URL=http://localhost:3001
API_TIMEOUT=10000
API_RETRY_ATTEMPTS=3

# Logging
LOG_LEVEL=debug
ENABLE_CONSOLE_LOGGING=true
```

### 1.2.6: Validation for Step 1.2

```bash
# Make scripts executable
chmod +x tools/scripts/*.js

# Run development setup
npm run dev:setup

# Verify Docker configuration
docker-compose config

# Test environment files creation
ls packages/*/env 2>/dev/null || echo "Environment files not created yet (expected for now)"

# Validate webpack configurations
node -e "console.log('Webpack configs loaded successfully'); require('./tools/webpack/webpack.base'); require('./tools/webpack/webpack.mf');"
```

**Expected Results:**
- ✅ Development scripts are executable
- ✅ Docker Compose configuration is valid
- ✅ Webpack configurations load without errors
- ✅ Environment example file exists

---

## 🎯 Phase 1 Validation Checklist

### Step 1.1 Validation:
- [ ] **Unit Tests**: All shared package tests pass
- [ ] **Coverage**: Test coverage >80%
- [ ] **Build**: `npm run build:shared` succeeds
- [ ] **Types**: TypeScript compilation successful
- [ ] **Linting**: No ESLint errors

### Step 1.2 Validation:
- [ ] **Scripts**: Development scripts execute without errors
- [ ] **Docker**: `docker-compose config` validates successfully
- [ ] **Webpack**: Configuration files load properly
- [ ] **Environment**: Sample environment files created

### Integration Tests:
```bash
# Test Event Bus integration
cd packages/shared && npm test -- EventBus.test.ts

# Test Storage Service integration
cd packages/shared && npm test -- LocalStorageService.test.ts

# Build and verify shared package
npm run build:shared
ls packages/shared/dist/

# Verify Docker setup
docker-compose build shared
```

---

## 📊 Success Metrics

Upon completion of Phase 1, you should have:

1. **Shared Package** (`@todo-mfe/shared`):
   - ✅ Event Bus for inter-MFE communication
   - ✅ Local Storage service with type safety
   - ✅ Complete TypeScript type definitions
   - ✅ Utility functions for common operations
   - ✅ >80% test coverage

2. **Development Infrastructure**:
   - ✅ Webpack configuration templates
   - ✅ Module Federation setup
   - ✅ Docker development environment
   - ✅ Development orchestration scripts

3. **Quality Assurance**:
   - ✅ All tests passing
   - ✅ No TypeScript errors
   - ✅ No linting violations
   - ✅ Documentation complete

---

## 🚀 Next Steps

With Phase 1 complete, you're ready to proceed to **Phase 2: Shell Container Application**, where you'll:

1. Create the shell container that hosts all microfrontends
2. Implement Module Federation host configuration
3. Set up routing and navigation
4. Create error boundaries and loading states
5. Implement MFE orchestration system

The shared infrastructure created in Phase 1 will be the foundation that all subsequent microfrontends will build upon, ensuring consistent communication and data management across the entire application.