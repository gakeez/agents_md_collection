---
name: "Ghost CMS Theme Development Guide"
description: "A comprehensive development guide for building scalable Ghost CMS themes using Handlebars templating, Alpine.js, Tailwind CSS, and JavaScript with best practices for content management and performance optimization"
category: "Content Management System"
author: "Agents.md Collection"
authorUrl: "https://github.com/gakeez/agents_md_collection"
tags:
  [
    "ghost-cms",
    "handlebars",
    "alpine-js",
    "tailwind-css",
    "javascript",
    "content-management",
    "theme-development",
  ]
lastUpdated: "2025-01-02"
---

# Ghost CMS Theme Development Guide

## Project Overview

This comprehensive guide outlines best practices for developing scalable Ghost CMS themes using Handlebars templating, Alpine.js, Tailwind CSS, and JavaScript. The guide emphasizes leveraging Ghost's content API, dynamic routing, performance optimization, and proper asset management while following Ghost's naming conventions and theme structure.

## Tech Stack

- **CMS Platform**: Ghost CMS 5.0+
- **Templating Engine**: Handlebars.js
- **Frontend Framework**: Alpine.js for dynamic interactions
- **CSS Framework**: Tailwind CSS with utility-first approach
- **JavaScript**: ES6+ with modern browser support
- **Build Tools**: Ghost CLI, PostCSS, and custom build scripts
- **API**: Ghost Content API for dynamic content
- **Performance**: Built-in Ghost optimization features

## Project Structure

```
ghost-theme/
├── assets/
│   ├── css/
│   │   ├── screen.css          # Main stylesheet
│   │   └── components/         # Component styles
│   ├── js/
│   │   ├── main.js            # Main JavaScript
│   │   ├── components/        # JavaScript components
│   │   └── utils/             # Utility functions
│   ├── images/
│   │   ├── icons/
│   │   └── graphics/
│   └── fonts/
├── partials/
│   ├── header.hbs             # Site header
│   ├── footer.hbs             # Site footer
│   ├── navigation.hbs         # Navigation menu
│   ├── post-card.hbs          # Post preview card
│   ├── author-card.hbs        # Author information
│   └── pagination.hbs         # Pagination component
├── templates/
│   ├── index.hbs              # Homepage template
│   ├── post.hbs               # Single post template
│   ├── page.hbs               # Static page template
│   ├── tag.hbs                # Tag archive template
│   ├── author.hbs             # Author archive template
│   ├── error.hbs              # 404 error template
│   └── custom-*.hbs           # Custom templates
├── default.hbs                # Base layout template
├── package.json               # Theme configuration
├── routes.yaml                # Custom routing
├── tailwind.config.js         # Tailwind configuration
├── postcss.config.js          # PostCSS configuration
└── README.md
```

## Development Guidelines

### Key Principles

- Write concise, technical responses with accurate Ghost theme examples
- Leverage Ghost's content API and dynamic routing effectively
- Prioritize performance optimization and proper asset management
- Use descriptive variable names and follow Ghost's naming conventions
- Organize files using Ghost's recommended theme structure

### Ghost Theme Architecture

- Follow Ghost's template hierarchy system for proper template resolution
- Implement proper partial composition and reusability patterns
- Use Ghost helpers for data handling and templating operations
- Leverage Ghost's built-in helpers like `{{content}}` appropriately
- Implement custom helpers when necessary for complex logic

## Environment Setup

### Development Requirements

- Node.js >= 16.0.0
- Ghost CLI >= 1.24.0
- Ghost CMS >= 5.0.0
- npm >= 8.0.0 or yarn >= 1.22.0

### Installation Steps

```bash
# 1. Install Ghost CLI globally
npm install -g ghost-cli

# 2. Create new Ghost installation (local development)
mkdir my-ghost-site
cd my-ghost-site
ghost install local

# 3. Create new theme directory
mkdir content/themes/my-theme
cd content/themes/my-theme

# 4. Initialize theme package.json
npm init -y

# 5. Install development dependencies
npm install -D tailwindcss postcss autoprefixer
npm install -D @tailwindcss/typography @tailwindcss/forms
npm install alpinejs

# 6. Initialize Tailwind CSS
npx tailwindcss init -p

# 7. Start Ghost development server
ghost start
```

### Theme Configuration

```json
{
  "name": "My Ghost Theme",
  "description": "A modern Ghost theme built with Handlebars, Alpine.js, and Tailwind CSS",
  "version": "1.0.0",
  "engines": {
    "ghost": ">=5.0.0",
    "ghost-api": "v5"
  },
  "keywords": ["ghost", "theme", "handlebars", "tailwind", "alpine"],
  "config": {
    "posts_per_page": 10,
    "image_sizes": {
      "xs": {
        "width": 150
      },
      "s": {
        "width": 400
      },
      "m": {
        "width": 750
      },
      "l": {
        "width": 1200
      },
      "xl": {
        "width": 2000
      }
    },
    "card_assets": true,
    "custom": {
      "navigation_layout": {
        "type": "select",
        "options": ["Logo on the left", "Logo in the middle"],
        "default": "Logo on the left"
      },
      "show_publication_cover": {
        "type": "boolean",
        "default": true
      }
    }
  }
}
```

## Core Feature Implementation

### Base Layout Template

```handlebars
{{!-- default.hbs --}}
<!DOCTYPE html>
<html lang="{{@site.locale}}" class="{{body_class}}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{{meta_title}}</title>
    <meta name="description" content="{{meta_description}}">

    {{!-- SEO and Social Meta Tags --}}
    <meta property="og:site_name" content="{{@site.title}}">
    <meta property="og:type" content="{{#is "post"}}article{{else}}website{{/is}}">
    <meta property="og:title" content="{{meta_title}}">
    <meta property="og:description" content="{{meta_description}}">
    <meta property="og:url" content="{{url absolute="true"}}">
    <meta property="og:image" content="{{img_url feature_image size="l" absolute="true"}}">

    {{!-- Canonical URL --}}
    <link rel="canonical" href="{{url absolute="true"}}">

    {{!-- Favicon --}}
    {{#if @site.icon}}
        <link rel="icon" href="{{img_url @site.icon size="s"}}" type="image/png">
    {{/if}}

    {{!-- Styles --}}
    <link rel="stylesheet" type="text/css" href="{{asset "css/screen.css"}}">

    {{!-- Ghost Head --}}
    {{ghost_head}}
</head>
<body class="{{body_class}} bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100">

    {{!-- Site Header --}}
    {{> header}}

    {{!-- Main Content --}}
    <main id="site-main" class="site-main">
        {{{body}}}
    </main>

    {{!-- Site Footer --}}
    {{> footer}}

    {{!-- JavaScript --}}
    <script defer src="{{asset "js/main.js"}}"></script>

    {{!-- Alpine.js --}}
    <script defer src="https://unpkg.com/alpinejs@3.x.x/dist/cdn.min.js"></script>

    {{!-- Ghost Foot --}}
    {{ghost_foot}}
</body>
</html>
```

### Homepage Template

```handlebars
{{!-- index.hbs --}}
{{!< default}}

{{!-- Hero Section --}}
{{#if @site.cover_image}}
<section class="hero relative h-96 bg-cover bg-center"
         style="background-image: url({{img_url @site.cover_image size="xl"}})">
    <div class="absolute inset-0 bg-black bg-opacity-50"></div>
    <div class="relative z-10 container mx-auto px-4 h-full flex items-center justify-center text-center">
        <div class="text-white">
            <h1 class="text-4xl md:text-6xl font-bold mb-4">{{@site.title}}</h1>
            {{#if @site.description}}
                <p class="text-xl md:text-2xl opacity-90">{{@site.description}}</p>
            {{/if}}
        </div>
    </div>
</section>
{{/if}}

{{!-- Featured Posts --}}
{{#get "posts" limit="3" filter="featured:true" as |featured_posts|}}
    {{#if featured_posts}}
    <section class="featured-posts py-16 bg-gray-50 dark:bg-gray-800">
        <div class="container mx-auto px-4">
            <h2 class="text-3xl font-bold text-center mb-12">Featured Posts</h2>
            <div class="grid md:grid-cols-3 gap-8">
                {{#foreach featured_posts}}
                    {{> post-card featured=true}}
                {{/foreach}}
            </div>
        </div>
    </section>
    {{/if}}
{{/get}}

{{!-- Recent Posts --}}
<section class="recent-posts py-16">
    <div class="container mx-auto px-4">
        <h2 class="text-3xl font-bold text-center mb-12">Recent Posts</h2>
        <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-8">
            {{#foreach posts}}
                {{> post-card}}
            {{/foreach}}
        </div>

        {{!-- Pagination --}}
        {{pagination}}
    </div>
</section>
```

### Post Card Component

```handlebars
{{! partials/post-card.hbs }}
<article
  class="post-card bg-white dark:bg-gray-800 rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow duration-300"
>
  {{#if feature_image}}
    <div class="post-card-image">
      <a href="{{url}}" class="block">
        <img
          src="{{img_url feature_image size="m"}}"
          alt="{{title}}"
          class="w-full h-48 object-cover"
          loading="lazy"
        />
      </a>
    </div>
  {{/if}}

  <div class="post-card-content p-6">
    {{! Post Meta }}
    <div
      class="post-card-meta flex items-center text-sm text-gray-600 dark:text-gray-400 mb-3"
    >
      <time datetime="{{date format="YYYY-MM-DD"}}">
        {{date format="MMM DD, YYYY"}}
      </time>
      {{#if reading_time}}
        <span class="mx-2">•</span>
        <span>{{reading_time}}</span>
      {{/if}}
    </div>

    {{! Post Title }}
    <h3 class="post-card-title text-xl font-bold mb-3">
      <a
        href="{{url}}"
        class="text-gray-900 dark:text-gray-100 hover:text-blue-600 dark:hover:text-blue-400 transition-colors"
      >
        {{title}}
      </a>
    </h3>

    {{! Post Excerpt }}
    {{#if excerpt}}
      <p class="post-card-excerpt text-gray-600 dark:text-gray-300 mb-4">
        {{excerpt words="25"}}
      </p>
    {{/if}}

    {{! Post Tags }}
    {{#if tags}}
      <div class="post-card-tags flex flex-wrap gap-2 mb-4">
        {{#foreach tags limit="3"}}
          <a
            href="{{url}}"
            class="inline-block px-3 py-1 text-xs bg-gray-200 dark:bg-gray-700 text-gray-700 dark:text-gray-300 rounded-full hover:bg-gray-300 dark:hover:bg-gray-600 transition-colors"
          >
            {{name}}
          </a>
        {{/foreach}}
      </div>
    {{/if}}

    {{! Author Info }}
    <div class="post-card-author flex items-center">
      {{#if primary_author.profile_image}}
        <img
          src="{{img_url primary_author.profile_image size="xs"}}"
          alt="{{primary_author.name}}"
          class="w-8 h-8 rounded-full mr-3"
        />
      {{/if}}
      <div>
        <a
          href="{{primary_author.url}}"
          class="text-sm font-medium text-gray-900 dark:text-gray-100 hover:text-blue-600 dark:hover:text-blue-400"
        >
          {{primary_author.name}}
        </a>
      </div>
    </div>
  </div>
</article>
```

### Single Post Template

```handlebars
{{!-- post.hbs --}}
{{!< default}}

{{#post}}
<article class="post-full {{post_class}}">
    {{!-- Post Header --}}
    <header class="post-full-header">
        {{#if feature_image}}
        <div class="post-full-image">
            <img src="{{img_url feature_image size="xl"}}"
                 alt="{{title}}"
                 class="w-full h-64 md:h-96 object-cover">
        </div>
        {{/if}}

        <div class="container mx-auto px-4 py-12">
            {{!-- Post Meta --}}
            <div class="post-full-meta text-center mb-6">
                <div class="flex items-center justify-center text-sm text-gray-600 dark:text-gray-400 mb-4">
                    <time datetime="{{date format="YYYY-MM-DD"}}">
                        {{date format="MMMM DD, YYYY"}}
                    </time>
                    {{#if reading_time}}
                        <span class="mx-2">•</span>
                        <span>{{reading_time}}</span>
                    {{/if}}
                </div>

                {{!-- Post Tags --}}
                {{#if tags}}
                <div class="post-full-tags flex flex-wrap justify-center gap-2 mb-6">
                    {{#foreach tags}}
                    <a href="{{url}}" class="inline-block px-3 py-1 text-sm bg-blue-100 dark:bg-blue-900 text-blue-800 dark:text-blue-200 rounded-full hover:bg-blue-200 dark:hover:bg-blue-800 transition-colors">
                        {{name}}
                    </a>
                    {{/foreach}}
                </div>
                {{/if}}
            </div>

            {{!-- Post Title --}}
            <h1 class="post-full-title text-4xl md:text-5xl font-bold text-center mb-8">
                {{title}}
            </h1>

            {{!-- Post Excerpt --}}
            {{#if custom_excerpt}}
            <p class="post-full-excerpt text-xl text-gray-600 dark:text-gray-300 text-center max-w-3xl mx-auto mb-8">
                {{custom_excerpt}}
            </p>
            {{/if}}
        </div>
    </header>

    {{!-- Post Content --}}
    <section class="post-full-content">
        <div class="container mx-auto px-4 max-w-4xl">
            <div class="prose prose-lg dark:prose-invert mx-auto">
                {{content}}
            </div>
        </div>
    </section>

    {{!-- Post Footer --}}
    <footer class="post-full-footer py-12 border-t border-gray-200 dark:border-gray-700 mt-12">
        <div class="container mx-auto px-4">
            {{!-- Author Bio --}}
            {{#if primary_author}}
            <div class="author-card bg-gray-50 dark:bg-gray-800 rounded-lg p-6 mb-8">
                <div class="flex items-start space-x-4">
                    {{#if primary_author.profile_image}}
                    <img src="{{img_url primary_author.profile_image size="s"}}"
                         alt="{{primary_author.name}}"
                         class="w-16 h-16 rounded-full">
                    {{/if}}
                    <div class="flex-1">
                        <h4 class="text-xl font-bold mb-2">
                            <a href="{{primary_author.url}}" class="text-gray-900 dark:text-gray-100 hover:text-blue-600 dark:hover:text-blue-400">
                                {{primary_author.name}}
                            </a>
                        </h4>
                        {{#if primary_author.bio}}
                        <p class="text-gray-600 dark:text-gray-300 mb-3">{{primary_author.bio}}</p>
                        {{/if}}
                        <div class="flex space-x-4">
                            {{#if primary_author.website}}
                            <a href="{{primary_author.website}}" class="text-blue-600 dark:text-blue-400 hover:underline" target="_blank" rel="noopener">
                                Website
                            </a>
                            {{/if}}
                            {{#if primary_author.twitter}}
                            <a href="https://twitter.com/{{primary_author.twitter}}" class="text-blue-600 dark:text-blue-400 hover:underline" target="_blank" rel="noopener">
                                Twitter
                            </a>
                            {{/if}}
                        </div>
                    </div>
                </div>
            </div>
            {{/if}}

            {{!-- Related Posts --}}
            {{#get "posts" limit="3" filter="tags:[{{primary_tag.slug}}]+id:-{{id}}" as |related_posts|}}
                {{#if related_posts}}
                <div class="related-posts">
                    <h3 class="text-2xl font-bold mb-6">Related Posts</h3>
                    <div class="grid md:grid-cols-3 gap-6">
                        {{#foreach related_posts}}
                            {{> post-card}}
                        {{/foreach}}
                    </div>
                </div>
                {{/if}}
            {{/get}}
        </div>
    </footer>
</article>
{{/post}}
```

## Alpine.js Integration

### Dynamic Components

```javascript
// assets/js/components/search.js
document.addEventListener("alpine:init", () => {
  Alpine.data("searchComponent", () => ({
    query: "",
    results: [],
    isLoading: false,
    isOpen: false,

    async search() {
      if (this.query.length < 3) {
        this.results = [];
        return;
      }

      this.isLoading = true;

      try {
        const response = await fetch(
          `/ghost/api/content/posts/?key=${ghostApiKey}&filter=title:~'${this.query}',excerpt:~'${this.query}'&limit=5`
        );
        const data = await response.json();
        this.results = data.posts || [];
      } catch (error) {
        console.error("Search error:", error);
        this.results = [];
      } finally {
        this.isLoading = false;
      }
    },

    selectResult(post) {
      window.location.href = post.url;
    },

    clearSearch() {
      this.query = "";
      this.results = [];
      this.isOpen = false;
    },
  }));
});
```

```handlebars
{{! partials/search.hbs }}
<div x-data="searchComponent" class="relative">
  <div class="relative">
    <input
      type="text"
      x-model="query"
      @input.debounce.300ms="search()"
      @focus="isOpen = true"
      @click.away="isOpen = false"
      placeholder="Search posts..."
      class="w-full px-4 py-2 pl-10 pr-4 text-gray-900 bg-white border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent dark:bg-gray-800 dark:text-gray-100 dark:border-gray-600"
    />

    <div class="absolute inset-y-0 left-0 flex items-center pl-3">
      <svg
        class="w-5 h-5 text-gray-400"
        fill="none"
        stroke="currentColor"
        viewBox="0 0 24 24"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
        ></path>
      </svg>
    </div>
  </div>

  {{! Search Results }}
  <div
    x-show="isOpen && (results.length > 0 || isLoading)"
    x-transition
    class="absolute z-50 w-full mt-1 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 rounded-lg shadow-lg max-h-96 overflow-y-auto"
  >

    {{! Loading State }}
    <div
      x-show="isLoading"
      class="p-4 text-center text-gray-500 dark:text-gray-400"
    >
      <div
        class="animate-spin rounded-full h-6 w-6 border-b-2 border-blue-500 mx-auto"
      ></div>
      <p class="mt-2">Searching...</p>
    </div>

    {{! Results }}
    <template x-for="post in results" :key="post.id">
      <div
        @click="selectResult(post)"
        class="p-4 hover:bg-gray-50 dark:hover:bg-gray-700 cursor-pointer border-b border-gray-200 dark:border-gray-600 last:border-b-0"
      >
        <h4
          class="font-medium text-gray-900 dark:text-gray-100"
          x-text="post.title"
        ></h4>
        <p
          class="text-sm text-gray-600 dark:text-gray-400 mt-1"
          x-text="post.excerpt"
        ></p>
      </div>
    </template>

    {{! No Results }}
    <div
      x-show="!isLoading && results.length === 0 && query.length >= 3"
      class="p-4 text-center text-gray-500 dark:text-gray-400"
    >
      No posts found for "<span x-text="query"></span>"
    </div>
  </div>
</div>
```

### Theme Switcher Component

```javascript
// assets/js/components/theme-switcher.js
document.addEventListener("alpine:init", () => {
  Alpine.data("themeSwitcher", () => ({
    theme: localStorage.getItem("theme") || "light",

    init() {
      this.updateTheme();
    },

    toggleTheme() {
      this.theme = this.theme === "light" ? "dark" : "light";
      this.updateTheme();
    },

    updateTheme() {
      localStorage.setItem("theme", this.theme);

      if (this.theme === "dark") {
        document.documentElement.classList.add("dark");
      } else {
        document.documentElement.classList.remove("dark");
      }
    },
  }));
});
```

## Content API Integration

### Dynamic Content Loading

```javascript
// assets/js/utils/ghost-api.js
class GhostAPI {
  constructor(apiUrl, apiKey) {
    this.apiUrl = apiUrl;
    this.apiKey = apiKey;
  }

  async getPosts(options = {}) {
    const params = new URLSearchParams({
      key: this.apiKey,
      ...options,
    });

    try {
      const response = await fetch(`${this.apiUrl}/posts/?${params}`);
      const data = await response.json();
      return data;
    } catch (error) {
      console.error("Error fetching posts:", error);
      throw error;
    }
  }

  async getPost(slug) {
    try {
      const response = await fetch(
        `${this.apiUrl}/posts/slug/${slug}/?key=${this.apiKey}`
      );
      const data = await response.json();
      return data.posts[0];
    } catch (error) {
      console.error("Error fetching post:", error);
      throw error;
    }
  }

  async getTags(options = {}) {
    const params = new URLSearchParams({
      key: this.apiKey,
      ...options,
    });

    try {
      const response = await fetch(`${this.apiUrl}/tags/?${params}`);
      const data = await response.json();
      return data;
    } catch (error) {
      console.error("Error fetching tags:", error);
      throw error;
    }
  }

  async getAuthors(options = {}) {
    const params = new URLSearchParams({
      key: this.apiKey,
      ...options,
    });

    try {
      const response = await fetch(`${this.apiUrl}/authors/?${params}`);
      const data = await response.json();
      return data;
    } catch (error) {
      console.error("Error fetching authors:", error);
      throw error;
    }
  }
}

// Initialize API client
const ghostAPI = new GhostAPI("/ghost/api/content", window.ghostApiKey);
```

## Performance Optimization

### Asset Optimization

```css
/* assets/css/screen.css */
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";

/* Custom Components */
@layer components {
  .post-card {
    @apply bg-white dark:bg-gray-800 rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow duration-300;
  }

  .btn {
    @apply inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md focus:outline-none focus:ring-2 focus:ring-offset-2 transition-colors duration-200;
  }

  .btn-primary {
    @apply text-white bg-blue-600 hover:bg-blue-700 focus:ring-blue-500;
  }

  .btn-secondary {
    @apply text-gray-700 bg-gray-200 hover:bg-gray-300 focus:ring-gray-500 dark:text-gray-200 dark:bg-gray-700 dark:hover:bg-gray-600;
  }
}

/* Custom Utilities */
@layer utilities {
  .text-balance {
    text-wrap: balance;
  }
}
```

### Image Optimization

```handlebars
{{!-- partials/responsive-image.hbs --}}
<picture class="{{class}}">
    {{#if feature_image}}
    <source media="(min-width: 1024px)"
            srcset="{{img_url feature_image size="xl"}} 1x, {{img_url feature_image size="xxl"}} 2x">
    <source media="(min-width: 768px)"
            srcset="{{img_url feature_image size="l"}} 1x, {{img_url feature_image size="xl"}} 2x">
    <source media="(min-width: 480px)"
            srcset="{{img_url feature_image size="m"}} 1x, {{img_url feature_image size="l"}} 2x">
    <img src="{{img_url feature_image size="m"}}"
         alt="{{alt}}"
         class="w-full h-auto object-cover"
         loading="{{#if eager}}eager{{else}}lazy{{/if}}"
         decoding="async">
    {{/if}}
</picture>
```

### JavaScript Performance

```javascript
// assets/js/main.js
// Lazy load components
const lazyLoadComponents = () => {
  // Intersection Observer for lazy loading
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const element = entry.target;

          // Load images
          if (element.dataset.src) {
            element.src = element.dataset.src;
            element.removeAttribute("data-src");
          }

          // Load components
          if (element.dataset.component) {
            loadComponent(element.dataset.component, element);
          }

          observer.unobserve(element);
        }
      });
    },
    {
      rootMargin: "50px",
    }
  );

  // Observe lazy elements
  document.querySelectorAll("[data-src], [data-component]").forEach((el) => {
    observer.observe(el);
  });
};

// Component loader
const loadComponent = async (componentName, element) => {
  try {
    const module = await import(`./components/${componentName}.js`);
    if (module.default) {
      module.default(element);
    }
  } catch (error) {
    console.error(`Failed to load component: ${componentName}`, error);
  }
};

// Initialize on DOM ready
document.addEventListener("DOMContentLoaded", () => {
  lazyLoadComponents();
});
```

## SEO and Meta Tags

### SEO Optimization

```handlebars
{{!-- partials/meta-tags.hbs --}}
{{!-- Basic Meta Tags --}}
<meta name="description" content="{{meta_description}}">
<meta name="keywords" content="{{#if tags}}{{#foreach tags}}{{name}}{{#unless @last}}, {{/unless}}{{/foreach}}{{/if}}">
<meta name="author" content="{{#if primary_author}}{{primary_author.name}}{{else}}{{@site.title}}{{/if}}">

{{!-- Open Graph Meta Tags --}}
<meta property="og:site_name" content="{{@site.title}}">
<meta property="og:type" content="{{#is "post"}}article{{else}}website{{/is}}">
<meta property="og:title" content="{{meta_title}}">
<meta property="og:description" content="{{meta_description}}">
<meta property="og:url" content="{{url absolute="true"}}">
<meta property="og:image" content="{{img_url feature_image size="l" absolute="true"}}">
{{#if feature_image}}
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
{{/if}}

{{!-- Twitter Card Meta Tags --}}
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="{{meta_title}}">
<meta name="twitter:description" content="{{meta_description}}">
<meta name="twitter:image" content="{{img_url feature_image size="l" absolute="true"}}">
{{#if @site.twitter}}
<meta name="twitter:site" content="@{{@site.twitter}}">
{{/if}}
{{#if primary_author.twitter}}
<meta name="twitter:creator" content="@{{primary_author.twitter}}">
{{/if}}

{{!-- Article Meta Tags --}}
{{#is "post"}}
<meta property="article:published_time" content="{{date format="YYYY-MM-DDTHH:mm:ss.sssZ"}}">
<meta property="article:modified_time" content="{{updated_at format="YYYY-MM-DDTHH:mm:ss.sssZ"}}">
{{#if primary_author}}
<meta property="article:author" content="{{primary_author.name}}">
{{/if}}
{{#if tags}}
{{#foreach tags}}
<meta property="article:tag" content="{{name}}">
{{/foreach}}
{{/if}}
{{/is}}

{{!-- Canonical URL --}}
<link rel="canonical" href="{{url absolute="true"}}">

{{!-- JSON-LD Structured Data --}}
<script type="application/ld+json">
{
    "@context": "https://schema.org",
    "@type": "{{#is "post"}}Article{{else}}WebSite{{/is}}",
    "headline": "{{meta_title}}",
    "description": "{{meta_description}}",
    "url": "{{url absolute="true"}}",
    {{#if feature_image}}
    "image": "{{img_url feature_image size="l" absolute="true"}}",
    {{/if}}
    {{#is "post"}}
    "datePublished": "{{date format="YYYY-MM-DDTHH:mm:ss.sssZ"}}",
    "dateModified": "{{updated_at format="YYYY-MM-DDTHH:mm:ss.sssZ"}}",
    "author": {
        "@type": "Person",
        "name": "{{primary_author.name}}"{{#if primary_author.website}},
        "url": "{{primary_author.website}}"{{/if}}
    },
    "publisher": {
        "@type": "Organization",
        "name": "{{@site.title}}"{{#if @site.logo}},
        "logo": {
            "@type": "ImageObject",
            "url": "{{img_url @site.logo absolute="true"}}"
        }{{/if}}
    },
    {{/is}}
    "mainEntityOfPage": {
        "@type": "WebPage",
        "@id": "{{url absolute="true"}}"
    }
}
</script>
```

## Custom Routes and Templates

### Routes Configuration

```yaml
# routes.yaml
routes:
  /custom-page/: page-custom
  /portfolio/: portfolio
  /contact/: page-contact

collections:
  /blog/:
    permalink: /blog/{slug}/
    template: index
    filter: tag:blog

  /tutorials/:
    permalink: /tutorials/{slug}/
    template: tutorials
    filter: tag:tutorial

  /portfolio/:
    permalink: /portfolio/{slug}/
    template: portfolio-item
    filter: tag:portfolio

taxonomies:
  tag: /tag/{slug}/
  author: /author/{slug}/
```

### Custom Templates

```handlebars
{{! portfolio.hbs }}
{{!< default}}

<div class="portfolio-page">
  <header
    class="page-header py-16 bg-gradient-to-r from-blue-600 to-purple-600 text-white"
  >
    <div class="container mx-auto px-4 text-center">
      <h1 class="text-4xl md:text-6xl font-bold mb-4">Portfolio</h1>
      <p class="text-xl opacity-90">Showcasing our best work</p>
    </div>
  </header>

  <main class="portfolio-content py-16">
    <div class="container mx-auto px-4">
      {{#get "posts" filter="tag:portfolio" limit="all" as |portfolio_posts|}}
        {{#if portfolio_posts}}
          <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-8">
            {{#foreach portfolio_posts}}
              <article class="portfolio-item group">
                <div
                  class="relative overflow-hidden rounded-lg bg-white dark:bg-gray-800 shadow-lg hover:shadow-xl transition-shadow duration-300"
                >
                  {{#if feature_image}}
                    <div class="aspect-w-16 aspect-h-9 overflow-hidden">
                      <img
                        src="{{img_url feature_image size="m"}}"
                        alt="{{title}}"
                        class="w-full h-48 object-cover group-hover:scale-105 transition-transform duration-300"
                      />
                    </div>
                  {{/if}}

                  <div class="p-6">
                    <h3 class="text-xl font-bold mb-2">
                      <a
                        href="{{url}}"
                        class="text-gray-900 dark:text-gray-100 hover:text-blue-600 dark:hover:text-blue-400"
                      >
                        {{title}}
                      </a>
                    </h3>

                    {{#if excerpt}}
                      <p class="text-gray-600 dark:text-gray-300 mb-4">
                        {{excerpt words="20"}}
                      </p>
                    {{/if}}

                    <a
                      href="{{url}}"
                      class="inline-flex items-center text-blue-600 dark:text-blue-400 hover:underline"
                    >
                      View Project
                      <svg
                        class="w-4 h-4 ml-1"
                        fill="none"
                        stroke="currentColor"
                        viewBox="0 0 24 24"
                      >
                        <path
                          stroke-linecap="round"
                          stroke-linejoin="round"
                          stroke-width="2"
                          d="M9 5l7 7-7 7"
                        ></path>
                      </svg>
                    </a>
                  </div>
                </div>
              </article>
            {{/foreach}}
          </div>
        {{else}}
          <div class="text-center py-16">
            <p class="text-gray-600 dark:text-gray-400 text-lg">No portfolio
              items found.</p>
          </div>
        {{/if}}
      {{/get}}
    </div>
  </main>
</div>
```

## Testing and Deployment

### Theme Testing

```bash
# Install GScan for theme validation
npm install -g gscan

# Test theme
gscan /path/to/theme

# Test with specific Ghost version
gscan /path/to/theme --ghost-version 5.0.0
```

### Build Process

```javascript
// build.js
const fs = require("fs");
const path = require("path");
const postcss = require("postcss");
const tailwindcss = require("tailwindcss");
const autoprefixer = require("autoprefixer");
const cssnano = require("cssnano");

async function buildCSS() {
  const css = fs.readFileSync("assets/css/screen.css", "utf8");

  const result = await postcss([
    tailwindcss,
    autoprefixer,
    cssnano({
      preset: "default",
    }),
  ]).process(css, {
    from: "assets/css/screen.css",
    to: "assets/built/screen.css",
  });

  fs.writeFileSync("assets/built/screen.css", result.css);

  if (result.map) {
    fs.writeFileSync("assets/built/screen.css.map", result.map.toString());
  }
}

buildCSS().catch(console.error);
```

```json
{
  "scripts": {
    "build": "node build.js",
    "dev": "node build.js --watch",
    "test": "gscan .",
    "zip": "npm run build && zip -r theme.zip . -x node_modules/\\* .git/\\* *.log"
  }
}
```

### Deployment Checklist

1. **主题验证** - 使用 GScan 验证主题兼容性
2. **性能测试** - 使用 Lighthouse 测试 Core Web Vitals
3. **响应式测试** - 在不同设备上测试布局
4. **SEO 检查** - 验证 meta 标签和结构化数据
5. **可访问性测试** - 确保符合 WCAG 指南
6. **浏览器兼容性** - 测试主流浏览器支持

这个全面的 Ghost CMS 主题开发指南提供了使用 Handlebars、Alpine.js、Tailwind CSS 和 JavaScript 构建现代、高性能 Ghost 主题的完整基础。指南涵盖了从项目设置到部署的所有关键方面，确保开发者能够创建符合最佳实践和 Ghost 平台要求的高质量主题。
