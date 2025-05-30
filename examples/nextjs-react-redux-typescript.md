---
name: "Next.js React Redux TypeScript Development Guide"
description: "A comprehensive development guide for modern web applications using Next.js, React, Redux, TypeScript with best practices, conventions, and standards for scalable development"
category: "Full-Stack Framework"
author: "Agents.md Collection"
authorUrl: "https://github.com/gakeez/agents_md_collection"
tags:
  ["nextjs", "react", "redux", "typescript", "javascript", "tailwind", "shadcn"]
lastUpdated: "2024-12-19"
---

# Next.js React Redux TypeScript Development Guide

## Project Overview

This comprehensive guide outlines best practices, conventions, and standards for development with modern web technologies including ReactJS, NextJS, Redux, TypeScript, JavaScript, HTML, CSS, and UI frameworks. The guide emphasizes clean, maintainable, and scalable code following SOLID principles and functional programming patterns.

## Tech Stack

- **Frontend Framework**: Next.js 14+ with App Router
- **UI Library**: React 18+ with TypeScript
- **State Management**: Redux Toolkit
- **Styling**: Tailwind CSS + Shadcn UI + Radix UI
- **Form Handling**: React Hook Form + Zod validation
- **Data Sanitization**: DOMPurify
- **Internationalization**: next-i18next
- **Testing**: Jest + React Testing Library
- **Code Quality**: ESLint + Prettier + TypeScript strict mode

## Project Structure

```
nextjs-project/
├── public/                    # Static assets
├── src/
│   ├── app/                  # Next.js App Router pages
│   │   ├── globals.css
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/           # Reusable components
│   │   ├── ui/              # Shadcn UI components
│   │   └── common/          # Common components
│   ├── lib/                 # Utility libraries
│   │   ├── store/           # Redux store configuration
│   │   ├── utils.ts         # Utility functions
│   │   └── validations.ts   # Zod schemas
│   ├── hooks/               # Custom React hooks
│   ├── types/               # TypeScript type definitions
│   ├── styles/              # Global styles
│   └── constants/           # Application constants
├── tests/                   # Test files
├── docs/                    # Documentation
├── .env.example            # Environment variables template
├── next.config.js          # Next.js configuration
├── tailwind.config.js      # Tailwind CSS configuration
├── tsconfig.json           # TypeScript configuration
└── package.json
```

## Development Guidelines

### Development Philosophy

- Write clean, maintainable, and scalable code
- Follow SOLID principles
- Prefer functional and declarative programming patterns over imperative
- Emphasize type safety and static analysis
- Practice component-driven development

### Code Implementation Guidelines

#### Planning Phase

- Begin with step-by-step planning
- Write detailed pseudocode before implementation
- Document component architecture and data flow
- Consider edge cases and error scenarios

#### Code Style Standards

- Use tabs for indentation
- Use single quotes for strings (except to avoid escaping)
- Omit semicolons (unless required for disambiguation)
- Eliminate unused variables
- Add space after keywords
- Add space before function declaration parentheses
- Always use strict equality (===) instead of loose equality (==)
- Space infix operators
- Add space after commas
- Keep else statements on the same line as closing curly braces
- Use curly braces for multi-line if statements
- Always handle error parameters in callbacks
- Limit line length to 80 characters
- Use trailing commas in multiline object/array literals

### Naming Conventions

#### General Rules

- **PascalCase for**: Components, Type definitions, Interfaces
- **kebab-case for**: Directory names (e.g., components/auth-wizard), File names (e.g., user-profile.tsx)
- **camelCase for**: Variables, Functions, Methods, Hooks, Properties, Props
- **UPPERCASE for**: Environment variables, Constants, Global configurations

#### Specific Naming Patterns

- Prefix event handlers with 'handle': `handleClick`, `handleSubmit`
- Prefix boolean variables with verbs: `isLoading`, `hasError`, `canSubmit`
- Prefix custom hooks with 'use': `useAuth`, `useForm`
- Use complete words over abbreviations except for:
  - err (error)
  - req (request)
  - res (response)
  - props (properties)
  - ref (reference)

## Environment Setup

### Development Requirements

- Node.js >= 18.0.0
- npm >= 8.0.0 or yarn >= 1.22.0
- TypeScript >= 5.0.0

### Installation Steps

```bash
# 1. Create Next.js project with TypeScript
npx create-next-app@latest my-nextjs-app --typescript --tailwind --eslint --app

# 2. Navigate to project directory
cd my-nextjs-app

# 3. Install additional dependencies
npm install @reduxjs/toolkit react-redux
npm install @hookform/resolvers react-hook-form zod
npm install @radix-ui/react-* # Install specific Radix components
npm install dompurify next-i18next
npm install -D @types/dompurify

# 4. Install Shadcn UI
npx shadcn-ui@latest init

# 5. Start development server
npm run dev
```

### Environment Variables Configuration

```env
# .env.local
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:3001/api
DATABASE_URL=postgresql://username:password@localhost:5432/database
NEXTAUTH_SECRET=your-secret-key
NEXTAUTH_URL=http://localhost:3000
```

## Core Feature Implementation

### React Component Best Practices

#### Component Architecture

- Use functional components with TypeScript interfaces
- Define components using the function keyword
- Extract reusable logic into custom hooks
- Implement proper component composition
- Use React.memo() strategically for performance
- Implement proper cleanup in useEffect hooks

```tsx
// Example: User Profile Component
interface UserProfileProps {
  userId: string;
  onUpdate?: (user: User) => void;
}

export function UserProfile({ userId, onUpdate }: UserProfileProps) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [hasError, setHasError] = useState(false);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setIsLoading(true);
        const userData = await getUserById(userId);
        setUser(userData);
      } catch (error) {
        setHasError(true);
        console.error("Failed to fetch user:", error);
      } finally {
        setIsLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  if (isLoading) return <div>Loading...</div>;
  if (hasError) return <div>Error loading user</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

#### React Performance Optimization

- Use useCallback for memoizing callback functions
- Implement useMemo for expensive computations
- Avoid inline function definitions in JSX
- Implement code splitting using dynamic imports
- Implement proper key props in lists (avoid using index as key)

```tsx
import { memo, useMemo, useCallback } from "react";

interface UserListProps {
  users: User[];
  onUserSelect: (userId: string) => void;
}

export const UserList = memo(({ users, onUserSelect }: UserListProps) => {
  const sortedUsers = useMemo(() => {
    return users.sort((a, b) => a.name.localeCompare(b.name));
  }, [users]);

  const handleUserClick = useCallback(
    (userId: string) => {
      onUserSelect(userId);
    },
    [onUserSelect]
  );

  return (
    <div className="user-list">
      {sortedUsers.map((user) => (
        <div
          key={user.id}
          onClick={() => handleUserClick(user.id)}
          className="user-item"
        >
          {user.name}
        </div>
      ))}
    </div>
  );
});
```

### Next.js Best Practices

#### Core Concepts

- Utilize App Router for routing
- Implement proper metadata management
- Use proper caching strategies
- Implement proper error boundaries

#### Components and Features

- Use Next.js built-in components:
  - Image component for optimized images
  - Link component for client-side navigation
  - Script component for external scripts
  - Head component for metadata
- Implement proper loading states
- Use proper data fetching methods

#### Server Components

- Default to Server Components
- Use URL query parameters for data fetching and server state management
- Use 'use client' directive only when necessary:
  - Event listeners
  - Browser APIs
  - State management
  - Client-side-only libraries

```tsx
// Example: Server Component with data fetching
interface PostsPageProps {
  searchParams: { page?: string; category?: string };
}

export default async function PostsPage({ searchParams }: PostsPageProps) {
  const page = Number(searchParams.page) || 1;
  const category = searchParams.category || "all";

  const posts = await fetchPosts({ page, category });

  return (
    <div className="posts-page">
      <h1>Posts</h1>
      <PostsList posts={posts} />
      <Pagination currentPage={page} />
    </div>
  );
}
```

### TypeScript Implementation

- Enable strict mode
- Define clear interfaces for component props, state, and Redux state structure
- Use type guards to handle potential undefined or null values safely
- Apply generics to functions, actions, and slices where type flexibility is needed
- Utilize TypeScript utility types (Partial, Pick, Omit) for cleaner and reusable code
- Prefer interface over type for defining object structures, especially when extending
- Use mapped types for creating variations of existing types dynamically

```tsx
// Example: TypeScript interfaces and types
interface User {
  id: string;
  name: string;
  email: string;
  role: "admin" | "user" | "moderator";
  createdAt: Date;
  updatedAt: Date;
}

interface ApiResponse<T> {
  data: T;
  message: string;
  success: boolean;
}

type UserCreateInput = Omit<User, "id" | "createdAt" | "updatedAt">;
type UserUpdateInput = Partial<Pick<User, "name" | "email" | "role">>;

// Type guard example
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === "object" &&
    obj !== null &&
    "id" in obj &&
    "name" in obj &&
    "email" in obj
  );
}
```

## State Management

### Local State

- Use useState for component-level state
- Implement useReducer for complex state
- Use useContext for shared state
- Implement proper state initialization

### Global State with Redux Toolkit

- Use Redux Toolkit for global state
- Use createSlice to define state, reducers, and actions together
- Avoid using createReducer and createAction unless necessary
- Normalize state structure to avoid deeply nested data
- Use selectors to encapsulate state access
- Avoid large, all-encompassing slices; separate concerns by feature

```tsx
// Example: Redux slice with TypeScript
import { createSlice, PayloadAction } from "@reduxjs/toolkit";

interface UserState {
  users: User[];
  currentUser: User | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: UserState = {
  users: [],
  currentUser: null,
  isLoading: false,
  error: null,
};

const userSlice = createSlice({
  name: "user",
  initialState,
  reducers: {
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.isLoading = action.payload;
    },
    setUsers: (state, action: PayloadAction<User[]>) => {
      state.users = action.payload;
      state.isLoading = false;
      state.error = null;
    },
    setCurrentUser: (state, action: PayloadAction<User>) => {
      state.currentUser = action.payload;
    },
    setError: (state, action: PayloadAction<string>) => {
      state.error = action.payload;
      state.isLoading = false;
    },
    clearError: (state) => {
      state.error = null;
    },
  },
});

export const { setLoading, setUsers, setCurrentUser, setError, clearError } =
  userSlice.actions;
export default userSlice.reducer;

// Selectors
export const selectUsers = (state: RootState) => state.user.users;
export const selectCurrentUser = (state: RootState) => state.user.currentUser;
export const selectUserLoading = (state: RootState) => state.user.isLoading;
export const selectUserError = (state: RootState) => state.user.error;
```

## UI and Styling

### Component Libraries

- Use Shadcn UI for consistent, accessible component design
- Integrate Radix UI primitives for customizable, accessible UI elements
- Apply composition patterns to create modular, reusable components

### Styling Guidelines

- Use Tailwind CSS for utility-first, maintainable styling
- Design with mobile-first, responsive principles for flexibility across devices
- Implement dark mode using CSS variables or Tailwind's dark mode features
- Ensure color contrast ratios meet accessibility standards for readability
- Maintain consistent spacing values to establish visual harmony
- Define CSS variables for theme colors and spacing to support easy theming and maintainability

```tsx
// Example: Styled component with Tailwind and dark mode
interface CardProps {
  title: string;
  children: React.ReactNode;
  variant?: "default" | "outlined" | "filled";
}

export function Card({ title, children, variant = "default" }: CardProps) {
  const baseClasses = "rounded-lg p-6 transition-colors duration-200";
  const variantClasses = {
    default: "bg-white dark:bg-gray-800 shadow-md",
    outlined: "border border-gray-200 dark:border-gray-700",
    filled: "bg-gray-50 dark:bg-gray-900",
  };

  return (
    <div className={`${baseClasses} ${variantClasses[variant]}`}>
      <h3 className="text-lg font-semibold text-gray-900 dark:text-gray-100 mb-4">
        {title}
      </h3>
      <div className="text-gray-700 dark:text-gray-300">{children}</div>
    </div>
  );
}
```

## Testing Strategy

### Unit Testing

- Write thorough unit tests to validate individual functions and components
- Use Jest and React Testing Library for reliable and efficient testing of React components
- Follow patterns like Arrange-Act-Assert to ensure clarity and consistency in tests
- Mock external dependencies and API calls to isolate unit tests

### Integration Testing

- Focus on user workflows to ensure app functionality
- Set up and tear down test environments properly to maintain test independence
- Use snapshot testing selectively to catch unintended UI changes without over-relying on it
- Leverage testing utilities (e.g., screen in RTL) for cleaner and more readable tests

```tsx
// Example: Component testing with React Testing Library
import { render, screen, fireEvent, waitFor } from "@testing-library/react";
import { Provider } from "react-redux";
import { store } from "../lib/store";
import { UserProfile } from "../components/UserProfile";

const renderWithProvider = (component: React.ReactElement) => {
  return render(<Provider store={store}>{component}</Provider>);
};

describe("UserProfile Component", () => {
  test("displays user information correctly", async () => {
    const mockUser = {
      id: "1",
      name: "John Doe",
      email: "john@example.com",
    };

    renderWithProvider(<UserProfile userId="1" />);

    await waitFor(() => {
      expect(screen.getByText("John Doe")).toBeInTheDocument();
      expect(screen.getByText("john@example.com")).toBeInTheDocument();
    });
  });

  test("handles loading state", () => {
    renderWithProvider(<UserProfile userId="1" />);
    expect(screen.getByText("Loading...")).toBeInTheDocument();
  });
});
```

## Error Handling and Validation

### Form Validation

- Use Zod for schema validation
- Implement proper error messages
- Use proper form libraries (e.g., React Hook Form)

```tsx
// Example: Form validation with Zod and React Hook Form
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const userSchema = z.object({
  name: z.string().min(2, "Name must be at least 2 characters"),
  email: z.string().email("Invalid email address"),
  age: z.number().min(18, "Must be at least 18 years old"),
});

type UserFormData = z.infer<typeof userSchema>;

export function UserForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
  });

  const onSubmit = async (data: UserFormData) => {
    try {
      await createUser(data);
      // Handle success
    } catch (error) {
      // Handle error
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <input
          {...register("name")}
          placeholder="Name"
          className="w-full p-2 border rounded"
        />
        {errors.name && (
          <p className="text-red-500 text-sm">{errors.name.message}</p>
        )}
      </div>

      <div>
        <input
          {...register("email")}
          type="email"
          placeholder="Email"
          className="w-full p-2 border rounded"
        />
        {errors.email && (
          <p className="text-red-500 text-sm">{errors.email.message}</p>
        )}
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full p-2 bg-blue-500 text-white rounded disabled:opacity-50"
      >
        {isSubmitting ? "Submitting..." : "Submit"}
      </button>
    </form>
  );
}
```

### Error Boundaries

- Use error boundaries to catch and handle errors in React component trees gracefully
- Log caught errors to an external service (e.g., Sentry) for tracking and debugging
- Design user-friendly fallback UIs to display when errors occur, keeping users informed without breaking the app

```tsx
// Example: Error Boundary component
import { Component, ErrorInfo, ReactNode } from "react";

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  public state: State = {
    hasError: false,
  };

  public static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  public componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error("Uncaught error:", error, errorInfo);
    // Log to external service like Sentry
  }

  public render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary p-8 text-center">
          <h2 className="text-xl font-bold text-red-600 mb-4">
            Something went wrong
          </h2>
          <p className="text-gray-600 mb-4">
            We're sorry, but something unexpected happened.
          </p>
          <button
            onClick={() => this.setState({ hasError: false })}
            className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
          >
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## Performance Optimization

### Frontend Optimization

- Code splitting with dynamic imports
- Lazy loading for non-critical components
- Caching strategies for API responses
- Image optimization with Next.js Image component

```tsx
// Example: Code splitting and lazy loading
import { lazy, Suspense } from "react";

const LazyDashboard = lazy(() => import("./Dashboard"));
const LazySettings = lazy(() => import("./Settings"));

export function App() {
  return (
    <div className="app">
      <Suspense fallback={<div>Loading Dashboard...</div>}>
        <LazyDashboard />
      </Suspense>

      <Suspense fallback={<div>Loading Settings...</div>}>
        <LazySettings />
      </Suspense>
    </div>
  );
}
```

### Backend Optimization

- Database query optimization
- Caching mechanisms with Redis
- Load balancing strategies
- API response optimization

## Security Considerations

### Data Security

- Implement input sanitization to prevent XSS attacks
- Use DOMPurify for sanitizing HTML content
- Use proper authentication methods
- Validate all user inputs

```tsx
// Example: Input sanitization with DOMPurify
import DOMPurify from "dompurify";

interface SafeHtmlProps {
  html: string;
  className?: string;
}

export function SafeHtml({ html, className }: SafeHtmlProps) {
  const sanitizedHtml = DOMPurify.sanitize(html);

  return (
    <div
      className={className}
      dangerouslySetInnerHTML={{ __html: sanitizedHtml }}
    />
  );
}
```

### Authentication & Authorization

- Implement proper user authentication flow
- Use JWT tokens securely
- Implement role-based access control
- Secure API endpoints

## Accessibility (a11y)

### Core Requirements

- Use semantic HTML for meaningful structure
- Apply accurate ARIA attributes where needed
- Ensure full keyboard navigation support
- Manage focus order and visibility effectively
- Maintain accessible color contrast ratios
- Follow a logical heading hierarchy
- Make all interactive elements accessible
- Provide clear and accessible error feedback

```tsx
// Example: Accessible form component
export function AccessibleForm() {
  const [errors, setErrors] = useState<Record<string, string>>({});

  return (
    <form role="form" aria-labelledby="form-title">
      <h2 id="form-title">User Registration</h2>

      <div className="form-group">
        <label htmlFor="email" className="required">
          Email Address
        </label>
        <input
          id="email"
          type="email"
          required
          aria-describedby={errors.email ? "email-error" : undefined}
          aria-invalid={!!errors.email}
          className="form-input"
        />
        {errors.email && (
          <div id="email-error" role="alert" className="error-message">
            {errors.email}
          </div>
        )}
      </div>

      <button type="submit" className="submit-button">
        Register
        <span className="sr-only">(opens in new window)</span>
      </button>
    </form>
  );
}
```

## Internationalization (i18n)

### Implementation with next-i18next

- Use next-i18next for translations
- Implement proper locale detection
- Use proper number and date formatting
- Implement proper RTL support
- Use proper currency formatting

```tsx
// Example: i18n configuration
// next-i18next.config.js
module.exports = {
  i18n: {
    defaultLocale: "en",
    locales: ["en", "es", "fr", "de"],
  },
  reloadOnPrerender: process.env.NODE_ENV === "development",
};

// Example: Using translations in components
import { useTranslation } from "next-i18next";
import { GetStaticProps } from "next";
import { serverSideTranslations } from "next-i18next/serverSideTranslations";

export function WelcomePage() {
  const { t } = useTranslation("common");

  return (
    <div>
      <h1>{t("welcome.title")}</h1>
      <p>{t("welcome.description")}</p>
    </div>
  );
}

export const getStaticProps: GetStaticProps = async ({ locale }) => ({
  props: {
    ...(await serverSideTranslations(locale ?? "en", ["common"])),
  },
});
```

## Deployment Guide

### Build Process

```bash
# Build for production
npm run build

# Start production server
npm start

# Export static files (if using static export)
npm run export
```

### Environment Variables for Production

```env
# Production environment variables
NODE_ENV=production
NEXT_PUBLIC_APP_URL=https://yourdomain.com
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
DATABASE_URL=postgresql://user:password@host:port/database
REDIS_URL=redis://host:port
NEXTAUTH_SECRET=your-production-secret
NEXTAUTH_URL=https://yourdomain.com
```

### Deployment Configuration

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    appDir: true,
  },
  images: {
    domains: ["example.com", "cdn.example.com"],
    formats: ["image/webp", "image/avif"],
  },
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },
  async headers() {
    return [
      {
        source: "/(.*)",
        headers: [
          {
            key: "X-Content-Type-Options",
            value: "nosniff",
          },
          {
            key: "X-Frame-Options",
            value: "DENY",
          },
          {
            key: "X-XSS-Protection",
            value: "1; mode=block",
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

## Monitoring and Logging

### Application Monitoring

- Performance metrics tracking
- Error tracking with Sentry
- User behavior analytics
- Core Web Vitals monitoring

### Log Management

- Structured logging with appropriate log levels
- Centralized log storage
- Error alerting and notification

```tsx
// Example: Error tracking setup
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
});

// Custom error logging utility
export const logger = {
  error: (message: string, error?: Error, context?: Record<string, any>) => {
    console.error(message, error, context);
    Sentry.captureException(error || new Error(message), {
      extra: context,
    });
  },
  warn: (message: string, context?: Record<string, any>) => {
    console.warn(message, context);
  },
  info: (message: string, context?: Record<string, any>) => {
    console.info(message, context);
  },
};
```

## Common Issues

### Issue 1: Hydration Mismatch Errors

**Solution**:

- Ensure server and client render the same content
- Use `useEffect` for client-only code
- Use `dynamic` imports with `ssr: false` for client-only components
- Check for differences in date/time formatting between server and client

### Issue 2: Performance Issues with Large Lists

**Solution**:

- Implement virtualization for large datasets
- Use pagination or infinite scrolling
- Optimize re-renders with `React.memo` and `useMemo`
- Consider server-side filtering and sorting

### Issue 3: TypeScript Type Errors in Production Build

**Solution**:

- Enable strict mode in TypeScript configuration
- Fix all type errors before deployment
- Use proper type definitions for third-party libraries
- Implement proper error boundaries for runtime type issues

### Issue 4: SEO and Meta Tags Not Working

**Solution**:

- Use Next.js `Metadata` API in App Router
- Implement proper Open Graph tags
- Ensure meta tags are rendered server-side
- Test with social media debuggers

## Reference Resources

- [Next.js Official Documentation](https://nextjs.org/docs)
- [React Official Documentation](https://react.dev/)
- [TypeScript Official Documentation](https://www.typescriptlang.org/)
- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Shadcn UI Documentation](https://ui.shadcn.com/)
- [React Hook Form Documentation](https://react-hook-form.com/)
- [Zod Documentation](https://zod.dev/)
- [React Testing Library Documentation](https://testing-library.com/docs/react-testing-library/intro/)

## Changelog

### v1.0.0 (2024-12-19)

- Initial release of Next.js React Redux TypeScript development guide
- Comprehensive coverage of modern web development practices
- Included examples for all major concepts and patterns
- Added accessibility, security, and performance optimization guidelines

---

**Note**: This guide is based on the latest stable versions of Next.js, React, and TypeScript. Please adjust configurations and examples according to your specific project requirements and the versions you are using. Always refer to the official documentation for the most up-to-date information.
