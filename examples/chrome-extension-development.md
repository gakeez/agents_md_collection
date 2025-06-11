---
name: "Chrome Extension Development Guide"
description: "A comprehensive development guide for building modern Chrome extensions using JavaScript/TypeScript, Manifest V3, and browser APIs with best practices for security, performance, and user experience"
category: "Browser Extension"
author: "Agents.md Collection"
authorUrl: "https://github.com/gakeez/agents_md_collection"
tags:
  [
    "chrome-extension",
    "javascript",
    "typescript",
    "manifest-v3",
    "browser-api",
    "web-development",
    "security",
  ]
lastUpdated: "2025-01-02"
---

# Chrome Extension Development Guide

## Project Overview

This comprehensive guide outlines best practices for developing modern Chrome extensions using JavaScript/TypeScript, browser extension APIs, and web development technologies. The guide emphasizes Manifest V3 specifications, security best practices, performance optimization, and creating exceptional user experiences while following Chrome Web Store guidelines.

## Tech Stack

- **Core Languages**: JavaScript/TypeScript with ES6+ features
- **Manifest Version**: Manifest V3 (required for new extensions)
- **Build Tools**: Webpack 5+ or Vite for modern development
- **UI Framework**: Vanilla JS/TS or React for complex UIs
- **Testing**: Jest + Chrome Extension Testing Library
- **Code Quality**: ESLint + Prettier + TypeScript strict mode
- **APIs**: Chrome Extension APIs, Web APIs, Service Workers

## Project Structure

```
chrome-extension/
├── public/
│   ├── manifest.json          # Extension manifest
│   ├── icons/                 # Extension icons
│   │   ├── icon16.png
│   │   ├── icon48.png
│   │   └── icon128.png
│   └── _locales/             # Internationalization
│       └── en/
│           └── messages.json
├── src/
│   ├── background/           # Service Worker scripts
│   │   └── background.ts
│   ├── content/             # Content scripts
│   │   ├── content.ts
│   │   └── content.css
│   ├── popup/               # Extension popup
│   │   ├── popup.html
│   │   ├── popup.ts
│   │   └── popup.css
│   ├── options/             # Options page
│   │   ├── options.html
│   │   ├── options.ts
│   │   └── options.css
│   ├── utils/               # Shared utilities
│   │   ├── storage.ts
│   │   ├── messaging.ts
│   │   └── constants.ts
│   └── types/               # TypeScript definitions
│       └── chrome.d.ts
├── tests/                   # Test files
├── webpack.config.js        # Build configuration
├── tsconfig.json           # TypeScript configuration
└── package.json
```

## Development Guidelines

### Key Principles

- Write clear, modular TypeScript code with proper type definitions
- Follow functional programming patterns; avoid classes unless necessary
- Use descriptive variable names (e.g., `isLoading`, `hasPermission`)
- Structure files logically: popup, background, content scripts, utils
- Implement proper error handling and logging throughout the extension
- Document code with JSDoc comments for better maintainability

### Architecture and Best Practices

- Strictly follow Manifest V3 specifications and requirements
- Divide responsibilities between background, content scripts and popup components
- Configure permissions following the principle of least privilege
- Use modern build tools (webpack/vite) for efficient development workflow
- Implement proper version control and change management processes

## Environment Setup

### Development Requirements

- Node.js >= 16.0.0
- npm >= 8.0.0 or yarn >= 1.22.0
- Chrome Browser >= 88 (for Manifest V3 support)
- TypeScript >= 4.5.0 (if using TypeScript)

### Installation Steps

```bash
# 1. Create project directory
mkdir my-chrome-extension
cd my-chrome-extension

# 2. Initialize npm project
npm init -y

# 3. Install development dependencies
npm install -D webpack webpack-cli webpack-merge
npm install -D typescript ts-loader @types/chrome
npm install -D copy-webpack-plugin html-webpack-plugin
npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
npm install -D prettier jest @types/jest

# 4. Install runtime dependencies (if needed)
npm install webextension-polyfill

# 5. Create basic project structure
mkdir -p src/{background,content,popup,options,utils,types}
mkdir -p public/{icons,_locales/en}
```

### Manifest V3 Configuration

```json
{
  "manifest_version": 3,
  "name": "My Chrome Extension",
  "version": "1.0.0",
  "description": "A modern Chrome extension built with Manifest V3",
  "permissions": ["storage", "activeTab"],
  "host_permissions": ["https://*.example.com/*"],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["https://*.example.com/*"],
      "js": ["content.js"],
      "css": ["content.css"]
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_title": "My Extension",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "options_page": "options.html",
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  },
  "web_accessible_resources": [
    {
      "resources": ["injected.js"],
      "matches": ["https://*.example.com/*"]
    }
  ]
}
```

## Core Feature Implementation

### Chrome API Usage

#### Storage API Implementation

```typescript
// utils/storage.ts
interface StorageData {
  settings: {
    theme: "light" | "dark";
    notifications: boolean;
  };
  userData: {
    preferences: string[];
    lastSync: number;
  };
}

class ExtensionStorage {
  // Get data from storage
  static async get<K extends keyof StorageData>(
    key: K
  ): Promise<StorageData[K] | null> {
    try {
      const result = await chrome.storage.sync.get(key);
      return result[key] || null;
    } catch (error) {
      console.error(`Error getting storage key ${key}:`, error);
      return null;
    }
  }

  // Set data in storage
  static async set<K extends keyof StorageData>(
    key: K,
    value: StorageData[K]
  ): Promise<boolean> {
    try {
      await chrome.storage.sync.set({ [key]: value });
      return true;
    } catch (error) {
      console.error(`Error setting storage key ${key}:`, error);
      return false;
    }
  }

  // Remove data from storage
  static async remove(key: keyof StorageData): Promise<boolean> {
    try {
      await chrome.storage.sync.remove(key);
      return true;
    } catch (error) {
      console.error(`Error removing storage key ${key}:`, error);
      return false;
    }
  }

  // Listen for storage changes
  static onChanged(
    callback: (changes: { [key: string]: chrome.storage.StorageChange }) => void
  ): void {
    chrome.storage.onChanged.addListener(callback);
  }
}

export { ExtensionStorage };
```

#### Messaging System

```typescript
// utils/messaging.ts
interface MessageTypes {
  GET_TAB_INFO: { tabId: number };
  UPDATE_BADGE: { text: string; color: string };
  CONTENT_SCRIPT_READY: { url: string };
  POPUP_OPENED: { timestamp: number };
}

type MessageType = keyof MessageTypes;
type MessagePayload<T extends MessageType> = MessageTypes[T];

class ExtensionMessaging {
  // Send message to background script
  static async sendToBackground<T extends MessageType>(
    type: T,
    payload: MessagePayload<T>
  ): Promise<any> {
    try {
      return await chrome.runtime.sendMessage({ type, payload });
    } catch (error) {
      console.error("Error sending message to background:", error);
      throw error;
    }
  }

  // Send message to content script
  static async sendToContentScript<T extends MessageType>(
    tabId: number,
    type: T,
    payload: MessagePayload<T>
  ): Promise<any> {
    try {
      return await chrome.tabs.sendMessage(tabId, { type, payload });
    } catch (error) {
      console.error("Error sending message to content script:", error);
      throw error;
    }
  }

  // Listen for messages
  static onMessage(
    callback: (
      message: { type: MessageType; payload: any },
      sender: chrome.runtime.MessageSender,
      sendResponse: (response?: any) => void
    ) => void | Promise<void>
  ): void {
    chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
      const result = callback(message, sender, sendResponse);

      // Handle async callbacks
      if (result instanceof Promise) {
        result.then(sendResponse).catch(console.error);
        return true; // Keep message channel open
      }
    });
  }
}

export { ExtensionMessaging };
```

### Service Worker (Background Script)

```typescript
// background/background.ts
import { ExtensionStorage } from "../utils/storage";
import { ExtensionMessaging } from "../utils/messaging";

class BackgroundService {
  private static instance: BackgroundService;

  private constructor() {
    this.initialize();
  }

  static getInstance(): BackgroundService {
    if (!BackgroundService.instance) {
      BackgroundService.instance = new BackgroundService();
    }
    return BackgroundService.instance;
  }

  private async initialize(): Promise<void> {
    // Set up message listeners
    this.setupMessageListeners();

    // Set up alarm listeners
    this.setupAlarmListeners();

    // Set up tab listeners
    this.setupTabListeners();

    // Initialize default settings
    await this.initializeDefaultSettings();

    console.log("Background service initialized");
  }

  private setupMessageListeners(): void {
    ExtensionMessaging.onMessage(async (message, sender, sendResponse) => {
      try {
        switch (message.type) {
          case "GET_TAB_INFO":
            const tabInfo = await this.getTabInfo(message.payload.tabId);
            sendResponse(tabInfo);
            break;

          case "UPDATE_BADGE":
            await this.updateBadge(message.payload.text, message.payload.color);
            sendResponse({ success: true });
            break;

          default:
            console.warn("Unknown message type:", message.type);
        }
      } catch (error) {
        console.error("Error handling message:", error);
        sendResponse({ error: error.message });
      }
    });
  }

  private setupAlarmListeners(): void {
    chrome.alarms.onAlarm.addListener(async (alarm) => {
      switch (alarm.name) {
        case "sync-data":
          await this.syncData();
          break;
        case "cleanup":
          await this.performCleanup();
          break;
      }
    });
  }

  private setupTabListeners(): void {
    chrome.tabs.onUpdated.addListener(async (tabId, changeInfo, tab) => {
      if (changeInfo.status === "complete" && tab.url) {
        await this.handleTabUpdate(tabId, tab);
      }
    });
  }

  private async initializeDefaultSettings(): Promise<void> {
    const settings = await ExtensionStorage.get("settings");
    if (!settings) {
      await ExtensionStorage.set("settings", {
        theme: "light",
        notifications: true,
      });
    }
  }

  private async getTabInfo(tabId: number): Promise<chrome.tabs.Tab | null> {
    try {
      return await chrome.tabs.get(tabId);
    } catch (error) {
      console.error("Error getting tab info:", error);
      return null;
    }
  }

  private async updateBadge(text: string, color: string): Promise<void> {
    await chrome.action.setBadgeText({ text });
    await chrome.action.setBadgeBackgroundColor({ color });
  }

  private async syncData(): Promise<void> {
    // Implement data synchronization logic
    console.log("Syncing data...");
  }

  private async performCleanup(): Promise<void> {
    // Implement cleanup logic
    console.log("Performing cleanup...");
  }

  private async handleTabUpdate(
    tabId: number,
    tab: chrome.tabs.Tab
  ): Promise<void> {
    // Handle tab updates
    console.log("Tab updated:", tabId, tab.url);
  }
}

// Initialize background service
BackgroundService.getInstance();
```

### Content Script Implementation

```typescript
// content/content.ts
import { ExtensionMessaging } from "../utils/messaging";

class ContentScript {
  private isInitialized = false;
  private observer: MutationObserver | null = null;

  constructor() {
    this.initialize();
  }

  private async initialize(): Promise<void> {
    if (this.isInitialized) return;

    try {
      // Wait for DOM to be ready
      if (document.readyState === "loading") {
        document.addEventListener("DOMContentLoaded", () => this.setup());
      } else {
        this.setup();
      }

      this.isInitialized = true;
    } catch (error) {
      console.error("Error initializing content script:", error);
    }
  }

  private setup(): void {
    // Set up message listeners
    this.setupMessageListeners();

    // Set up DOM observers
    this.setupDOMObserver();

    // Inject custom functionality
    this.injectCustomFeatures();

    // Notify background script
    ExtensionMessaging.sendToBackground("CONTENT_SCRIPT_READY", {
      url: window.location.href,
    });

    console.log("Content script initialized on:", window.location.href);
  }

  private setupMessageListeners(): void {
    ExtensionMessaging.onMessage(async (message, sender, sendResponse) => {
      try {
        switch (message.type) {
          case "GET_PAGE_DATA":
            const pageData = this.extractPageData();
            sendResponse(pageData);
            break;

          default:
            console.warn(
              "Unknown message type in content script:",
              message.type
            );
        }
      } catch (error) {
        console.error("Error handling message in content script:", error);
        sendResponse({ error: error.message });
      }
    });
  }

  private setupDOMObserver(): void {
    this.observer = new MutationObserver((mutations) => {
      mutations.forEach((mutation) => {
        if (mutation.type === "childList") {
          this.handleDOMChanges(mutation);
        }
      });
    });

    this.observer.observe(document.body, {
      childList: true,
      subtree: true,
    });
  }

  private injectCustomFeatures(): void {
    // Add custom CSS
    this.injectCSS();

    // Add custom UI elements
    this.addCustomUI();

    // Set up event listeners
    this.setupEventListeners();
  }

  private injectCSS(): void {
    const style = document.createElement("style");
    style.textContent = `
      .extension-highlight {
        background-color: #ffeb3b !important;
        transition: background-color 0.3s ease;
      }

      .extension-tooltip {
        position: absolute;
        background: #333;
        color: white;
        padding: 8px 12px;
        border-radius: 4px;
        font-size: 12px;
        z-index: 10000;
        pointer-events: none;
      }
    `;
    document.head.appendChild(style);
  }

  private addCustomUI(): void {
    // Add floating action button or other UI elements
    const floatingButton = document.createElement("div");
    floatingButton.id = "extension-floating-button";
    floatingButton.innerHTML = "🔧";
    floatingButton.style.cssText = `
      position: fixed;
      bottom: 20px;
      right: 20px;
      width: 50px;
      height: 50px;
      background: #4285f4;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      z-index: 10000;
      box-shadow: 0 2px 10px rgba(0,0,0,0.2);
      transition: transform 0.2s ease;
    `;

    floatingButton.addEventListener("mouseenter", () => {
      floatingButton.style.transform = "scale(1.1)";
    });

    floatingButton.addEventListener("mouseleave", () => {
      floatingButton.style.transform = "scale(1)";
    });

    floatingButton.addEventListener("click", () => {
      this.handleFloatingButtonClick();
    });

    document.body.appendChild(floatingButton);
  }

  private setupEventListeners(): void {
    // Listen for keyboard shortcuts
    document.addEventListener("keydown", (event) => {
      if (event.ctrlKey && event.shiftKey && event.key === "E") {
        event.preventDefault();
        this.toggleExtensionFeatures();
      }
    });
  }

  private handleDOMChanges(mutation: MutationRecord): void {
    // Handle dynamic content changes
    mutation.addedNodes.forEach((node) => {
      if (node.nodeType === Node.ELEMENT_NODE) {
        this.processNewElement(node as Element);
      }
    });
  }

  private processNewElement(element: Element): void {
    // Process newly added elements
    if (element.matches(".target-selector")) {
      this.enhanceElement(element);
    }
  }

  private enhanceElement(element: Element): void {
    // Add enhancement to specific elements
    element.classList.add("extension-enhanced");
  }

  private extractPageData(): any {
    return {
      title: document.title,
      url: window.location.href,
      timestamp: Date.now(),
      elementCount: document.querySelectorAll("*").length,
    };
  }

  private handleFloatingButtonClick(): void {
    // Handle floating button click
    console.log("Extension floating button clicked");
  }

  private toggleExtensionFeatures(): void {
    // Toggle extension features on/off
    console.log("Toggling extension features");
  }

  // Cleanup method
  destroy(): void {
    if (this.observer) {
      this.observer.disconnect();
    }

    // Remove injected elements
    const floatingButton = document.getElementById("extension-floating-button");
    if (floatingButton) {
      floatingButton.remove();
    }
  }
}

// Initialize content script
new ContentScript();
```

### Popup Implementation

```typescript
// popup/popup.ts
import { ExtensionStorage } from "../utils/storage";
import { ExtensionMessaging } from "../utils/messaging";

interface PopupState {
  isLoading: boolean;
  currentTab: chrome.tabs.Tab | null;
  settings: any;
  error: string | null;
}

class PopupController {
  private state: PopupState = {
    isLoading: true,
    currentTab: null,
    settings: null,
    error: null,
  };

  constructor() {
    this.initialize();
  }

  private async initialize(): Promise<void> {
    try {
      // Wait for DOM to be ready
      if (document.readyState === "loading") {
        document.addEventListener("DOMContentLoaded", () => this.setup());
      } else {
        this.setup();
      }
    } catch (error) {
      this.handleError("Failed to initialize popup", error);
    }
  }

  private async setup(): Promise<void> {
    try {
      // Load current tab info
      await this.loadCurrentTab();

      // Load settings
      await this.loadSettings();

      // Set up event listeners
      this.setupEventListeners();

      // Render initial UI
      this.render();

      // Notify background script
      ExtensionMessaging.sendToBackground("POPUP_OPENED", {
        timestamp: Date.now(),
      });
    } catch (error) {
      this.handleError("Failed to setup popup", error);
    } finally {
      this.state.isLoading = false;
      this.render();
    }
  }

  private async loadCurrentTab(): Promise<void> {
    const [tab] = await chrome.tabs.query({
      active: true,
      currentWindow: true,
    });
    this.state.currentTab = tab;
  }

  private async loadSettings(): Promise<void> {
    this.state.settings = await ExtensionStorage.get("settings");
  }

  private setupEventListeners(): void {
    // Theme toggle
    const themeToggle = document.getElementById(
      "theme-toggle"
    ) as HTMLInputElement;
    if (themeToggle) {
      themeToggle.addEventListener("change", (e) => {
        this.handleThemeToggle((e.target as HTMLInputElement).checked);
      });
    }

    // Notifications toggle
    const notificationsToggle = document.getElementById(
      "notifications-toggle"
    ) as HTMLInputElement;
    if (notificationsToggle) {
      notificationsToggle.addEventListener("change", (e) => {
        this.handleNotificationsToggle((e.target as HTMLInputElement).checked);
      });
    }

    // Action buttons
    const actionButton = document.getElementById("action-button");
    if (actionButton) {
      actionButton.addEventListener("click", () => this.handleActionClick());
    }

    const optionsButton = document.getElementById("options-button");
    if (optionsButton) {
      optionsButton.addEventListener("click", () => this.openOptionsPage());
    }
  }

  private async handleThemeToggle(isDark: boolean): Promise<void> {
    try {
      const settings = {
        ...this.state.settings,
        theme: isDark ? "dark" : "light",
      };
      await ExtensionStorage.set("settings", settings);
      this.state.settings = settings;
      this.updateTheme();
    } catch (error) {
      this.handleError("Failed to update theme", error);
    }
  }

  private async handleNotificationsToggle(enabled: boolean): Promise<void> {
    try {
      const settings = { ...this.state.settings, notifications: enabled };
      await ExtensionStorage.set("settings", settings);
      this.state.settings = settings;
    } catch (error) {
      this.handleError("Failed to update notifications setting", error);
    }
  }

  private async handleActionClick(): Promise<void> {
    try {
      if (!this.state.currentTab?.id) return;

      // Send message to content script
      const response = await ExtensionMessaging.sendToContentScript(
        this.state.currentTab.id,
        "PERFORM_ACTION",
        { action: "highlight" }
      );

      if (response?.success) {
        this.showSuccess("Action performed successfully");
      }
    } catch (error) {
      this.handleError("Failed to perform action", error);
    }
  }

  private openOptionsPage(): void {
    chrome.runtime.openOptionsPage();
    window.close();
  }

  private render(): void {
    this.updateLoadingState();
    this.updateTabInfo();
    this.updateSettings();
    this.updateTheme();
  }

  private updateLoadingState(): void {
    const loadingElement = document.getElementById("loading");
    const contentElement = document.getElementById("content");

    if (loadingElement && contentElement) {
      if (this.state.isLoading) {
        loadingElement.style.display = "block";
        contentElement.style.display = "none";
      } else {
        loadingElement.style.display = "none";
        contentElement.style.display = "block";
      }
    }
  }

  private updateTabInfo(): void {
    const tabTitleElement = document.getElementById("tab-title");
    const tabUrlElement = document.getElementById("tab-url");

    if (tabTitleElement && this.state.currentTab) {
      tabTitleElement.textContent = this.state.currentTab.title || "Unknown";
    }

    if (tabUrlElement && this.state.currentTab) {
      tabUrlElement.textContent = this.state.currentTab.url || "Unknown";
    }
  }

  private updateSettings(): void {
    if (!this.state.settings) return;

    const themeToggle = document.getElementById(
      "theme-toggle"
    ) as HTMLInputElement;
    if (themeToggle) {
      themeToggle.checked = this.state.settings.theme === "dark";
    }

    const notificationsToggle = document.getElementById(
      "notifications-toggle"
    ) as HTMLInputElement;
    if (notificationsToggle) {
      notificationsToggle.checked = this.state.settings.notifications;
    }
  }

  private updateTheme(): void {
    const isDark = this.state.settings?.theme === "dark";
    document.body.classList.toggle("dark-theme", isDark);
  }

  private showSuccess(message: string): void {
    this.showMessage(message, "success");
  }

  private showMessage(message: string, type: "success" | "error"): void {
    const messageElement = document.getElementById("message");
    if (messageElement) {
      messageElement.textContent = message;
      messageElement.className = `message ${type}`;
      messageElement.style.display = "block";

      setTimeout(() => {
        messageElement.style.display = "none";
      }, 3000);
    }
  }

  private handleError(message: string, error: any): void {
    console.error(message, error);
    this.state.error = message;
    this.showMessage(message, "error");
  }
}

// Initialize popup
new PopupController();
```

```html
<!-- popup/popup.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Extension Popup</title>
    <link rel="stylesheet" href="popup.css" />
  </head>
  <body>
    <div id="loading" class="loading">
      <div class="spinner"></div>
      <p>Loading...</p>
    </div>

    <div id="content" class="content" style="display: none;">
      <header class="header">
        <h1>My Extension</h1>
        <div id="message" class="message" style="display: none;"></div>
      </header>

      <main class="main">
        <section class="tab-info">
          <h2>Current Tab</h2>
          <p id="tab-title" class="tab-title">Loading...</p>
          <p id="tab-url" class="tab-url">Loading...</p>
        </section>

        <section class="settings">
          <h2>Settings</h2>
          <div class="setting-item">
            <label for="theme-toggle">Dark Theme</label>
            <input type="checkbox" id="theme-toggle" class="toggle" />
          </div>
          <div class="setting-item">
            <label for="notifications-toggle">Notifications</label>
            <input type="checkbox" id="notifications-toggle" class="toggle" />
          </div>
        </section>

        <section class="actions">
          <button id="action-button" class="btn btn-primary">
            Perform Action
          </button>
          <button id="options-button" class="btn btn-secondary">Options</button>
        </section>
      </main>
    </div>

    <script src="popup.js"></script>
  </body>
</html>
```

## Security and Privacy

### Content Security Policy (CSP)

```json
// manifest.json - CSP configuration
{
  "content_security_policy": {
    "extension_pages": "script-src 'self'; object-src 'self'; frame-ancestors 'none';"
  }
}
```

### Secure Data Handling

```typescript
// utils/security.ts
class SecurityUtils {
  // Sanitize user input to prevent XSS
  static sanitizeInput(input: string): string {
    const div = document.createElement("div");
    div.textContent = input;
    return div.innerHTML;
  }

  // Validate URLs before making requests
  static isValidUrl(url: string): boolean {
    try {
      const urlObj = new URL(url);
      return ["http:", "https:"].includes(urlObj.protocol);
    } catch {
      return false;
    }
  }

  // Encrypt sensitive data before storage
  static async encryptData(data: string, key: string): Promise<string> {
    const encoder = new TextEncoder();
    const keyData = encoder.encode(key);
    const dataBuffer = encoder.encode(data);

    const cryptoKey = await crypto.subtle.importKey(
      "raw",
      keyData,
      { name: "AES-GCM" },
      false,
      ["encrypt"]
    );

    const iv = crypto.getRandomValues(new Uint8Array(12));
    const encrypted = await crypto.subtle.encrypt(
      { name: "AES-GCM", iv },
      cryptoKey,
      dataBuffer
    );

    const result = new Uint8Array(iv.length + encrypted.byteLength);
    result.set(iv);
    result.set(new Uint8Array(encrypted), iv.length);

    return btoa(String.fromCharCode(...result));
  }

  // Decrypt sensitive data from storage
  static async decryptData(
    encryptedData: string,
    key: string
  ): Promise<string> {
    const encoder = new TextEncoder();
    const decoder = new TextDecoder();
    const keyData = encoder.encode(key);

    const data = new Uint8Array(
      atob(encryptedData)
        .split("")
        .map((char) => char.charCodeAt(0))
    );

    const iv = data.slice(0, 12);
    const encrypted = data.slice(12);

    const cryptoKey = await crypto.subtle.importKey(
      "raw",
      keyData,
      { name: "AES-GCM" },
      false,
      ["decrypt"]
    );

    const decrypted = await crypto.subtle.decrypt(
      { name: "AES-GCM", iv },
      cryptoKey,
      encrypted
    );

    return decoder.decode(decrypted);
  }
}

export { SecurityUtils };
```

### Permission Management

```typescript
// utils/permissions.ts
class PermissionManager {
  // Request additional permissions at runtime
  static async requestPermission(permission: string): Promise<boolean> {
    try {
      return await chrome.permissions.request({
        permissions: [permission],
      });
    } catch (error) {
      console.error("Error requesting permission:", error);
      return false;
    }
  }

  // Check if permission is granted
  static async hasPermission(permission: string): Promise<boolean> {
    try {
      return await chrome.permissions.contains({
        permissions: [permission],
      });
    } catch (error) {
      console.error("Error checking permission:", error);
      return false;
    }
  }

  // Remove permission
  static async removePermission(permission: string): Promise<boolean> {
    try {
      return await chrome.permissions.remove({
        permissions: [permission],
      });
    } catch (error) {
      console.error("Error removing permission:", error);
      return false;
    }
  }
}

export { PermissionManager };
```

## Performance and Optimization

### Memory Management

```typescript
// utils/performance.ts
class PerformanceManager {
  private static memoryUsage: Map<string, number> = new Map();

  // Monitor memory usage
  static trackMemoryUsage(label: string): void {
    if ("memory" in performance) {
      const memory = (performance as any).memory;
      this.memoryUsage.set(label, memory.usedJSHeapSize);
    }
  }

  // Get memory usage difference
  static getMemoryDiff(startLabel: string, endLabel: string): number {
    const start = this.memoryUsage.get(startLabel) || 0;
    const end = this.memoryUsage.get(endLabel) || 0;
    return end - start;
  }

  // Debounce function calls
  static debounce<T extends (...args: any[]) => any>(
    func: T,
    delay: number
  ): (...args: Parameters<T>) => void {
    let timeoutId: NodeJS.Timeout;
    return (...args: Parameters<T>) => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => func(...args), delay);
    };
  }

  // Throttle function calls
  static throttle<T extends (...args: any[]) => any>(
    func: T,
    limit: number
  ): (...args: Parameters<T>) => void {
    let inThrottle: boolean;
    return (...args: Parameters<T>) => {
      if (!inThrottle) {
        func(...args);
        inThrottle = true;
        setTimeout(() => (inThrottle = false), limit);
      }
    };
  }

  // Cache function results
  static memoize<T extends (...args: any[]) => any>(
    func: T
  ): (...args: Parameters<T>) => ReturnType<T> {
    const cache = new Map();
    return (...args: Parameters<T>): ReturnType<T> => {
      const key = JSON.stringify(args);
      if (cache.has(key)) {
        return cache.get(key);
      }
      const result = func(...args);
      cache.set(key, result);
      return result;
    };
  }
}

export { PerformanceManager };
```

## Testing Strategy

### Unit Testing with Jest

```typescript
// tests/storage.test.ts
import { ExtensionStorage } from "../src/utils/storage";

// Mock Chrome APIs
const mockChrome = {
  storage: {
    sync: {
      get: jest.fn(),
      set: jest.fn(),
      remove: jest.fn(),
    },
    onChanged: {
      addListener: jest.fn(),
    },
  },
};

(global as any).chrome = mockChrome;

describe("ExtensionStorage", () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test("should get data from storage", async () => {
    const mockData = { settings: { theme: "dark" } };
    mockChrome.storage.sync.get.mockResolvedValue(mockData);

    const result = await ExtensionStorage.get("settings");

    expect(mockChrome.storage.sync.get).toHaveBeenCalledWith("settings");
    expect(result).toEqual(mockData.settings);
  });

  test("should set data in storage", async () => {
    const testData = { theme: "light", notifications: true };
    mockChrome.storage.sync.set.mockResolvedValue(undefined);

    const result = await ExtensionStorage.set("settings", testData);

    expect(mockChrome.storage.sync.set).toHaveBeenCalledWith({
      settings: testData,
    });
    expect(result).toBe(true);
  });

  test("should handle storage errors gracefully", async () => {
    mockChrome.storage.sync.get.mockRejectedValue(new Error("Storage error"));

    const result = await ExtensionStorage.get("settings");

    expect(result).toBeNull();
  });
});
```

### Integration Testing

```typescript
// tests/integration/popup.test.ts
import { JSDOM } from "jsdom";

describe("Popup Integration", () => {
  let dom: JSDOM;
  let document: Document;

  beforeEach(() => {
    dom = new JSDOM(`
      <!DOCTYPE html>
      <html>
        <body>
          <div id="theme-toggle"></div>
          <div id="content"></div>
        </body>
      </html>
    `);
    document = dom.window.document;
    (global as any).document = document;
    (global as any).window = dom.window;
  });

  test("should initialize popup correctly", async () => {
    // Mock Chrome APIs
    (global as any).chrome = {
      tabs: {
        query: jest.fn().mockResolvedValue([{ id: 1, title: "Test Tab" }]),
      },
      storage: {
        sync: {
          get: jest.fn().mockResolvedValue({ settings: { theme: "light" } }),
        },
      },
    };

    // Import and test popup functionality
    const { PopupController } = await import("../src/popup/popup");

    // Test initialization
    expect(document.getElementById("content")).toBeTruthy();
  });
});
```

## Internationalization

### Localization Setup

```json
// public/_locales/en/messages.json
{
  "extensionName": {
    "message": "My Chrome Extension",
    "description": "Name of the extension"
  },
  "extensionDescription": {
    "message": "A powerful Chrome extension for enhanced browsing",
    "description": "Description of the extension"
  },
  "popupTitle": {
    "message": "Extension Settings",
    "description": "Title shown in popup"
  },
  "darkTheme": {
    "message": "Dark Theme",
    "description": "Label for dark theme toggle"
  },
  "notifications": {
    "message": "Notifications",
    "description": "Label for notifications toggle"
  },
  "performAction": {
    "message": "Perform Action",
    "description": "Button text for main action"
  }
}
```

```typescript
// utils/i18n.ts
class I18nManager {
  // Get localized message
  static getMessage(key: string, substitutions?: string[]): string {
    return chrome.i18n.getMessage(key, substitutions);
  }

  // Get current locale
  static getLocale(): string {
    return chrome.i18n.getUILanguage();
  }

  // Check if locale is RTL
  static isRTL(): boolean {
    const rtlLocales = ["ar", "he", "fa", "ur"];
    const currentLocale = this.getLocale().split("-")[0];
    return rtlLocales.includes(currentLocale);
  }

  // Localize HTML elements
  static localizeHTML(): void {
    document.querySelectorAll("[data-i18n]").forEach((element) => {
      const key = element.getAttribute("data-i18n");
      if (key) {
        element.textContent = this.getMessage(key);
      }
    });

    document.querySelectorAll("[data-i18n-placeholder]").forEach((element) => {
      const key = element.getAttribute("data-i18n-placeholder");
      if (key && element instanceof HTMLInputElement) {
        element.placeholder = this.getMessage(key);
      }
    });
  }
}

export { I18nManager };
```

## Build and Deployment

### Webpack Configuration

```javascript
// webpack.config.js
const path = require("path");
const CopyWebpackPlugin = require("copy-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "production",
  entry: {
    background: "./src/background/background.ts",
    content: "./src/content/content.ts",
    popup: "./src/popup/popup.ts",
    options: "./src/options/options.ts",
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].js",
    clean: true,
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: "ts-loader",
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
  resolve: {
    extensions: [".tsx", ".ts", ".js"],
  },
  plugins: [
    new CopyWebpackPlugin({
      patterns: [
        { from: "public/manifest.json", to: "manifest.json" },
        { from: "public/icons", to: "icons" },
        { from: "public/_locales", to: "_locales" },
        { from: "src/content/content.css", to: "content.css" },
      ],
    }),
    new HtmlWebpackPlugin({
      template: "./src/popup/popup.html",
      filename: "popup.html",
      chunks: ["popup"],
    }),
    new HtmlWebpackPlugin({
      template: "./src/options/options.html",
      filename: "options.html",
      chunks: ["options"],
    }),
  ],
};
```

### Package.json Scripts

```json
{
  "scripts": {
    "build": "webpack --mode=production",
    "dev": "webpack --mode=development --watch",
    "test": "jest",
    "test:watch": "jest --watch",
    "lint": "eslint src/**/*.{ts,tsx}",
    "lint:fix": "eslint src/**/*.{ts,tsx} --fix",
    "type-check": "tsc --noEmit",
    "package": "npm run build && zip -r extension.zip dist/"
  }
}
```

## Publishing and Maintenance

### Chrome Web Store Preparation

1. **Create high-quality screenshots** (1280x800 or 640x400)
2. **Write compelling store description** with keywords
3. **Prepare privacy policy** if handling user data
4. **Test thoroughly** across different Chrome versions
5. **Follow Chrome Web Store policies** strictly

### Update Management

```typescript
// background/updates.ts
class UpdateManager {
  static handleInstallOrUpdate(): void {
    chrome.runtime.onInstalled.addListener(async (details) => {
      switch (details.reason) {
        case "install":
          await this.handleFirstInstall();
          break;
        case "update":
          await this.handleUpdate(details.previousVersion);
          break;
      }
    });
  }

  private static async handleFirstInstall(): Promise<void> {
    // Set up default settings
    await ExtensionStorage.set("settings", {
      theme: "light",
      notifications: true,
    });

    // Open welcome page
    chrome.tabs.create({
      url: chrome.runtime.getURL("welcome.html"),
    });
  }

  private static async handleUpdate(previousVersion?: string): Promise<void> {
    // Perform migration if needed
    if (previousVersion && this.needsMigration(previousVersion)) {
      await this.performMigration(previousVersion);
    }

    // Show update notification
    chrome.notifications.create({
      type: "basic",
      iconUrl: "icons/icon48.png",
      title: "Extension Updated",
      message: "Your extension has been updated to the latest version!",
    });
  }

  private static needsMigration(previousVersion: string): boolean {
    // Check if migration is needed based on version
    return previousVersion < "2.0.0";
  }

  private static async performMigration(
    previousVersion: string
  ): Promise<void> {
    // Perform data migration
    console.log(`Migrating from version ${previousVersion}`);
  }
}

// Initialize update manager
UpdateManager.handleInstallOrUpdate();
```

This comprehensive Chrome Extension Development Guide provides a complete foundation for building modern, secure, and performant Chrome extensions using TypeScript and Manifest V3. The guide covers all essential aspects from project setup to deployment, ensuring developers can create high-quality extensions that follow best practices and Chrome Web Store guidelines.
