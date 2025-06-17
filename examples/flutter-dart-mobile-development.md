---
name: "Flutter Dart Mobile Application Development Guide"
description: "A comprehensive development guide for building modern mobile applications using Flutter, Dart, Riverpod, Freezed, Flutter Hooks, and Supabase with best practices and performance optimization"
category: "Mobile Framework"
author: "Agents.md Collection"
authorUrl: "https://github.com/gakeez/agents_md_collection"
tags:
  [
    "flutter",
    "dart",
    "mobile-development",
    "riverpod",
    "freezed",
    "flutter-hooks",
    "supabase",
    "state-management",
  ]
lastUpdated: "2025-06-16"
---

# Flutter Dart Mobile Application Development Guide

## Project Overview

This comprehensive guide outlines best practices for developing modern mobile applications using Flutter, Dart, Riverpod for state management, Freezed for immutable data classes, Flutter Hooks for lifecycle management, and Supabase for backend services. The guide emphasizes functional and declarative programming patterns, performance optimization, and maintainable code architecture.

## Tech Stack

- **Framework**: Flutter 3.16+
- **Language**: Dart 3.2+
- **State Management**: Riverpod 2.4+
- **Data Classes**: Freezed + json_annotation
- **Lifecycle Management**: Flutter Hooks
- **Backend**: Supabase (Database, Auth, Storage)
- **Navigation**: GoRouter or auto_route
- **HTTP Client**: Dio with interceptors
- **Image Handling**: cached_network_image
- **Code Generation**: build_runner
- **Testing**: flutter_test, mockito

## Development Environment Setup

### Installation Requirements

- Flutter SDK 3.16+
- Dart SDK 3.2+
- Android Studio / VS Code with Flutter extensions
- Xcode (for iOS development)
- Supabase CLI (optional)

### Installation Steps

```bash
# Install Flutter dependencies
flutter pub add riverpod flutter_riverpod riverpod_annotation
flutter pub add freezed_annotation json_annotation
flutter pub add flutter_hooks hooks_riverpod
flutter pub add supabase_flutter
flutter pub add go_router
flutter pub add cached_network_image
flutter pub add dio

# Development dependencies
flutter pub add --dev build_runner
flutter pub add --dev riverpod_generator
flutter pub add --dev freezed
flutter pub add --dev json_serializable
flutter pub add --dev flutter_test
flutter pub add --dev mockito

# Generate code
flutter pub run build_runner build --delete-conflicting-outputs
```

## Project Structure

```
flutter_app/
├── lib/
│   ├── main.dart                    # App entry point
│   ├── app.dart                     # App configuration
│   ├── core/                        # Core utilities
│   │   ├── constants/
│   │   ├── extensions/
│   │   ├── utils/
│   │   └── theme/
│   ├── features/                    # Feature modules
│   │   ├── auth/
│   │   │   ├── data/               # Data layer
│   │   │   ├── domain/             # Domain layer
│   │   │   └── presentation/       # UI layer
│   │   ├── home/
│   │   └── profile/
│   ├── shared/                      # Shared components
│   │   ├── widgets/
│   │   ├── models/
│   │   └── providers/
│   └── services/                    # External services
│       ├── supabase_service.dart
│       ├── storage_service.dart
│       └── api_service.dart
├── test/                           # Test files
├── assets/                         # Static assets
│   ├── images/
│   ├── fonts/
│   └── icons/
├── pubspec.yaml
└── analysis_options.yaml
```

## Key Principles and Guidelines

### Core Development Philosophy

- Write concise, technical Dart code with accurate examples
- Use functional and declarative programming patterns where appropriate
- Prefer composition over inheritance
- Use descriptive variable names with auxiliary verbs (isLoading, hasError)
- Structure files: exported widget, subwidgets, helpers, static content, types

### Naming Conventions and Code Style

```dart
// Use descriptive variable names with auxiliary verbs
bool isLoading = false;
bool hasError = false;
bool canSubmit = true;

// Use const constructors for immutable widgets
class CustomButton extends StatelessWidget {
  const CustomButton({
    super.key,
    required this.onPressed,
    required this.text,
  });

  final VoidCallback onPressed;
  final String text;

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onPressed,
      child: Text(text),
    );
  }
}

// Use arrow syntax for simple functions
String get fullName => '$firstName $lastName';
bool get isValid => email.isNotEmpty && password.length >= 6;

// Use trailing commas for better formatting
Widget buildCard() {
  return Card(
    elevation: 4,
    margin: const EdgeInsets.all(16),
    child: Padding(
      padding: const EdgeInsets.all(20),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text('Title'),
          const SizedBox(height: 8),
          Text('Content'),
        ],
      ),
    ),
  );
}
```

### File Structure Convention

```dart
// user_profile_screen.dart - Proper file structure

// 1. Exported widget
class UserProfileScreen extends HookConsumerWidget {
  const UserProfileScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      appBar: AppBar(title: const Text('Profile')),
      body: const _ProfileContent(),
    );
  }
}

// 2. Subwidgets (private)
class _ProfileContent extends ConsumerWidget {
  const _ProfileContent();

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(currentUserProvider);

    return userAsync.when(
      data: (user) => _UserDetails(user: user),
      loading: () => const _LoadingWidget(),
      error: (error, stack) => _ErrorWidget(error: error),
    );
  }
}

class _UserDetails extends StatelessWidget {
  const _UserDetails({required this.user});

  final User user;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(user.name),
        Text(user.email),
      ],
    );
  }
}

class _LoadingWidget extends StatelessWidget {
  const _LoadingWidget();

  @override
  Widget build(BuildContext context) {
    return const Center(
      child: CircularProgressIndicator(),
    );
  }
}

class _ErrorWidget extends StatelessWidget {
  const _ErrorWidget({required this.error});

  final Object error;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: SelectableText.rich(
        TextSpan(
          text: 'Error: ${error.toString()}',
          style: TextStyle(color: Colors.red),
        ),
      ),
    );
  }
}

// 3. Helpers and utilities
extension UserProfileHelpers on User {
  String get displayName => name.isEmpty ? email : name;
  bool get hasProfileImage => profileImageUrl?.isNotEmpty ?? false;
}

// 4. Static content and constants
class _Constants {
  static const double profileImageSize = 120;
  static const EdgeInsets contentPadding = EdgeInsets.all(16);
}

// 5. Types and models (if specific to this file)
enum ProfileTab { info, settings, security }
```

## Core Feature Implementation

### Riverpod State Management

```dart
// providers/user_provider.dart - Modern Riverpod patterns

// Use @riverpod annotation for generating providers
@riverpod
class CurrentUser extends _$CurrentUser {
  @override
  Future<User?> build() async {
    // Initialize with current user from Supabase
    final supabase = ref.read(supabaseProvider);
    final session = supabase.auth.currentSession;

    if (session?.user == null) return null;

    return await _fetchUserProfile(session!.user.id);
  }

  Future<void> updateProfile(UserUpdateRequest request) async {
    state = const AsyncValue.loading();

    try {
      final updatedUser = await ref
          .read(userRepositoryProvider)
          .updateProfile(request);

      state = AsyncValue.data(updatedUser);
    } catch (error, stackTrace) {
      state = AsyncValue.error(error, stackTrace);
    }
  }

  Future<void> signOut() async {
    await ref.read(supabaseProvider).auth.signOut();
    ref.invalidate(currentUserProvider);
  }

  Future<User> _fetchUserProfile(String userId) async {
    return await ref.read(userRepositoryProvider).getUser(userId);
  }
}

// Prefer AsyncNotifierProvider over StateProvider
@riverpod
class UserList extends _$UserList {
  @override
  Future<List<User>> build() async {
    return await ref.read(userRepositoryProvider).getAllUsers();
  }

  Future<void> refresh() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() =>
        ref.read(userRepositoryProvider).getAllUsers());
  }

  Future<void> addUser(User user) async {
    final currentList = state.valueOrNull ?? [];
    state = AsyncValue.data([...currentList, user]);

    try {
      await ref.read(userRepositoryProvider).createUser(user);
    } catch (error) {
      // Revert optimistic update
      state = AsyncValue.data(currentList);
      rethrow;
    }
  }
}

// Simple state provider for UI state
@riverpod
class SelectedTab extends _$SelectedTab {
  @override
  int build() => 0;

  void selectTab(int index) => state = index;
}
```

### Freezed Data Models

```dart
// models/user.dart - Immutable data classes with Freezed

@freezed
class User with _$User {
  const factory User({
    required String id,
    required String email,
    required String name,
    String? profileImageUrl,
    @JsonKey(name: 'created_at') required DateTime createdAt,
    @JsonKey(name: 'updated_at') required DateTime updatedAt,
    @JsonKey(name: 'is_deleted', includeFromJson: true, includeToJson: false)
    @Default(false) bool isDeleted,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}

@freezed
class UserUpdateRequest with _$UserUpdateRequest {
  const factory UserUpdateRequest({
    String? name,
    String? profileImageUrl,
  }) = _UserUpdateRequest;

  factory UserUpdateRequest.fromJson(Map<String, dynamic> json) =>
      _$UserUpdateRequestFromJson(json);
}

// Union types for handling different states
@freezed
class AuthState with _$AuthState {
  const factory AuthState.initial() = AuthInitial;
  const factory AuthState.loading() = AuthLoading;
  const factory AuthState.authenticated(User user) = AuthAuthenticated;
  const factory AuthState.unauthenticated() = AuthUnauthenticated;
  const factory AuthState.error(String message) = AuthError;
}

// Enums with database values
enum UserRole {
  @JsonValue(0) user,
  @JsonValue(1) admin,
  @JsonValue(2) moderator,
}
```

### Error Handling and Validation

```dart
// Error handling in UI components
class UserListScreen extends ConsumerWidget {
  const UserListScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final usersAsync = ref.watch(userListProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Users')),
      body: usersAsync.when(
        data: (users) => users.isEmpty
            ? const _EmptyState()
            : _UserList(users: users),
        loading: () => const Center(
          child: CircularProgressIndicator(),
        ),
        error: (error, stackTrace) => _ErrorDisplay(
          error: error,
          onRetry: () => ref.invalidate(userListProvider),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddUserDialog(context, ref),
        child: const Icon(Icons.add),
      ),
    );
  }
}

class _ErrorDisplay extends StatelessWidget {
  const _ErrorDisplay({
    required this.error,
    required this.onRetry,
  });

  final Object error;
  final VoidCallback onRetry;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          SelectableText.rich(
            TextSpan(
              text: 'Error: ${_getErrorMessage(error)}',
              style: Theme.of(context).textTheme.bodyLarge?.copyWith(
                color: Colors.red,
              ),
            ),
          ),
          const SizedBox(height: 16),
          ElevatedButton(
            onPressed: onRetry,
            child: const Text('Retry'),
          ),
        ],
      ),
    );
  }

  String _getErrorMessage(Object error) {
    if (error is SupabaseException) {
      return error.message;
    }
    return error.toString();
  }
}

class _EmptyState extends StatelessWidget {
  const _EmptyState();

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(
            Icons.people_outline,
            size: 64,
            color: Theme.of(context).disabledColor,
          ),
          const SizedBox(height: 16),
          Text(
            'No users found',
            style: Theme.of(context).textTheme.titleLarge,
          ),
          const SizedBox(height: 8),
          Text(
            'Add your first user to get started',
            style: Theme.of(context).textTheme.bodyMedium,
          ),
        ],
      ),
    );
  }
}
```

## Flutter Hooks Integration

### Lifecycle Management with Hooks

```dart
// Using Flutter Hooks for lifecycle management
class SearchScreen extends HookConsumerWidget {
  const SearchScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Use hooks for local state and lifecycle
    final searchController = useTextEditingController();
    final focusNode = useFocusNode();
    final isSearching = useState(false);
    final searchResults = useState<List<User>>([]);

    // Debounced search
    final debouncedSearchTerm = useDebounced(
      searchController.text,
      const Duration(milliseconds: 500),
    );

    // Effect for search
    useEffect(() {
      if (debouncedSearchTerm.isEmpty) {
        searchResults.value = [];
        return null;
      }

      isSearching.value = true;

      // Perform search
      ref.read(userRepositoryProvider)
          .searchUsers(debouncedSearchTerm)
          .then((results) {
        searchResults.value = results;
        isSearching.value = false;
      }).catchError((error) {
        isSearching.value = false;
        // Handle error
      });

      return null;
    }, [debouncedSearchTerm]);

    // Auto-focus on mount
    useEffect(() {
      focusNode.requestFocus();
      return null;
    }, []);

    return Scaffold(
      appBar: AppBar(
        title: TextField(
          controller: searchController,
          focusNode: focusNode,
          decoration: const InputDecoration(
            hintText: 'Search users...',
            border: InputBorder.none,
          ),
          textCapitalization: TextCapitalization.words,
          keyboardType: TextInputType.text,
          textInputAction: TextInputAction.search,
        ),
      ),
      body: _SearchBody(
        isSearching: isSearching.value,
        results: searchResults.value,
        searchTerm: debouncedSearchTerm,
      ),
    );
  }
}

// Custom hook for debouncing
String useDebounced(String value, Duration duration) {
  final debouncedValue = useState(value);

  useEffect(() {
    final timer = Timer(duration, () {
      debouncedValue.value = value;
    });

    return timer.cancel;
  }, [value]);

  return debouncedValue.value;
}
```

### Supabase Integration

```dart
// services/supabase_service.dart - Supabase integration
@riverpod
SupabaseClient supabase(SupabaseRef ref) {
  return Supabase.instance.client;
}

@riverpod
class AuthService extends _$AuthService {
  @override
  AuthState build() {
    final supabase = ref.read(supabaseProvider);
    final session = supabase.auth.currentSession;

    if (session?.user != null) {
      return AuthState.authenticated(
        User.fromJson(session!.user.userMetadata ?? {}),
      );
    }

    return const AuthState.unauthenticated();
  }

  Future<void> signInWithEmail(String email, String password) async {
    state = const AuthState.loading();

    try {
      final response = await ref.read(supabaseProvider).auth.signInWithPassword(
        email: email,
        password: password,
      );

      if (response.user != null) {
        final user = await _fetchUserProfile(response.user!.id);
        state = AuthState.authenticated(user);
      }
    } on AuthException catch (error) {
      state = AuthState.error(error.message);
    } catch (error) {
      state = AuthState.error('An unexpected error occurred');
    }
  }

  Future<void> signUp(String email, String password, String name) async {
    state = const AuthState.loading();

    try {
      final response = await ref.read(supabaseProvider).auth.signUp(
        email: email,
        password: password,
        data: {'name': name},
      );

      if (response.user != null) {
        // Create user profile
        await _createUserProfile(response.user!, name);
        final user = await _fetchUserProfile(response.user!.id);
        state = AuthState.authenticated(user);
      }
    } on AuthException catch (error) {
      state = AuthState.error(error.message);
    } catch (error) {
      state = AuthState.error('Failed to create account');
    }
  }

  Future<void> signOut() async {
    await ref.read(supabaseProvider).auth.signOut();
    state = const AuthState.unauthenticated();
  }

  Future<User> _fetchUserProfile(String userId) async {
    final response = await ref.read(supabaseProvider)
        .from('users')
        .select()
        .eq('id', userId)
        .single();

    return User.fromJson(response);
  }

  Future<void> _createUserProfile(auth.User authUser, String name) async {
    await ref.read(supabaseProvider).from('users').insert({
      'id': authUser.id,
      'email': authUser.email,
      'name': name,
      'created_at': DateTime.now().toIso8601String(),
      'updated_at': DateTime.now().toIso8601String(),
    });
  }
}
```

## UI Components and Performance

### Optimized List Views and Image Handling

```dart
// Optimized list with proper image handling
class UserListView extends StatelessWidget {
  const UserListView({
    super.key,
    required this.users,
  });

  final List<User> users;

  @override
  Widget build(BuildContext context) {
    return RefreshIndicator(
      onRefresh: () async {
        // Implement refresh logic
      },
      child: ListView.builder(
        itemCount: users.length,
        itemBuilder: (context, index) {
          final user = users[index];
          return _UserListItem(user: user);
        },
      ),
    );
  }
}

class _UserListItem extends StatelessWidget {
  const _UserListItem({required this.user});

  final User user;

  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: _UserAvatar(user: user),
      title: Text(user.name),
      subtitle: Text(user.email),
      trailing: const Icon(Icons.chevron_right),
      onTap: () => context.push('/user/${user.id}'),
    );
  }
}

class _UserAvatar extends StatelessWidget {
  const _UserAvatar({required this.user});

  final User user;

  @override
  Widget build(BuildContext context) {
    if (user.profileImageUrl?.isNotEmpty == true) {
      return CircleAvatar(
        child: CachedNetworkImage(
          imageUrl: user.profileImageUrl!,
          imageBuilder: (context, imageProvider) => CircleAvatar(
            backgroundImage: imageProvider,
          ),
          placeholder: (context, url) => const CircularProgressIndicator(),
          errorWidget: (context, url, error) => _DefaultAvatar(user: user),
        ),
      );
    }

    return _DefaultAvatar(user: user);
  }
}

class _DefaultAvatar extends StatelessWidget {
  const _DefaultAvatar({required this.user});

  final User user;

  @override
  Widget build(BuildContext context) {
    return CircleAvatar(
      child: Text(
        user.name.isNotEmpty ? user.name[0].toUpperCase() : '?',
        style: Theme.of(context).textTheme.titleLarge,
      ),
    );
  }
}
```

### Responsive Design and Theming

```dart
// Responsive design utilities
class ResponsiveLayout extends StatelessWidget {
  const ResponsiveLayout({
    super.key,
    required this.mobile,
    this.tablet,
    this.desktop,
  });

  final Widget mobile;
  final Widget? tablet;
  final Widget? desktop;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        if (constraints.maxWidth >= 1200) {
          return desktop ?? tablet ?? mobile;
        } else if (constraints.maxWidth >= 800) {
          return tablet ?? mobile;
        } else {
          return mobile;
        }
      },
    );
  }
}

// Theme configuration
class AppTheme {
  static ThemeData get lightTheme {
    return ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: Colors.blue,
        brightness: Brightness.light,
      ),
      textTheme: const TextTheme(
        titleLarge: TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
        titleMedium: TextStyle(fontSize: 18, fontWeight: FontWeight.w600),
        bodyLarge: TextStyle(fontSize: 16),
        bodyMedium: TextStyle(fontSize: 14),
      ),
      elevatedButtonTheme: ElevatedButtonThemeData(
        style: ElevatedButton.styleFrom(
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(8),
          ),
        ),
      ),
    );
  }

  static ThemeData get darkTheme {
    return ThemeData(
      useMaterial3: true,
      colorScheme: ColorScheme.fromSeed(
        seedColor: Colors.blue,
        brightness: Brightness.dark,
      ),
      textTheme: const TextTheme(
        titleLarge: TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
        titleMedium: TextStyle(fontSize: 18, fontWeight: FontWeight.w600),
        bodyLarge: TextStyle(fontSize: 16),
        bodyMedium: TextStyle(fontSize: 14),
      ),
    );
  }
}
```

### Navigation with GoRouter

```dart
// router/app_router.dart - Navigation setup
@riverpod
GoRouter goRouter(GoRouterRef ref) {
  final authState = ref.watch(authServiceProvider);

  return GoRouter(
    initialLocation: '/',
    redirect: (context, state) {
      final isAuthenticated = authState.maybeWhen(
        authenticated: (_) => true,
        orElse: () => false,
      );

      final isAuthRoute = state.location.startsWith('/auth');

      if (!isAuthenticated && !isAuthRoute) {
        return '/auth/login';
      }

      if (isAuthenticated && isAuthRoute) {
        return '/';
      }

      return null;
    },
    routes: [
      GoRoute(
        path: '/',
        builder: (context, state) => const HomeScreen(),
      ),
      GoRoute(
        path: '/auth/login',
        builder: (context, state) => const LoginScreen(),
      ),
      GoRoute(
        path: '/auth/register',
        builder: (context, state) => const RegisterScreen(),
      ),
      GoRoute(
        path: '/profile',
        builder: (context, state) => const ProfileScreen(),
      ),
      GoRoute(
        path: '/user/:id',
        builder: (context, state) {
          final userId = state.pathParameters['id']!;
          return UserDetailScreen(userId: userId);
        },
      ),
    ],
  );
}
```

## Best Practices Summary

### Code Quality Guidelines

- **Use const constructors** for immutable widgets to optimize rebuilds
- **Leverage Freezed** for immutable state classes and unions
- **Prefer composition over inheritance** for better code reusability
- **Use descriptive variable names** with auxiliary verbs (isLoading, hasError)
- **Structure files properly** with exported widgets, subwidgets, helpers, and types
- **Implement proper error handling** using SelectableText.rich for error display
- **Use AsyncValue** for proper error handling and loading states

### Riverpod State Management

- **Use @riverpod annotation** for generating providers automatically
- **Prefer AsyncNotifierProvider and NotifierProvider** over StateProvider
- **Avoid StateProvider, StateNotifierProvider, and ChangeNotifierProvider**
- **Use ref.invalidate()** for manually triggering provider updates
- **Implement proper cancellation** of asynchronous operations when widgets are disposed

### Performance Optimization

- **Use const widgets** where possible to optimize rebuilds
- **Implement ListView.builder** for large lists instead of ListView with children
- **Use AssetImage for static images** and cached_network_image for remote images
- **Implement proper error handling** for Supabase operations, including network errors
- **Use RefreshIndicator** for pull-to-refresh functionality
- **Always include errorBuilder** when using Image.network

### UI and Styling Best Practices

- **Create small, private widget classes** instead of methods like Widget \_build...
- **Set appropriate textCapitalization, keyboardType, and textInputAction** in TextFields
- **Use Theme.of(context).textTheme.titleLarge** instead of deprecated headline6
- **Implement responsive design** using LayoutBuilder or MediaQuery
- **Use themes for consistent styling** across the app
- **Handle empty states** within the displaying screen

### Development Workflow

- **Use build_runner** for generating code from annotations (Freezed, Riverpod, JSON)
- **Run code generation** after modifying annotated classes
- **Use log instead of print** for debugging
- **Keep lines no longer than 80 characters** with trailing commas
- **Document complex logic** and non-obvious code decisions
- **Follow official documentation** for Flutter, Riverpod, and Supabase best practices

This comprehensive guide provides a solid foundation for building scalable, maintainable Flutter applications with modern state management, proper error handling, and performance optimization.
