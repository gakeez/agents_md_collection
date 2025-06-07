---
name: "TypeScript React Native Expo Mobile Development Guide"
description: "A comprehensive development guide for building modern mobile applications using TypeScript, React Native, and Expo with best practices, performance optimization, and cross-platform compatibility"
category: "Mobile Development"
author: "Agents.md Collection"
authorUrl: "https://github.com/gakeez/agents_md_collection"
tags:
  [
    "typescript",
    "react-native",
    "expo",
    "mobile",
    "ios",
    "android",
    "cross-platform",
  ]
lastUpdated: "2024-12-19"
---

# TypeScript React Native Expo Mobile Development Guide

## Project Overview

This comprehensive guide outlines best practices for developing modern mobile applications using TypeScript, React Native, and Expo. It emphasizes concise, technical TypeScript code with accurate examples, functional and declarative programming patterns, and cross-platform mobile development excellence. The guide focuses on performance optimization, accessibility, and following Expo's managed workflow for streamlined development and deployment.

## Tech Stack

- **Framework**: React Native with Expo SDK 49+
- **Language**: TypeScript with strict mode
- **Navigation**: React Navigation 6+ / Expo Router
- **State Management**: React Context + useReducer / Zustand / Redux Toolkit
- **Data Fetching**: React Query (TanStack Query)
- **Styling**: Styled-components / NativeWind (Tailwind CSS)
- **Animation**: React Native Reanimated 3+ / React Native Gesture Handler
- **Testing**: Jest + React Native Testing Library + Detox
- **Storage**: Expo SecureStore / React Native Encrypted Storage

## Project Structure

```
expo-app/
├── app/                          # Expo Router pages (if using)
│   ├── (tabs)/
│   ├── _layout.tsx
│   └── index.tsx
├── src/
│   ├── components/
│   │   ├── ui/                   # Reusable UI components
│   │   ├── forms/                # Form components
│   │   └── layout/               # Layout components
│   ├── screens/                  # Screen components
│   ├── navigation/               # Navigation configuration
│   ├── hooks/                    # Custom hooks
│   ├── services/                 # API services
│   ├── store/                    # State management
│   ├── utils/                    # Utility functions
│   ├── types/                    # TypeScript type definitions
│   ├── constants/                # App constants
│   └── assets/                   # Images, fonts, etc.
├── __tests__/                    # Test files
├── app.json                      # Expo configuration
├── babel.config.js
├── metro.config.js
├── tsconfig.json
└── package.json
```

## Development Guidelines

### Code Style and Structure

- Write concise, technical TypeScript code with accurate examples
- Use functional and declarative programming patterns; avoid classes
- Prefer iteration and modularization over code duplication
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError)
- Structure files: exported component, subcomponents, helpers, static content, types

### Naming Conventions

- Use lowercase with dashes for directories (e.g., components/auth-wizard)
- Favor named exports for components
- Use PascalCase for component names
- Use camelCase for functions, variables, and props

### TypeScript Usage

```typescript
// Use interfaces over types
interface UserProfile {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  preferences: UserPreferences;
}

interface UserPreferences {
  theme: "light" | "dark" | "system";
  notifications: boolean;
  language: string;
}

// Avoid enums; use maps instead
const THEME_OPTIONS = {
  LIGHT: "light",
  DARK: "dark",
  SYSTEM: "system",
} as const;

type ThemeOption = (typeof THEME_OPTIONS)[keyof typeof THEME_OPTIONS];

// Functional components with TypeScript interfaces
interface ProfileCardProps {
  user: UserProfile;
  onEdit: (userId: string) => void;
  isLoading?: boolean;
}

export function ProfileCard({
  user,
  onEdit,
  isLoading = false,
}: ProfileCardProps) {
  const handleEditPress = () => {
    if (!isLoading) onEdit(user.id);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.name}>{user.name}</Text>
      <Text style={styles.email}>{user.email}</Text>
      <TouchableOpacity onPress={handleEditPress} disabled={isLoading}>
        <Text>Edit Profile</Text>
      </TouchableOpacity>
    </View>
  );
}
```

## Environment Setup

### Development Requirements

- Node.js >= 18.0.0
- npm >= 8.0.0 or yarn >= 1.22.0
- Expo CLI >= 6.0.0
- TypeScript >= 5.0.0
- iOS Simulator (for iOS development)
- Android Studio + Android SDK (for Android development)

### Installation Steps

```bash
# 1. Install Expo CLI globally
npm install -g @expo/cli

# 2. Create new Expo project with TypeScript
npx create-expo-app MyApp --template

# 3. Navigate to project directory
cd MyApp

# 4. Install additional dependencies
npx expo install expo-router expo-constants expo-linking
npx expo install react-native-safe-area-context react-native-screens
npx expo install @react-navigation/native @react-navigation/stack
npx expo install react-native-reanimated react-native-gesture-handler
npx expo install @tanstack/react-query expo-secure-store
npx expo install react-native-svg expo-image

# 5. Install development dependencies
npm install -D @types/react @types/react-native
npm install -D jest @testing-library/react-native @testing-library/jest-native
npm install -D detox

# 6. Start development server
npx expo start
```

### TypeScript Configuration

```json
// tsconfig.json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/components/*": ["src/components/*"],
      "@/screens/*": ["src/screens/*"],
      "@/hooks/*": ["src/hooks/*"],
      "@/utils/*": ["src/utils/*"],
      "@/types/*": ["src/types/*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx", ".expo/types/**/*.ts", "expo-env.d.ts"]
}
```

### Expo Configuration

```json
// app.json
{
  "expo": {
    "name": "MyApp",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": ["**/*"],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yourcompany.myapp"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      },
      "package": "com.yourcompany.myapp"
    },
    "web": {
      "favicon": "./assets/favicon.png"
    },
    "plugins": [
      "expo-router",
      "expo-secure-store",
      [
        "expo-image-picker",
        {
          "photosPermission": "The app accesses your photos to let you share them."
        }
      ]
    ],
    "experiments": {
      "typedRoutes": true
    }
  }
}
```

## Core Feature Implementation

### Component Architecture

```typescript
// src/components/ui/Button.tsx
import React from "react";
import {
  TouchableOpacity,
  Text,
  ActivityIndicator,
  StyleSheet,
} from "react-native";
import { useColorScheme } from "react-native";

interface ButtonProps {
  title: string;
  onPress: () => void;
  variant?: "primary" | "secondary" | "outline";
  size?: "small" | "medium" | "large";
  isLoading?: boolean;
  disabled?: boolean;
  testID?: string;
}

export function Button({
  title,
  onPress,
  variant = "primary",
  size = "medium",
  isLoading = false,
  disabled = false,
  testID,
}: ButtonProps) {
  const colorScheme = useColorScheme();
  const isDark = colorScheme === "dark";

  const buttonStyles = [
    styles.base,
    styles[size],
    styles[variant],
    isDark && styles.dark,
    (disabled || isLoading) && styles.disabled,
  ];

  const textStyles = [
    styles.text,
    styles[`${variant}Text`],
    isDark && styles.darkText,
  ];

  return (
    <TouchableOpacity
      style={buttonStyles}
      onPress={onPress}
      disabled={disabled || isLoading}
      testID={testID}
      accessibilityRole="button"
      accessibilityState={{ disabled: disabled || isLoading }}
    >
      {isLoading ? (
        <ActivityIndicator color={variant === "primary" ? "#fff" : "#007AFF"} />
      ) : (
        <Text style={textStyles}>{title}</Text>
      )}
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  base: {
    borderRadius: 8,
    alignItems: "center",
    justifyContent: "center",
    flexDirection: "row",
  },
  small: {
    paddingHorizontal: 12,
    paddingVertical: 6,
    minHeight: 32,
  },
  medium: {
    paddingHorizontal: 16,
    paddingVertical: 12,
    minHeight: 44,
  },
  large: {
    paddingHorizontal: 24,
    paddingVertical: 16,
    minHeight: 56,
  },
  primary: {
    backgroundColor: "#007AFF",
  },
  secondary: {
    backgroundColor: "#F2F2F7",
  },
  outline: {
    backgroundColor: "transparent",
    borderWidth: 1,
    borderColor: "#007AFF",
  },
  disabled: {
    opacity: 0.5,
  },
  dark: {
    backgroundColor: "#1C1C1E",
  },
  text: {
    fontSize: 16,
    fontWeight: "600",
  },
  primaryText: {
    color: "#FFFFFF",
  },
  secondaryText: {
    color: "#000000",
  },
  outlineText: {
    color: "#007AFF",
  },
  darkText: {
    color: "#FFFFFF",
  },
});
```

### Safe Area Management

```typescript
// src/components/layout/SafeAreaWrapper.tsx
import React from "react";
import { SafeAreaProvider, SafeAreaView } from "react-native-safe-area-context";
import { StatusBar } from "expo-status-bar";
import { useColorScheme } from "react-native";

interface SafeAreaWrapperProps {
  children: React.ReactNode;
  edges?: ("top" | "bottom" | "left" | "right")[];
}

export function SafeAreaWrapper({
  children,
  edges = ["top", "bottom"],
}: SafeAreaWrapperProps) {
  const colorScheme = useColorScheme();

  return (
    <SafeAreaProvider>
      <SafeAreaView style={{ flex: 1 }} edges={edges}>
        <StatusBar style={colorScheme === "dark" ? "light" : "dark"} />
        {children}
      </SafeAreaView>
    </SafeAreaProvider>
  );
}

// src/components/layout/SafeAreaScrollView.tsx
import React from "react";
import { ScrollView, ScrollViewProps } from "react-native";
import { useSafeAreaInsets } from "react-native-safe-area-context";

interface SafeAreaScrollViewProps extends ScrollViewProps {
  children: React.ReactNode;
}

export function SafeAreaScrollView({
  children,
  style,
  ...props
}: SafeAreaScrollViewProps) {
  const insets = useSafeAreaInsets();

  return (
    <ScrollView
      style={[{ paddingTop: insets.top, paddingBottom: insets.bottom }, style]}
      contentInsetAdjustmentBehavior="automatic"
      {...props}
    >
      {children}
    </ScrollView>
  );
}
```

### Navigation Setup

```typescript
// src/navigation/AppNavigator.tsx
import React from "react";
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";
import { Ionicons } from "@expo/vector-icons";

import { HomeScreen } from "@/screens/HomeScreen";
import { ProfileScreen } from "@/screens/ProfileScreen";
import { SettingsScreen } from "@/screens/SettingsScreen";
import { LoginScreen } from "@/screens/auth/LoginScreen";

export type RootStackParamList = {
  Main: undefined;
  Login: undefined;
  Profile: { userId: string };
};

export type TabParamList = {
  Home: undefined;
  Profile: undefined;
  Settings: undefined;
};

const Stack = createNativeStackNavigator<RootStackParamList>();
const Tab = createBottomTabNavigator<TabParamList>();

function TabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName: keyof typeof Ionicons.glyphMap;

          switch (route.name) {
            case "Home":
              iconName = focused ? "home" : "home-outline";
              break;
            case "Profile":
              iconName = focused ? "person" : "person-outline";
              break;
            case "Settings":
              iconName = focused ? "settings" : "settings-outline";
              break;
            default:
              iconName = "help-outline";
          }

          return <Ionicons name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: "#007AFF",
        tabBarInactiveTintColor: "gray",
        headerShown: false,
      })}
    >
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
      <Tab.Screen name="Settings" component={SettingsScreen} />
    </Tab.Navigator>
  );
}

export function AppNavigator() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Main">
        <Stack.Screen
          name="Main"
          component={TabNavigator}
          options={{ headerShown: false }}
        />
        <Stack.Screen
          name="Login"
          component={LoginScreen}
          options={{ presentation: "modal" }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

### State Management

```typescript
// src/store/AuthContext.tsx
import React, { createContext, useContext, useReducer, useEffect } from "react";
import * as SecureStore from "expo-secure-store";

interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthState {
  user: User | null;
  isLoading: boolean;
  isAuthenticated: boolean;
}

type AuthAction =
  | { type: "SET_LOADING"; payload: boolean }
  | { type: "SET_USER"; payload: User | null }
  | { type: "LOGOUT" };

const initialState: AuthState = {
  user: null,
  isLoading: true,
  isAuthenticated: false,
};

function authReducer(state: AuthState, action: AuthAction): AuthState {
  switch (action.type) {
    case "SET_LOADING":
      return { ...state, isLoading: action.payload };
    case "SET_USER":
      return {
        ...state,
        user: action.payload,
        isAuthenticated: !!action.payload,
        isLoading: false,
      };
    case "LOGOUT":
      return {
        ...state,
        user: null,
        isAuthenticated: false,
        isLoading: false,
      };
    default:
      return state;
  }
}

interface AuthContextType extends AuthState {
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  checkAuthStatus: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(authReducer, initialState);

  const login = async (email: string, password: string) => {
    dispatch({ type: "SET_LOADING", payload: true });

    try {
      // Simulate API call
      const response = await fetch("/api/login", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email, password }),
      });

      const data = await response.json();

      if (response.ok) {
        await SecureStore.setItemAsync("authToken", data.token);
        dispatch({ type: "SET_USER", payload: data.user });
      } else {
        throw new Error(data.message);
      }
    } catch (error) {
      dispatch({ type: "SET_LOADING", payload: false });
      throw error;
    }
  };

  const logout = async () => {
    await SecureStore.deleteItemAsync("authToken");
    dispatch({ type: "LOGOUT" });
  };

  const checkAuthStatus = async () => {
    try {
      const token = await SecureStore.getItemAsync("authToken");

      if (token) {
        // Validate token with API
        const response = await fetch("/api/me", {
          headers: { Authorization: `Bearer ${token}` },
        });

        if (response.ok) {
          const user = await response.json();
          dispatch({ type: "SET_USER", payload: user });
        } else {
          await SecureStore.deleteItemAsync("authToken");
          dispatch({ type: "SET_USER", payload: null });
        }
      } else {
        dispatch({ type: "SET_USER", payload: null });
      }
    } catch (error) {
      dispatch({ type: "SET_USER", payload: null });
    }
  };

  useEffect(() => {
    checkAuthStatus();
  }, []);

  const value: AuthContextType = {
    ...state,
    login,
    logout,
    checkAuthStatus,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error("useAuth must be used within an AuthProvider");
  }
  return context;
}
```

## Data Fetching and API Integration

### React Query Setup

```typescript
// src/services/api.ts
import { QueryClient } from "@tanstack/react-query";
import * as SecureStore from "expo-secure-store";

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 3,
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
    },
  },
});

class ApiClient {
  private baseURL = "https://api.example.com";

  private async getAuthToken(): Promise<string | null> {
    return await SecureStore.getItemAsync("authToken");
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const token = await this.getAuthToken();

    const config: RequestInit = {
      headers: {
        "Content-Type": "application/json",
        ...(token && { Authorization: `Bearer ${token}` }),
        ...options.headers,
      },
      ...options,
    };

    const response = await fetch(`${this.baseURL}${endpoint}`, config);

    if (!response.ok) {
      throw new Error(`API Error: ${response.status} ${response.statusText}`);
    }

    return response.json();
  }

  async get<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint);
  }

  async post<T>(endpoint: string, data: unknown): Promise<T> {
    return this.request<T>(endpoint, {
      method: "POST",
      body: JSON.stringify(data),
    });
  }

  async put<T>(endpoint: string, data: unknown): Promise<T> {
    return this.request<T>(endpoint, {
      method: "PUT",
      body: JSON.stringify(data),
    });
  }

  async delete<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, {
      method: "DELETE",
    });
  }
}

export const apiClient = new ApiClient();
```

### Custom Hooks for Data Fetching

```typescript
// src/hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { apiClient } from "@/services/api";

interface User {
  id: string;
  name: string;
  email: string;
  avatar?: string;
}

interface CreateUserData {
  name: string;
  email: string;
}

export function useUsers() {
  return useQuery({
    queryKey: ["users"],
    queryFn: () => apiClient.get<User[]>("/users"),
  });
}

export function useUser(userId: string) {
  return useQuery({
    queryKey: ["users", userId],
    queryFn: () => apiClient.get<User>(`/users/${userId}`),
    enabled: !!userId,
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (userData: CreateUserData) =>
      apiClient.post<User>("/users", userData),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["users"] });
    },
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, ...userData }: Partial<User> & { id: string }) =>
      apiClient.put<User>(`/users/${id}`, userData),
    onSuccess: (data) => {
      queryClient.setQueryData(["users", data.id], data);
      queryClient.invalidateQueries({ queryKey: ["users"] });
    },
  });
}
```

## Performance Optimization

### Image Optimization

```typescript
// src/components/ui/OptimizedImage.tsx
import React, { useState } from "react";
import { View, StyleSheet, ActivityIndicator } from "react-native";
import { Image } from "expo-image";

interface OptimizedImageProps {
  source: string | { uri: string };
  width: number;
  height: number;
  placeholder?: string;
  alt?: string;
  onLoad?: () => void;
  onError?: () => void;
}

export function OptimizedImage({
  source,
  width,
  height,
  placeholder,
  alt,
  onLoad,
  onError,
}: OptimizedImageProps) {
  const [isLoading, setIsLoading] = useState(true);
  const [hasError, setHasError] = useState(false);

  const handleLoad = () => {
    setIsLoading(false);
    onLoad?.();
  };

  const handleError = () => {
    setIsLoading(false);
    setHasError(true);
    onError?.();
  };

  return (
    <View style={[styles.container, { width, height }]}>
      <Image
        source={source}
        style={styles.image}
        placeholder={placeholder}
        contentFit="cover"
        transition={200}
        onLoad={handleLoad}
        onError={handleError}
        accessible={true}
        accessibilityLabel={alt}
      />
      {isLoading && (
        <View style={styles.loadingOverlay}>
          <ActivityIndicator size="small" color="#007AFF" />
        </View>
      )}
      {hasError && (
        <View style={styles.errorOverlay}>
          <Text style={styles.errorText}>Failed to load image</Text>
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    position: "relative",
    overflow: "hidden",
    borderRadius: 8,
  },
  image: {
    width: "100%",
    height: "100%",
  },
  loadingOverlay: {
    position: "absolute",
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    justifyContent: "center",
    alignItems: "center",
    backgroundColor: "rgba(0, 0, 0, 0.1)",
  },
  errorOverlay: {
    position: "absolute",
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    justifyContent: "center",
    alignItems: "center",
    backgroundColor: "#f5f5f5",
  },
  errorText: {
    fontSize: 12,
    color: "#666",
    textAlign: "center",
  },
});
```

### Memoization and Performance

```typescript
// src/components/UserList.tsx
import React, { useMemo, useCallback } from "react";
import { FlatList, View, Text, StyleSheet } from "react-native";
import { useUsers } from "@/hooks/useUsers";

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserListProps {
  searchQuery?: string;
  onUserPress: (userId: string) => void;
}

const UserItem = React.memo(
  ({ user, onPress }: { user: User; onPress: (id: string) => void }) => {
    const handlePress = useCallback(() => {
      onPress(user.id);
    }, [user.id, onPress]);

    return (
      <TouchableOpacity style={styles.userItem} onPress={handlePress}>
        <Text style={styles.userName}>{user.name}</Text>
        <Text style={styles.userEmail}>{user.email}</Text>
      </TouchableOpacity>
    );
  }
);

export function UserList({ searchQuery, onUserPress }: UserListProps) {
  const { data: users, isLoading, error } = useUsers();

  const filteredUsers = useMemo(() => {
    if (!users || !searchQuery) return users || [];

    return users.filter(
      (user) =>
        user.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
        user.email.toLowerCase().includes(searchQuery.toLowerCase())
    );
  }, [users, searchQuery]);

  const renderUser = useCallback(
    ({ item }: { item: User }) => (
      <UserItem user={item} onPress={onUserPress} />
    ),
    [onUserPress]
  );

  const keyExtractor = useCallback((item: User) => item.id, []);

  if (isLoading) {
    return (
      <View style={styles.centered}>
        <ActivityIndicator size="large" color="#007AFF" />
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.centered}>
        <Text style={styles.errorText}>Failed to load users</Text>
      </View>
    );
  }

  return (
    <FlatList
      data={filteredUsers}
      renderItem={renderUser}
      keyExtractor={keyExtractor}
      removeClippedSubviews={true}
      maxToRenderPerBatch={10}
      windowSize={10}
      initialNumToRender={10}
      getItemLayout={(data, index) => ({
        length: 80,
        offset: 80 * index,
        index,
      })}
    />
  );
}

const styles = StyleSheet.create({
  userItem: {
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: "#e0e0e0",
    height: 80,
    justifyContent: "center",
  },
  userName: {
    fontSize: 16,
    fontWeight: "600",
    marginBottom: 4,
  },
  userEmail: {
    fontSize: 14,
    color: "#666",
  },
  centered: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
  },
  errorText: {
    fontSize: 16,
    color: "#ff3b30",
  },
});
```

## Animation and Gestures

### React Native Reanimated

```typescript
// src/components/ui/AnimatedCard.tsx
import React from "react";
import { View, Text, StyleSheet } from "react-native";
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming,
  runOnJS,
} from "react-native-reanimated";
import { Gesture, GestureDetector } from "react-native-gesture-handler";

interface AnimatedCardProps {
  title: string;
  content: string;
  onSwipeLeft?: () => void;
  onSwipeRight?: () => void;
}

export function AnimatedCard({
  title,
  content,
  onSwipeLeft,
  onSwipeRight,
}: AnimatedCardProps) {
  const translateX = useSharedValue(0);
  const opacity = useSharedValue(1);
  const scale = useSharedValue(1);

  const panGesture = Gesture.Pan()
    .onUpdate((event) => {
      translateX.value = event.translationX;
      opacity.value = 1 - Math.abs(event.translationX) / 300;
    })
    .onEnd((event) => {
      const shouldDismiss = Math.abs(event.translationX) > 150;

      if (shouldDismiss) {
        translateX.value = withTiming(event.translationX > 0 ? 300 : -300);
        opacity.value = withTiming(0, undefined, () => {
          if (event.translationX > 0 && onSwipeRight) {
            runOnJS(onSwipeRight)();
          } else if (event.translationX < 0 && onSwipeLeft) {
            runOnJS(onSwipeLeft)();
          }
        });
      } else {
        translateX.value = withSpring(0);
        opacity.value = withSpring(1);
      }
    });

  const tapGesture = Gesture.Tap()
    .onBegin(() => {
      scale.value = withSpring(0.95);
    })
    .onFinalize(() => {
      scale.value = withSpring(1);
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }, { scale: scale.value }],
    opacity: opacity.value,
  }));

  return (
    <GestureDetector gesture={Gesture.Simultaneous(panGesture, tapGesture)}>
      <Animated.View style={[styles.card, animatedStyle]}>
        <Text style={styles.title}>{title}</Text>
        <Text style={styles.content}>{content}</Text>
      </Animated.View>
    </GestureDetector>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: "#fff",
    borderRadius: 12,
    padding: 20,
    margin: 16,
    shadowColor: "#000",
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 4,
  },
  title: {
    fontSize: 18,
    fontWeight: "600",
    marginBottom: 8,
  },
  content: {
    fontSize: 14,
    color: "#666",
    lineHeight: 20,
  },
});
```

## Testing Strategy

### Unit Testing

```typescript
// __tests__/components/Button.test.tsx
import React from "react";
import { render, fireEvent } from "@testing-library/react-native";
import { Button } from "@/components/ui/Button";

describe("Button Component", () => {
  it("renders correctly with title", () => {
    const { getByText } = render(
      <Button title="Test Button" onPress={() => {}} />
    );

    expect(getByText("Test Button")).toBeTruthy();
  });

  it("calls onPress when pressed", () => {
    const mockOnPress = jest.fn();
    const { getByRole } = render(
      <Button title="Test Button" onPress={mockOnPress} />
    );

    fireEvent.press(getByRole("button"));
    expect(mockOnPress).toHaveBeenCalledTimes(1);
  });

  it("shows loading indicator when isLoading is true", () => {
    const { getByTestId, queryByText } = render(
      <Button title="Test Button" onPress={() => {}} isLoading={true} />
    );

    expect(queryByText("Test Button")).toBeNull();
    // ActivityIndicator should be present
  });

  it("is disabled when disabled prop is true", () => {
    const mockOnPress = jest.fn();
    const { getByRole } = render(
      <Button title="Test Button" onPress={mockOnPress} disabled={true} />
    );

    const button = getByRole("button");
    expect(button.props.accessibilityState.disabled).toBe(true);

    fireEvent.press(button);
    expect(mockOnPress).not.toHaveBeenCalled();
  });
});
```

### Integration Testing

```typescript
// __tests__/screens/LoginScreen.test.tsx
import React from "react";
import { render, fireEvent, waitFor } from "@testing-library/react-native";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { LoginScreen } from "@/screens/auth/LoginScreen";
import { AuthProvider } from "@/store/AuthContext";

const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });

const TestWrapper = ({ children }: { children: React.ReactNode }) => {
  const queryClient = createTestQueryClient();

  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>{children}</AuthProvider>
    </QueryClientProvider>
  );
};

describe("LoginScreen", () => {
  it("renders login form correctly", () => {
    const { getByPlaceholderText, getByText } = render(
      <TestWrapper>
        <LoginScreen />
      </TestWrapper>
    );

    expect(getByPlaceholderText("Email")).toBeTruthy();
    expect(getByPlaceholderText("Password")).toBeTruthy();
    expect(getByText("Login")).toBeTruthy();
  });

  it("shows validation errors for invalid input", async () => {
    const { getByPlaceholderText, getByText, findByText } = render(
      <TestWrapper>
        <LoginScreen />
      </TestWrapper>
    );

    const emailInput = getByPlaceholderText("Email");
    const loginButton = getByText("Login");

    fireEvent.changeText(emailInput, "invalid-email");
    fireEvent.press(loginButton);

    await waitFor(() => {
      expect(findByText("Invalid email address")).toBeTruthy();
    });
  });
});
```

## Error Handling and Validation

### Form Validation with Zod

```typescript
// src/utils/validation.ts
import { z } from "zod";

export const loginSchema = z.object({
  email: z.string().email("Invalid email address"),
  password: z.string().min(6, "Password must be at least 6 characters"),
});

export const registerSchema = z
  .object({
    name: z.string().min(2, "Name must be at least 2 characters"),
    email: z.string().email("Invalid email address"),
    password: z.string().min(8, "Password must be at least 8 characters"),
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Passwords don't match",
    path: ["confirmPassword"],
  });

export type LoginFormData = z.infer<typeof loginSchema>;
export type RegisterFormData = z.infer<typeof registerSchema>;
```

### Error Boundary

```typescript
// src/components/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from "react";
import { View, Text, StyleSheet, TouchableOpacity } from "react-native";

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
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
    console.error("ErrorBoundary caught an error:", error, errorInfo);

    // Log to crash reporting service
    // crashlytics().recordError(error);
  }

  private handleRetry = () => {
    this.setState({ hasError: false, error: undefined });
  };

  public render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <View style={styles.container}>
          <Text style={styles.title}>Oops! Something went wrong</Text>
          <Text style={styles.message}>
            We're sorry, but something unexpected happened.
          </Text>
          <TouchableOpacity style={styles.button} onPress={this.handleRetry}>
            <Text style={styles.buttonText}>Try Again</Text>
          </TouchableOpacity>
        </View>
      );
    }

    return this.props.children;
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
    padding: 20,
    backgroundColor: "#f5f5f5",
  },
  title: {
    fontSize: 24,
    fontWeight: "600",
    color: "#333",
    marginBottom: 16,
    textAlign: "center",
  },
  message: {
    fontSize: 16,
    color: "#666",
    textAlign: "center",
    marginBottom: 32,
    lineHeight: 24,
  },
  button: {
    backgroundColor: "#007AFF",
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 8,
  },
  buttonText: {
    color: "#fff",
    fontSize: 16,
    fontWeight: "600",
  },
});
```

## Security and Storage

### Secure Storage Implementation

```typescript
// src/utils/secureStorage.ts
import * as SecureStore from "expo-secure-store";
import * as Crypto from "expo-crypto";

class SecureStorageService {
  private async encrypt(value: string): Promise<string> {
    // In production, use a proper encryption key
    return value; // Simplified for example
  }

  private async decrypt(value: string): Promise<string> {
    // In production, use proper decryption
    return value; // Simplified for example
  }

  async setItem(key: string, value: string): Promise<void> {
    try {
      const encryptedValue = await this.encrypt(value);
      await SecureStore.setItemAsync(key, encryptedValue);
    } catch (error) {
      console.error("Error storing secure item:", error);
      throw error;
    }
  }

  async getItem(key: string): Promise<string | null> {
    try {
      const encryptedValue = await SecureStore.getItemAsync(key);
      if (!encryptedValue) return null;

      return await this.decrypt(encryptedValue);
    } catch (error) {
      console.error("Error retrieving secure item:", error);
      return null;
    }
  }

  async removeItem(key: string): Promise<void> {
    try {
      await SecureStore.deleteItemAsync(key);
    } catch (error) {
      console.error("Error removing secure item:", error);
      throw error;
    }
  }

  async clear(): Promise<void> {
    // Note: SecureStore doesn't have a clear all method
    // You need to track keys and remove them individually
    const keysToRemove = ["authToken", "refreshToken", "userPreferences"];

    await Promise.all(keysToRemove.map((key) => this.removeItem(key)));
  }
}

export const secureStorage = new SecureStorageService();
```

## Internationalization

### i18n Setup

```typescript
// src/i18n/index.ts
import * as Localization from 'expo-localization';
import { I18n } from 'i18n-js';

import en from './locales/en.json';
import es from './locales/es.json';
import fr from './locales/fr.json';

const i18n = new I18n({
  en,
  es,
  fr,
});

i18n.locale = Localization.locale;
i18n.enableFallback = true;
i18n.defaultLocale = 'en';

export default i18n;

// src/i18n/locales/en.json
{
  "common": {
    "loading": "Loading...",
    "error": "Error",
    "retry": "Retry",
    "cancel": "Cancel",
    "save": "Save",
    "delete": "Delete"
  },
  "auth": {
    "login": "Login",
    "logout": "Logout",
    "email": "Email",
    "password": "Password",
    "forgotPassword": "Forgot Password?",
    "invalidCredentials": "Invalid email or password"
  },
  "profile": {
    "title": "Profile",
    "editProfile": "Edit Profile",
    "settings": "Settings"
  }
}
```

## Deployment and Distribution

### Build Configuration

```json
// eas.json
{
  "cli": {
    "version": ">= 3.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "simulator": true
      }
    },
    "production": {
      "autoIncrement": true,
      "env": {
        "NODE_ENV": "production"
      }
    }
  },
  "submit": {
    "production": {}
  }
}
```

### App Store Optimization

```json
// app.json - Production configuration
{
  "expo": {
    "name": "MyApp",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "updates": {
      "fallbackToCacheTimeout": 0,
      "url": "https://u.expo.dev/your-project-id"
    },
    "runtimeVersion": {
      "policy": "sdkVersion"
    },
    "assetBundlePatterns": ["**/*"],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yourcompany.myapp",
      "buildNumber": "1",
      "infoPlist": {
        "NSCameraUsageDescription": "This app uses the camera to take photos.",
        "NSPhotoLibraryUsageDescription": "This app accesses your photo library to select images."
      }
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      },
      "package": "com.yourcompany.myapp",
      "versionCode": 1,
      "permissions": [
        "android.permission.CAMERA",
        "android.permission.READ_EXTERNAL_STORAGE"
      ]
    },
    "web": {
      "favicon": "./assets/favicon.png"
    },
    "extra": {
      "eas": {
        "projectId": "your-project-id"
      }
    }
  }
}
```

## Common Issues and Solutions

### Issue 1: Metro Bundle Size Too Large

**Solution**:

- Use dynamic imports for large libraries
- Implement code splitting
- Remove unused dependencies
- Use Flipper only in development

### Issue 2: Performance Issues on Android

**Solution**:

- Enable Hermes engine
- Use FlatList for large datasets
- Optimize images with expo-image
- Avoid unnecessary re-renders

### Issue 3: iOS Build Failures

**Solution**:

- Ensure proper code signing
- Update Xcode and iOS SDK
- Clear derived data
- Check bundle identifier conflicts

### Issue 4: Navigation State Persistence

**Solution**:

```typescript
// Implement navigation state persistence
import AsyncStorage from "@react-native-async-storage/async-storage";

const PERSISTENCE_KEY = "NAVIGATION_STATE_V1";

export function AppNavigator() {
  const [isReady, setIsReady] = useState(false);
  const [initialState, setInitialState] = useState();

  useEffect(() => {
    const restoreState = async () => {
      try {
        const savedStateString = await AsyncStorage.getItem(PERSISTENCE_KEY);
        const state = savedStateString
          ? JSON.parse(savedStateString)
          : undefined;
        setInitialState(state);
      } finally {
        setIsReady(true);
      }
    };

    if (!isReady) {
      restoreState();
    }
  }, [isReady]);

  if (!isReady) {
    return null;
  }

  return (
    <NavigationContainer
      initialState={initialState}
      onStateChange={(state) =>
        AsyncStorage.setItem(PERSISTENCE_KEY, JSON.stringify(state))
      }
    >
      {/* Your navigation structure */}
    </NavigationContainer>
  );
}
```

## Reference Resources

- [Expo Official Documentation](https://docs.expo.dev/)
- [React Native Documentation](https://reactnative.dev/docs/getting-started)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [React Navigation Documentation](https://reactnavigation.org/docs/getting-started)
- [React Native Reanimated Documentation](https://docs.swmansion.com/react-native-reanimated/)
- [React Query Documentation](https://tanstack.com/query/latest)
- [Expo Security Guidelines](https://docs.expo.dev/guides/security/)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)

## Changelog

### v1.0.0 (2024-12-19)

- Initial release of TypeScript React Native Expo mobile development guide
- Comprehensive coverage of modern mobile development practices
- Included examples for components, navigation, state management, and testing
- Added performance optimization, security, and deployment considerations
- Covered accessibility, internationalization, and cross-platform compatibility

---

**Note**: This guide is based on Expo SDK 49+, React Native 0.72+, and TypeScript 5.0+. Please adjust configurations and examples according to your specific project requirements and the versions you are using. Always refer to the official Expo and React Native documentation for the most up-to-date information.
