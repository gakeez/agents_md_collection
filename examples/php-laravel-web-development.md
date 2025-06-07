---
name: "PHP Laravel Web Development Guide"
description: "A comprehensive development guide for building modern web applications using PHP and Laravel with best practices, SOLID principles, and advanced features"
category: "Backend Service"
author: "Agents.md Collection"
authorUrl: "https://github.com/gakeez/agents_md_collection"
tags: ["php", "laravel", "web-development", "backend", "api", "mvc", "eloquent"]
lastUpdated: "2024-12-19"
---

# PHP Laravel Web Development Guide

## Project Overview

This comprehensive guide outlines best practices for developing modern web applications using PHP and Laravel. It emphasizes concise, technical implementation with accurate PHP examples, following Laravel conventions and SOLID principles. The guide focuses on object-oriented programming, dependency injection, and Laravel's built-in features for maximum efficiency and maintainability.

## Tech Stack

- **Backend Framework**: Laravel 10+ with PHP 8.1+
- **Database**: MySQL/PostgreSQL with Eloquent ORM
- **Frontend**: Blade templating engine + Laravel Mix
- **Authentication**: Laravel Sanctum/Passport
- **Caching**: Redis/Memcached
- **Queue System**: Redis/Database queues
- **Testing**: PHPUnit + Laravel Dusk
- **Package Manager**: Composer

## Project Structure

```
laravel-project/
├── app/
│   ├── Console/
│   │   └── Commands/
│   ├── Exceptions/
│   │   └── Handler.php
│   ├── Http/
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Providers/
│   ├── Repositories/
│   └── Services/
├── bootstrap/
├── config/
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
├── public/
├── resources/
│   ├── css/
│   ├── js/
│   ├── lang/
│   └── views/
├── routes/
│   ├── api.php
│   ├── channels.php
│   ├── console.php
│   └── web.php
├── storage/
├── tests/
│   ├── Feature/
│   └── Unit/
├── vendor/
├── .env
├── artisan
├── composer.json
└── webpack.mix.js
```

## Development Guidelines

### Key Principles

- Write concise, technical responses with accurate PHP examples
- Follow Laravel best practices and conventions
- Use object-oriented programming with a focus on SOLID principles
- Prefer iteration and modularization over duplication
- Use descriptive variable and method names
- Use lowercase with dashes for directories
- Favor dependency injection and service containers

### PHP/Laravel Best Practices

#### Modern PHP Features

- Use PHP 8.1+ features when appropriate (typed properties, match expressions)
- Follow PSR-12 coding standards
- Use strict typing: `declare(strict_types=1);`
- Utilize Laravel's built-in features and helpers when possible

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Article extends Model
{
    use HasFactory;

    protected $fillable = [
        'title',
        'slug',
        'content',
        'excerpt',
        'is_published',
        'published_at',
        'user_id',
        'category_id',
    ];

    protected $casts = [
        'is_published' => 'boolean',
        'published_at' => 'datetime',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }

    public function scopePublished($query)
    {
        return $query->where('is_published', true);
    }

    public function scopeRecent($query, int $days = 7)
    {
        return $query->where('created_at', '>=', now()->subDays($days));
    }
}
```

#### Controllers and Request Handling

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers;

use App\Http\Requests\StoreArticleRequest;
use App\Http\Requests\UpdateArticleRequest;
use App\Http\Resources\ArticleResource;
use App\Models\Article;
use App\Services\ArticleService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\AnonymousResourceCollection;

class ArticleController extends Controller
{
    public function __construct(
        private readonly ArticleService $articleService
    ) {
        $this->middleware('auth')->except(['index', 'show']);
    }

    public function index(Request $request): AnonymousResourceCollection
    {
        $articles = Article::query()
            ->with(['user', 'category'])
            ->published()
            ->when($request->filled('category'), function ($query) use ($request) {
                $query->whereHas('category', function ($q) use ($request) {
                    $q->where('slug', $request->category);
                });
            })
            ->when($request->filled('search'), function ($query) use ($request) {
                $query->where('title', 'like', "%{$request->search}%")
                      ->orWhere('content', 'like', "%{$request->search}%");
            })
            ->latest('published_at')
            ->paginate(15);

        return ArticleResource::collection($articles);
    }

    public function store(StoreArticleRequest $request): JsonResponse
    {
        $article = $this->articleService->create($request->validated());

        return response()->json([
            'message' => 'Article created successfully',
            'data' => new ArticleResource($article),
        ], 201);
    }

    public function show(Article $article): ArticleResource
    {
        $article->load(['user', 'category', 'comments.user']);

        return new ArticleResource($article);
    }

    public function update(UpdateArticleRequest $request, Article $article): JsonResponse
    {
        $this->authorize('update', $article);

        $article = $this->articleService->update($article, $request->validated());

        return response()->json([
            'message' => 'Article updated successfully',
            'data' => new ArticleResource($article),
        ]);
    }

    public function destroy(Article $article): JsonResponse
    {
        $this->authorize('delete', $article);

        $this->articleService->delete($article);

        return response()->json([
            'message' => 'Article deleted successfully',
        ]);
    }
}
```

#### Form Request Validation

```php
<?php

declare(strict_types=1);

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class StoreArticleRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', Article::class);
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'slug' => [
                'required',
                'string',
                'max:255',
                'regex:/^[a-z0-9-]+$/',
                Rule::unique('articles'),
            ],
            'content' => ['required', 'string', 'min:100'],
            'excerpt' => ['nullable', 'string', 'max:500'],
            'category_id' => ['required', 'exists:categories,id'],
            'is_published' => ['boolean'],
            'published_at' => ['nullable', 'date', 'after_or_equal:now'],
            'tags' => ['array'],
            'tags.*' => ['string', 'max:50'],
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => 'The article title is required.',
            'content.min' => 'The article content must be at least 100 characters.',
            'slug.regex' => 'The slug may only contain lowercase letters, numbers, and hyphens.',
        ];
    }

    protected function prepareForValidation(): void
    {
        if ($this->filled('title') && !$this->filled('slug')) {
            $this->merge([
                'slug' => str($this->title)->slug(),
            ]);
        }
    }
}
```

## Environment Setup

### Development Requirements

- PHP >= 8.1
- Composer >= 2.0
- Laravel >= 10.0
- MySQL >= 8.0 or PostgreSQL >= 13
- Redis >= 6.0 (optional, for caching and queues)
- Node.js >= 16 (for asset compilation)

### Installation Steps

```bash
# 1. Create new Laravel project
composer create-project laravel/laravel my-laravel-app

# 2. Navigate to project directory
cd my-laravel-app

# 3. Set up environment variables
cp .env.example .env
php artisan key:generate

# 4. Configure database in .env file
# DB_CONNECTION=mysql
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=laravel_app
# DB_USERNAME=root
# DB_PASSWORD=

# 5. Run database migrations
php artisan migrate

# 6. Install frontend dependencies
npm install

# 7. Compile assets
npm run dev

# 8. Start development server
php artisan serve
```

### Environment Configuration

```env
# .env
APP_NAME="Laravel App"
APP_ENV=local
APP_KEY=base64:generated-key-here
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_app
DB_USERNAME=root
DB_PASSWORD=

BROADCAST_DRIVER=log
CACHE_DRIVER=redis
FILESYSTEM_DISK=local
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

## Core Feature Implementation

### Service Layer Pattern

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\Article;
use App\Repositories\ArticleRepository;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Pagination\LengthAwarePaginator;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Cache;

class ArticleService
{
    public function __construct(
        private readonly ArticleRepository $articleRepository
    ) {}

    public function create(array $data): Article
    {
        return DB::transaction(function () use ($data) {
            $article = $this->articleRepository->create([
                ...$data,
                'user_id' => auth()->id(),
                'published_at' => $data['is_published'] ? now() : null,
            ]);

            if (isset($data['tags'])) {
                $article->tags()->sync($this->getOrCreateTags($data['tags']));
            }

            Cache::tags(['articles'])->flush();

            return $article->load(['user', 'category', 'tags']);
        });
    }

    public function update(Article $article, array $data): Article
    {
        return DB::transaction(function () use ($article, $data) {
            $article->update([
                ...$data,
                'published_at' => $data['is_published'] && !$article->published_at
                    ? now()
                    : $article->published_at,
            ]);

            if (isset($data['tags'])) {
                $article->tags()->sync($this->getOrCreateTags($data['tags']));
            }

            Cache::tags(['articles'])->flush();

            return $article->fresh(['user', 'category', 'tags']);
        });
    }

    public function delete(Article $article): bool
    {
        return DB::transaction(function () use ($article) {
            $article->tags()->detach();
            $deleted = $article->delete();

            Cache::tags(['articles'])->flush();

            return $deleted;
        });
    }

    public function getPublished(array $filters = []): LengthAwarePaginator
    {
        $cacheKey = 'articles.published.' . md5(serialize($filters));

        return Cache::tags(['articles'])->remember($cacheKey, 3600, function () use ($filters) {
            return $this->articleRepository->getPublished($filters);
        });
    }

    private function getOrCreateTags(array $tagNames): array
    {
        $tags = [];
        foreach ($tagNames as $tagName) {
            $tag = Tag::firstOrCreate(['name' => $tagName]);
            $tags[] = $tag->id;
        }
        return $tags;
    }
}
```

### Repository Pattern

```php
<?php

declare(strict_types=1);

namespace App\Repositories;

use App\Models\Article;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Pagination\LengthAwarePaginator;

class ArticleRepository
{
    public function create(array $data): Article
    {
        return Article::create($data);
    }

    public function findById(int $id): ?Article
    {
        return Article::with(['user', 'category', 'tags'])->find($id);
    }

    public function findBySlug(string $slug): ?Article
    {
        return Article::with(['user', 'category', 'tags'])
            ->where('slug', $slug)
            ->first();
    }

    public function getPublished(array $filters = []): LengthAwarePaginator
    {
        return Article::query()
            ->with(['user', 'category'])
            ->published()
            ->when(isset($filters['category']), function (Builder $query) use ($filters) {
                $query->whereHas('category', function ($q) use ($filters) {
                    $q->where('slug', $filters['category']);
                });
            })
            ->when(isset($filters['search']), function (Builder $query) use ($filters) {
                $query->where(function ($q) use ($filters) {
                    $q->where('title', 'like', "%{$filters['search']}%")
                      ->orWhere('content', 'like', "%{$filters['search']}%");
                });
            })
            ->when(isset($filters['tag']), function (Builder $query) use ($filters) {
                $query->whereHas('tags', function ($q) use ($filters) {
                    $q->where('name', $filters['tag']);
                });
            })
            ->latest('published_at')
            ->paginate($filters['per_page'] ?? 15);
    }

    public function getByUser(int $userId): Collection
    {
        return Article::with(['category', 'tags'])
            ->where('user_id', $userId)
            ->latest()
            ->get();
    }
}
```

### API Resources

```php
<?php

declare(strict_types=1);

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ArticleResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'slug' => $this->slug,
            'excerpt' => $this->excerpt,
            'content' => $this->when($request->routeIs('articles.show'), $this->content),
            'is_published' => $this->is_published,
            'published_at' => $this->published_at?->toISOString(),
            'created_at' => $this->created_at->toISOString(),
            'updated_at' => $this->updated_at->toISOString(),
            'author' => new UserResource($this->whenLoaded('user')),
            'category' => new CategoryResource($this->whenLoaded('category')),
            'tags' => TagResource::collection($this->whenLoaded('tags')),
            'comments_count' => $this->when(
                $this->relationLoaded('comments'),
                fn() => $this->comments->count()
            ),
            'reading_time' => $this->getReadingTime(),
        ];
    }

    private function getReadingTime(): int
    {
        $wordCount = str_word_count(strip_tags($this->content));
        return max(1, (int) ceil($wordCount / 200)); // Assume 200 WPM
    }
}
```

### Middleware Implementation

```php
<?php

declare(strict_types=1);

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\RateLimiter;
use Symfony\Component\HttpFoundation\Response;

class ApiRateLimit
{
    public function handle(Request $request, Closure $next, string $maxAttempts = '60', string $decayMinutes = '1'): Response
    {
        $key = $this->resolveRequestSignature($request);

        if (RateLimiter::tooManyAttempts($key, (int) $maxAttempts)) {
            return response()->json([
                'message' => 'Too many requests. Please try again later.',
                'retry_after' => RateLimiter::availableIn($key),
            ], 429);
        }

        RateLimiter::hit($key, (int) $decayMinutes * 60);

        $response = $next($request);

        $response->headers->set('X-RateLimit-Limit', $maxAttempts);
        $response->headers->set('X-RateLimit-Remaining', RateLimiter::remaining($key, (int) $maxAttempts));

        return $response;
    }

    private function resolveRequestSignature(Request $request): string
    {
        $userId = $request->user()?->id;
        $route = $request->route()?->getName();

        return sha1($userId . '|' . $request->ip() . '|' . $route);
    }
}
```

### Database Migrations

```php
<?php

declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('articles', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->string('slug')->unique();
            $table->text('excerpt')->nullable();
            $table->longText('content');
            $table->boolean('is_published')->default(false);
            $table->timestamp('published_at')->nullable();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->foreignId('category_id')->constrained()->onDelete('cascade');
            $table->timestamps();

            $table->index(['is_published', 'published_at']);
            $table->index(['user_id', 'created_at']);
            $table->index(['category_id', 'published_at']);
            $table->fullText(['title', 'content']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('articles');
    }
};
```

### Model Factories

```php
<?php

declare(strict_types=1);

namespace Database\Factories;

use App\Models\Category;
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

class ArticleFactory extends Factory
{
    public function definition(): array
    {
        $title = fake()->sentence(6, true);

        return [
            'title' => $title,
            'slug' => str($title)->slug(),
            'excerpt' => fake()->paragraph(2),
            'content' => fake()->paragraphs(8, true),
            'is_published' => fake()->boolean(70),
            'published_at' => fake()->optional(0.7)->dateTimeBetween('-1 year', 'now'),
            'user_id' => User::factory(),
            'category_id' => Category::factory(),
        ];
    }

    public function published(): static
    {
        return $this->state(fn (array $attributes) => [
            'is_published' => true,
            'published_at' => fake()->dateTimeBetween('-6 months', 'now'),
        ]);
    }

    public function draft(): static
    {
        return $this->state(fn (array $attributes) => [
            'is_published' => false,
            'published_at' => null,
        ]);
    }
}
```

## Authentication and Authorization

### Authentication Setup

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Http\Requests\Auth\LoginRequest;
use App\Http\Requests\Auth\RegisterRequest;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    public function register(RegisterRequest $request): JsonResponse
    {
        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
        ]);

        $token = $user->createToken('auth-token')->plainTextToken;

        return response()->json([
            'message' => 'Registration successful',
            'user' => $user,
            'token' => $token,
        ], 201);
    }

    public function login(LoginRequest $request): JsonResponse
    {
        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        $token = $user->createToken('auth-token')->plainTextToken;

        return response()->json([
            'message' => 'Login successful',
            'user' => $user,
            'token' => $token,
        ]);
    }

    public function logout(): JsonResponse
    {
        auth()->user()->currentAccessToken()->delete();

        return response()->json([
            'message' => 'Logout successful',
        ]);
    }

    public function me(): JsonResponse
    {
        return response()->json([
            'user' => auth()->user(),
        ]);
    }
}
```

### Authorization Policies

```php
<?php

declare(strict_types=1);

namespace App\Policies;

use App\Models\Article;
use App\Models\User;

class ArticlePolicy
{
    public function viewAny(?User $user): bool
    {
        return true;
    }

    public function view(?User $user, Article $article): bool
    {
        if ($article->is_published) {
            return true;
        }

        return $user && ($user->id === $article->user_id || $user->isAdmin());
    }

    public function create(User $user): bool
    {
        return $user->hasVerifiedEmail();
    }

    public function update(User $user, Article $article): bool
    {
        return $user->id === $article->user_id || $user->isAdmin();
    }

    public function delete(User $user, Article $article): bool
    {
        return $user->id === $article->user_id || $user->isAdmin();
    }

    public function publish(User $user, Article $article): bool
    {
        return ($user->id === $article->user_id && $user->canPublish()) || $user->isAdmin();
    }
}
```

## Queue Jobs and Background Processing

### Job Implementation

```php
<?php

declare(strict_types=1);

namespace App\Jobs;

use App\Models\Article;
use App\Notifications\ArticlePublished;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Notification;

class ProcessArticlePublication implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;
    public int $maxExceptions = 2;

    public function __construct(
        private readonly Article $article
    ) {}

    public function handle(): void
    {
        // Generate SEO metadata
        $this->generateSeoMetadata();

        // Process images
        $this->processImages();

        // Send notifications
        $this->sendNotifications();

        // Update search index
        $this->updateSearchIndex();
    }

    public function failed(\Throwable $exception): void
    {
        \Log::error('Article publication failed', [
            'article_id' => $this->article->id,
            'error' => $exception->getMessage(),
        ]);
    }

    private function generateSeoMetadata(): void
    {
        if (!$this->article->excerpt) {
            $this->article->update([
                'excerpt' => str($this->article->content)->limit(160),
            ]);
        }
    }

    private function processImages(): void
    {
        // Process and optimize images in article content
        // This could involve resizing, compression, etc.
    }

    private function sendNotifications(): void
    {
        $subscribers = $this->article->user->followers()
            ->where('notify_on_publish', true)
            ->get();

        Notification::send($subscribers, new ArticlePublished($this->article));
    }

    private function updateSearchIndex(): void
    {
        // Update search index (Elasticsearch, Algolia, etc.)
    }
}
```

### Event and Listener System

```php
<?php

declare(strict_types=1);

namespace App\Events;

use App\Models\Article;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class ArticlePublished
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public function __construct(
        public readonly Article $article
    ) {}
}

// Listener
namespace App\Listeners;

use App\Events\ArticlePublished;
use App\Jobs\ProcessArticlePublication;

class HandleArticlePublication
{
    public function handle(ArticlePublished $event): void
    {
        ProcessArticlePublication::dispatch($event->article);
    }
}
```

## Testing Strategy

### Feature Tests

```php
<?php

declare(strict_types=1);

namespace Tests\Feature;

use App\Models\Article;
use App\Models\Category;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ArticleControllerTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_list_published_articles(): void
    {
        $publishedArticles = Article::factory()->published()->count(3)->create();
        $draftArticles = Article::factory()->draft()->count(2)->create();

        $response = $this->getJson('/api/articles');

        $response->assertOk()
            ->assertJsonCount(3, 'data')
            ->assertJsonStructure([
                'data' => [
                    '*' => [
                        'id',
                        'title',
                        'slug',
                        'excerpt',
                        'is_published',
                        'published_at',
                        'author',
                        'category',
                    ],
                ],
                'links',
                'meta',
            ]);
    }

    public function test_authenticated_user_can_create_article(): void
    {
        $user = User::factory()->create();
        $category = Category::factory()->create();

        $articleData = [
            'title' => 'Test Article',
            'slug' => 'test-article',
            'content' => 'This is a test article content that is long enough to pass validation.',
            'category_id' => $category->id,
            'is_published' => true,
        ];

        $response = $this->actingAs($user)
            ->postJson('/api/articles', $articleData);

        $response->assertCreated()
            ->assertJsonFragment([
                'title' => 'Test Article',
                'slug' => 'test-article',
            ]);

        $this->assertDatabaseHas('articles', [
            'title' => 'Test Article',
            'user_id' => $user->id,
        ]);
    }

    public function test_user_can_only_update_own_articles(): void
    {
        $user = User::factory()->create();
        $otherUser = User::factory()->create();
        $article = Article::factory()->create(['user_id' => $otherUser->id]);

        $response = $this->actingAs($user)
            ->putJson("/api/articles/{$article->id}", [
                'title' => 'Updated Title',
            ]);

        $response->assertForbidden();
    }
}
```

### Unit Tests

```php
<?php

declare(strict_types=1);

namespace Tests\Unit;

use App\Models\Article;
use App\Models\User;
use App\Services\ArticleService;
use App\Repositories\ArticleRepository;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;
use Mockery;

class ArticleServiceTest extends TestCase
{
    use RefreshDatabase;

    private ArticleService $articleService;
    private $articleRepository;

    protected function setUp(): void
    {
        parent::setUp();

        $this->articleRepository = Mockery::mock(ArticleRepository::class);
        $this->articleService = new ArticleService($this->articleRepository);
    }

    public function test_create_article_sets_user_id(): void
    {
        $user = User::factory()->create();
        $this->actingAs($user);

        $articleData = [
            'title' => 'Test Article',
            'content' => 'Test content',
            'is_published' => true,
        ];

        $expectedData = [
            ...$articleData,
            'user_id' => $user->id,
            'published_at' => now(),
        ];

        $article = Article::factory()->make($expectedData);

        $this->articleRepository
            ->shouldReceive('create')
            ->once()
            ->with(Mockery::subset($expectedData))
            ->andReturn($article);

        $result = $this->articleService->create($articleData);

        $this->assertEquals($article->title, $result->title);
        $this->assertEquals($user->id, $result->user_id);
    }
}
```

## Performance Optimization

### Database Query Optimization

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers;

use App\Models\Article;
use Illuminate\Http\Request;
use Illuminate\Database\Eloquent\Builder;

class OptimizedArticleController extends Controller
{
    public function index(Request $request)
    {
        // Eager loading to prevent N+1 queries
        $articles = Article::query()
            ->with([
                'user:id,name,email',
                'category:id,name,slug',
                'tags:id,name'
            ])
            ->withCount(['comments', 'likes'])
            ->published()
            ->when($request->filled('search'), function (Builder $query) use ($request) {
                $query->whereFullText(['title', 'content'], $request->search);
            })
            ->latest('published_at')
            ->paginate(15);

        return response()->json($articles);
    }

    public function popular()
    {
        // Using database views for complex queries
        return Article::query()
            ->fromSub(function ($query) {
                $query->select([
                    'articles.*',
                    \DB::raw('(
                        (COALESCE(likes_count, 0) * 2) +
                        (COALESCE(comments_count, 0) * 3) +
                        (COALESCE(views_count, 0) * 0.1)
                    ) as popularity_score')
                ])
                ->from('articles')
                ->leftJoin('article_stats', 'articles.id', '=', 'article_stats.article_id')
                ->where('is_published', true);
            }, 'ranked_articles')
            ->orderByDesc('popularity_score')
            ->limit(10)
            ->get();
    }
}
```

### Caching Strategies

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\Article;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Redis;

class CacheService
{
    public function getPopularArticles(): array
    {
        return Cache::tags(['articles', 'popular'])
            ->remember('articles.popular', 3600, function () {
                return Article::query()
                    ->with(['user:id,name', 'category:id,name'])
                    ->published()
                    ->orderByDesc('views_count')
                    ->limit(10)
                    ->get()
                    ->toArray();
            });
    }

    public function incrementViewCount(Article $article): void
    {
        $key = "article:{$article->id}:views";

        Redis::incr($key);
        Redis::expire($key, 3600); // Expire after 1 hour

        // Batch update to database every 100 views
        if (Redis::get($key) % 100 === 0) {
            $article->increment('views_count', 100);
            Redis::del($key);
        }
    }

    public function clearArticleCache(Article $article): void
    {
        Cache::tags(['articles'])->flush();
        Cache::forget("article:{$article->id}");
        Cache::forget("article:slug:{$article->slug}");
    }
}
```

## Security Best Practices

### Input Validation and Sanitization

```php
<?php

declare(strict_types=1);

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class SecureArticleRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'title' => [
                'required',
                'string',
                'max:255',
                'regex:/^[a-zA-Z0-9\s\-_.,!?]+$/', // Only allow safe characters
            ],
            'content' => [
                'required',
                'string',
                'min:100',
                function ($attribute, $value, $fail) {
                    // Custom validation to prevent malicious content
                    if ($this->containsMaliciousContent($value)) {
                        $fail('The content contains prohibited elements.');
                    }
                },
            ],
        ];
    }

    protected function prepareForValidation(): void
    {
        $this->merge([
            'title' => strip_tags($this->title),
            'content' => $this->sanitizeContent($this->content),
        ]);
    }

    private function containsMaliciousContent(string $content): bool
    {
        $maliciousPatterns = [
            '/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/mi',
            '/javascript:/i',
            '/on\w+\s*=/i',
        ];

        foreach ($maliciousPatterns as $pattern) {
            if (preg_match($pattern, $content)) {
                return true;
            }
        }

        return false;
    }

    private function sanitizeContent(string $content): string
    {
        // Allow only safe HTML tags
        $allowedTags = '<p><br><strong><em><ul><ol><li><h1><h2><h3><h4><h5><h6><blockquote><a>';
        return strip_tags($content, $allowedTags);
    }
}
```

### CSRF and Security Headers

```php
<?php

declare(strict_types=1);

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class SecurityHeaders
{
    public function handle(Request $request, Closure $next)
    {
        $response = $next($request);

        $response->headers->set('X-Content-Type-Options', 'nosniff');
        $response->headers->set('X-Frame-Options', 'DENY');
        $response->headers->set('X-XSS-Protection', '1; mode=block');
        $response->headers->set('Referrer-Policy', 'strict-origin-when-cross-origin');
        $response->headers->set('Permissions-Policy', 'geolocation=(), microphone=(), camera=()');

        if ($request->secure()) {
            $response->headers->set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
        }

        return $response;
    }
}
```

## Deployment and Production

### Production Configuration

```php
<?php

// config/database.php - Production optimizations
return [
    'default' => env('DB_CONNECTION', 'mysql'),

    'connections' => [
        'mysql' => [
            'driver' => 'mysql',
            'url' => env('DATABASE_URL'),
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'forge'),
            'username' => env('DB_USERNAME', 'forge'),
            'password' => env('DB_PASSWORD', ''),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'prefix_indexes' => true,
            'strict' => true,
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
                PDO::ATTR_PERSISTENT => true,
                PDO::ATTR_TIMEOUT => 30,
            ]) : [],
        ],
    ],
];
```

### Deployment Script

```bash
#!/bin/bash

# Laravel Production Deployment Script

echo "Starting deployment..."

# Pull latest code
git pull origin main

# Install/update dependencies
composer install --no-dev --optimize-autoloader

# Clear and cache configurations
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache

# Run database migrations
php artisan migrate --force

# Clear application cache
php artisan cache:clear

# Restart queue workers
php artisan queue:restart

# Compile assets
npm ci --production
npm run production

# Set proper permissions
chmod -R 755 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache

# Restart services
sudo systemctl reload nginx
sudo systemctl restart php8.1-fpm

echo "Deployment completed successfully!"
```

## Common Issues and Solutions

### Issue 1: N+1 Query Problems

**Solution**: Use eager loading with `with()` method and optimize relationships.

### Issue 2: Memory Issues with Large Datasets

**Solution**: Use `chunk()` or `cursor()` methods for processing large datasets.

```php
// Instead of
$articles = Article::all();

// Use
Article::chunk(100, function ($articles) {
    foreach ($articles as $article) {
        // Process article
    }
});
```

### Issue 3: Queue Job Failures

**Solution**: Implement proper error handling and retry mechanisms.

### Issue 4: Cache Invalidation

**Solution**: Use cache tags and implement proper cache invalidation strategies.

## Reference Resources

- [Laravel Official Documentation](https://laravel.com/docs)
- [PHP The Right Way](https://phptherightway.com/)
- [PSR Standards](https://www.php-fig.org/psr/)
- [Laravel Best Practices](https://github.com/alexeymezenin/laravel-best-practices)
- [Eloquent ORM Documentation](https://laravel.com/docs/eloquent)
- [Laravel Testing Documentation](https://laravel.com/docs/testing)
- [Laravel Security Best Practices](https://laravel.com/docs/security)

## Changelog

### v1.0.0 (2024-12-19)

- Initial release of PHP Laravel web development guide
- Comprehensive coverage of Laravel best practices and patterns
- Included examples for models, controllers, services, and repositories
- Added authentication, authorization, and security considerations
- Covered testing strategies, performance optimization, and deployment

---

**Note**: This guide is based on Laravel 10+ and PHP 8.1+. Please adjust configurations and examples according to your specific project requirements and the versions you are using. Always refer to the official Laravel documentation for the most up-to-date information.
