# Min Browser Architecture

## Overview

Min is a privacy-focused web browser built on **Electron** and **Chromium**. It uses a standard Electron architecture with a main process (Node.js) and renderer processes (web pages), communicating via IPC (Inter-Process Communication).

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Main Process                          │
│  (main/*.js - Node.js environment)                          │
│                                                               │
│  • Application lifecycle management                          │
│  • Window management                                         │
│  • Menu system                                               │
│  • View (WebContentsView) management                         │
│  • Content filtering                                         │
│  • Download handling                                         │
│  • Permission management                                     │
│  • Protocol handlers                                         │
└─────────────────────────────────────────────────────────────┘
                            ↕ IPC
┌─────────────────────────────────────────────────────────────┐
│                      Renderer Process                        │
│  (js/*.js - Browser UI environment)                         │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Browser UI (index.html)                  │  │
│  │                                                         │  │
│  │  • Tab Bar                                             │  │
│  │  • Searchbar                                           │  │
│  │  • Task Overlay                                        │  │
│  │  • Settings                                            │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                    Web Content Views                         │
│  (WebContentsView instances - isolated contexts)            │
│                                                               │
│  • Each tab runs in its own WebContentsView                 │
│  • Sandboxed with context isolation                         │
│  • Preload scripts for controlled access                    │
│  • Content security and filtering                           │
└─────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Main Process (main/*.js)

The main process is the application's backend, running in a Node.js environment.

#### Key Files:

- **`main.js`**: Application entry point
  - Initializes Electron app
  - Manages application lifecycle
  - Creates windows
  - Handles single-instance locking
  - Manages window bounds persistence

- **`viewManager.js`**: Manages web content views
  - Creates and destroys WebContentsView instances
  - Handles view visibility and bounds
  - Manages view events (navigation, loading, etc.)
  - Implements popup window handling
  - Controls JavaScript enable/disable per view

- **`menu.js`**: Application menu system
  - Creates native menus for different platforms
  - Handles menu actions
  - Updates menu states based on application state

- **`filtering.js`**: Content filtering engine
  - Ad blocking using filter lists
  - Tracker blocking
  - HTTPS upgrade functionality
  - Request interception and blocking

- **`download.js`**: Download management
  - Handles download requests
  - Manages download UI
  - File system operations for downloads

- **`permissionManager.js`**: Permission handling
  - Manages site permissions (camera, microphone, location, etc.)
  - Stores permission preferences
  - Handles permission requests

- **`remoteActions.js`**: IPC handlers for renderer
  - Exposes safe methods to renderer process
  - Handles actions from browser UI

### 2. Renderer Process (js/*.js)

The renderer process runs the browser's user interface.

#### Core UI Components:

##### Tab Management (`js/tabState/`)
- **`tab.js`**: `TabList` class
  - Manages collection of tabs
  - Tab CRUD operations (add, update, delete)
  - Tab selection and ordering
  - Emits events for tab changes

- **`task.js`**: `TaskList` class
  - Manages tasks (tab groups)
  - Each task contains a `TabList`
  - Task creation, deletion, and switching
  - Tab history per task
  - Event system for task changes

##### Browser UI (`js/browserUI.js`)
- Coordinates major UI actions
- Implements `addTab()`, `destroyTab()`, `switchToTab()`
- Manages focus between UI components
- Orchestrates webviews, tab bar, and searchbar

##### Navigation Bar (`js/navbar/`)
- **`tabBar.js`**: Tab strip UI
  - Renders tab elements
  - Drag-and-drop reordering
  - Tab close buttons
  - Visual indicators (audio, loading, security)

- **`tabEditor.js`**: URL/search input for tabs
  - Edit mode for entering URLs
  - Integrated with searchbar

- **`navigationButtons.js`**: Back/forward buttons
- **`bookmarkStar.js`**: Bookmark toggle button
- **`contentBlockingToggle.js`**: Toggle content blocking per-site
- **`progressBar.js`**: Page loading progress indicator

##### Searchbar (`js/searchbar/`)
- **`searchbar.js`**: Main searchbar component
  - Shows/hides search interface
  - Manages search input
  - Coordinates with plugins

- **`searchbarPlugins.js`**: Plugin system
  - Registers and manages searchbar plugins
  - Routes queries to appropriate plugins
  - Displays plugin results

- **Plugin implementations**:
  - `placesPlugin.js`: History and bookmark search
  - `bangsPlugin.js`: DuckDuckGo bang shortcuts
  - `customBangs.js`: User-defined shortcuts
  - `instantAnswerPlugin.js`: DuckDuckGo instant answers
  - `calculatorPlugin.js`: In-search calculations
  - `searchSuggestionsPlugin.js`: Search engine suggestions
  - `bookmarkManager.js`: Bookmark management UI
  - `historyViewer.js`: History viewing UI

##### Places System (`js/places/`)
The Places system is Min's database for browsing history, bookmarks, and full-text search.

- **`places.js`**: Main Places API
  - Communicates with Places service via MessagePort
  - Promise-based API for async operations
  - Saves pages to history with metadata
  - Updates visit counts and timestamps

- **`placesService.js`**: Places background service
  - Runs in a hidden iframe (service worker alternative)
  - Uses Dexie (IndexedDB wrapper) for storage
  - Handles database operations
  - Implements full-text search indexing

- **`fullTextSearch.js`**: Full-text search engine
  - Indexes page content for searching
  - Stemming for better search results
  - Scoring algorithm for relevance

- **`tagIndex.js`**: Bookmark tagging system
  - Tag management and search
  - Tag autocomplete

##### Webviews (`js/webviews.js`)
- Manages WebContentsView instances from renderer side
- Handles view selection and visibility
- Captures tab screenshots
- Manages scroll position restoration
- Coordinates with viewManager in main process via IPC

##### Password Managers (`js/passwordManager/`)
- **`passwordManager.js`**: Main password manager interface
- **Integrations**:
  - `bitwarden.js`: Bitwarden integration
  - `onePassword.js`: 1Password integration
  - `keychain.js`: System keychain integration
- **`passwordCapture.js`**: Detects login forms and offers to save
- **`passwordViewer.js`**: UI for saved passwords

##### Task Overlay (`js/taskOverlay/`)
- **`taskOverlay.js`**: Task switching interface
  - Shows all tasks and their tabs
  - Keyboard navigation
  - Task search

- **`taskOverlayBuilder.js`**: Renders task overlay HTML
  - Creates visual representation of tasks
  - Shows tab previews
  - Drag-and-drop between tasks

##### Other Components:
- **`readerView.js`**: Simplified reading mode
- **`readerDecision.js`**: Detects reader-compatible pages
- **`sessionRestore.js`**: Saves/restores browser state
- **`downloadManager.js`**: Download UI in renderer
- **`findinpage.js`**: In-page search
- **`pdfViewer.js`**: PDF viewing integration
- **`pageTranslations.js`**: Page translation feature
- **`statistics.js`**: Anonymous usage statistics
- **`userscripts.js`**: Custom user script injection

### 3. Preload Scripts (`js/preload/`)

Preload scripts run in web content views with limited Node.js access:

- **`textExtractor.js`**: Extracts page text for indexing
- **`readerDetector.js`**: Detects if page is reader-compatible
- **`passwordFill.js`**: Fills password forms securely

### 4. Utilities (`js/util/`)

Shared utility modules:

- **`urlParser.js`**: URL parsing and validation
- **`searchEngine.js`**: Search engine configuration
- **`database.js`**: Database initialization
- **`keyMap.js`**: Keyboard shortcut mapping
- **`theme.js`**: Theme color management
- **`settings/`**: Settings persistence

## Data Flow Examples

### Opening a New Tab

```
1. User clicks "+" button
   → addTabButton.js

2. Calls addTab()
   → browserUI.js

3. Creates tab in data model
   → tabs.add() in task.js

4. Updates tab bar UI
   → tabBar.addTab() in tabBar.js

5. Creates WebContentsView
   → webviews.add() → IPC → viewManager.createView() in main

6. Shows tab editor for URL input
   → tabEditor.show() in tabEditor.js
```

### Loading a URL

```
1. User enters URL in searchbar
   → searchbar.js

2. URL parsed and validated
   → urlParser.js

3. Navigates webview
   → webviews.update() → IPC → viewManager.loadURL()

4. Page starts loading
   → viewManager emits 'did-start-navigation'
   → IPC → webviews.js updates tab state

5. Page finishes loading
   → viewManager emits 'did-finish-load'
   → IPC → captures screenshot, extracts text

6. Text extracted from page
   → preload/textExtractor.js → IPC

7. Saved to Places database
   → places.savePage() → placesService.js
```

### Searching History

```
1. User types in searchbar
   → searchbar.js

2. Query sent to plugins
   → searchbarPlugins.js

3. Places plugin searches database
   → placesPlugin.js → places.search()
   → IPC → placesService.js

4. Full-text search in IndexedDB
   → fullTextSearch.js uses Dexie

5. Results ranked and returned
   → placesService.js → IPC → placesPlugin.js

6. Results displayed in searchbar
   → searchbar.js renders results
```

### Content Filtering

```
1. WebContentsView makes network request
   → Chromium network stack

2. Request intercepted
   → filtering.js in main process

3. URL checked against filter lists
   → ABP filter parser (ext/abp-filter-parser-modified/)

4. If blocked: request cancelled
   → onBeforeRequest returns {cancel: true}

5. If allowed: request proceeds
   → Content loads normally
```

## Build System

### Build Pipeline

1. **Localization** (`scripts/buildLocalization.js`)
   - Compiles JSON language files
   - Creates localization bundle

2. **Main Process** (`scripts/buildMain.js`)
   - Concatenates main process files
   - No bundling (Node.js module system)

3. **Renderer Process** (`scripts/buildBrowser.js`)
   - Uses Browserify to bundle renderer code
   - Resolves CommonJS requires
   - Applies electron-renderify transform
   - Output: `dist/bundle.js`

4. **Styles** (`scripts/buildBrowserStyles.js`)
   - Concatenates CSS files
   - Output: `dist/bundle.css`

5. **Preload Scripts** (`scripts/buildPreload.js`)
   - Bundles preload scripts separately
   - Output: `dist/preload.js`

### Watch Mode

`scripts/watch.js` uses chokidar to watch for file changes and rebuild automatically.

### Platform Builds

- `buildWindows.js`: Windows installer
- `buildMac.js`: macOS .app bundle
- `buildDebian.js`: .deb package
- `buildRedhat.js`: .rpm package
- `buildAppImage.js`: AppImage for Linux

## Key Concepts

### Tasks

**Tasks** are Min's equivalent of tab groups or workspaces. Each task:
- Contains its own set of tabs
- Has a name (optional)
- Can be collapsed to hide tabs
- Maintains independent tab history
- Can be saved and restored across sessions

**Use cases:**
- Separate work and personal browsing
- Organize research topics
- Isolate different projects

### Places

**Places** is Min's integrated database for:
- Browsing history
- Bookmarks with tags
- Full-text search of visited pages

**Storage:**
- Uses IndexedDB via Dexie
- Runs in a background service (iframe)
- Indexes page content for search

**Features:**
- Full-text search across all visited pages
- Tag-based bookmark organization
- Visit statistics and frecency ranking

### Content Filtering

Min includes built-in content filtering:
- Ad blocking via filter lists (EasyList, etc.)
- Tracker blocking
- HTTPS upgrades
- Per-site enable/disable

**Implementation:**
- Uses modified ABP (Adblock Plus) parser
- Filtering happens in main process
- Minimal performance impact

### Reader View

Automatic reader mode for compatible articles:
- Detects article content
- Removes clutter (ads, navigation, etc.)
- Provides clean reading experience
- Uses Mozilla's Readability library

### Password Managers

Min integrates with external password managers:
- Bitwarden
- 1Password  
- System keychain (macOS/Linux)

**How it works:**
- Detects login forms
- Communicates with password manager CLI
- Fills credentials securely
- Optional password capture for saving

### Privacy Features

- **Private tabs**: Don't save history, isolated cookies
- **No tracking**: Blocks trackers by default
- **No telemetry**: Minimal opt-in usage statistics
- **Sandboxing**: Each tab runs in isolated context
- **Content security**: CSP policies enforced

## Security Architecture

### Process Isolation

- **Main process**: Trusted, has Node.js access
- **Renderer process**: Partially trusted, limited API
- **Web content**: Untrusted, fully sandboxed

### Context Isolation

- WebContentsViews run with `contextIsolation: true`
- Preload scripts bridge renderer and web content
- No direct access to Node.js from web pages

### Permissions

- Sites must request permissions (camera, mic, location)
- User approval required
- Permissions stored per-site
- Can be revoked anytime

## Extension Points

### Userscripts

Min supports userscripts for customization:
- Inject custom JavaScript into pages
- Access to limited Min APIs
- Can modify page content
- Similar to browser extensions but simpler

**Location:** `~/.config/Min/userscripts/`

### Searchbar Plugins

Developers can add searchbar plugins:
- Register with `searchbarPlugins.register()`
- Provide search results for queries
- Display custom UI
- See existing plugins for examples

### Custom Bangs

Users can create custom bang shortcuts:
- Quick access to favorite sites
- Format: `!shortcut search terms`
- Defined in searchbar settings

## File Structure Summary

```
min/
├── main/                 # Main process (Node.js)
│   ├── main.js          # Entry point
│   ├── viewManager.js   # WebView management
│   ├── menu.js          # Menu system
│   ├── filtering.js     # Content filtering
│   └── ...
├── js/                  # Renderer process (Browser UI)
│   ├── browserUI.js     # Core UI orchestration
│   ├── tabState/        # Tab and task management
│   ├── navbar/          # Navigation bar components
│   ├── searchbar/       # Search and plugins
│   ├── places/          # History/bookmark database
│   ├── passwordManager/ # Password manager integrations
│   ├── taskOverlay/     # Task switching UI
│   ├── preload/         # Preload scripts for web content
│   └── util/            # Shared utilities
├── css/                 # Stylesheets
├── pages/               # Special pages (settings, etc.)
├── localization/        # Translation files
├── icons/               # Icon resources
├── ext/                 # External libraries
│   ├── abp-filter-parser-modified/  # Ad blocker
│   └── readability-master/          # Reader view
├── scripts/             # Build scripts
├── docs/                # Documentation (this file!)
├── index.html           # Main UI HTML
└── package.json         # Node.js dependencies
```

## Threading Model

Min uses Electron's multi-process architecture:

1. **Main Process** (1 instance)
   - Runs continuously
   - Manages application lifecycle
   - Creates and manages windows/views

2. **Renderer Process** (1 per window)
   - Runs browser UI
   - Handles user interactions
   - Communicates with main via IPC

3. **Web Content Processes** (multiple)
   - One per site (Chromium's process-per-site)
   - Fully sandboxed
   - Isolated from each other
   - Managed by Chromium

## Performance Considerations

### Lazy Loading

- Tabs load content only when selected
- Background tabs remain unloaded
- Reduces memory usage for many tabs

### Screenshot Caching

- Tab screenshots captured for task overlay
- Low resolution (1/10th size)
- Captured on navigation, not continuously

### Database Optimization

- Full-text indexing happens asynchronously
- Search results cached
- IndexedDB provides fast queries

### View Management

- Views reused when possible
- Destroyed when not needed
- Popup views managed separately

## Common Patterns

### Event System

Many components use a custom event system:
```javascript
// task.js
tasks.on('tab-added', (tabId, tab) => {
  // Handle tab added
})

tasks.emit('tab-added', tabId, tab)
```

### IPC Communication

Main ↔ Renderer communication:
```javascript
// Renderer → Main
ipc.send('action-name', data)

// Main → Renderer
window.webContents.send('view-event', data)

// Renderer handles Main messages
ipc.on('view-event', (event, data) => {
  // Handle event
})
```

### Async View Operations

Interacting with web content:
```javascript
// Call method on webview
webviews.callAsync(tabId, 'methodName', [args], callback)

// Execute JavaScript in webview
webviews.callAsync(tabId, 'executeJavaScript', code, callback)
```

### Settings Management

Persistent settings:
```javascript
// Get setting
const value = settings.get('settingName')

// Set setting
settings.set('settingName', value)

// Listen for changes
settings.on('settingName', (value) => {
  // Handle change
})
```

## Further Reading

- [GETTING_STARTED.md](./GETTING_STARTED.md) - Setup and basic usage
- [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md) - How to modify Min
- [COMPONENT_REFERENCE.md](./COMPONENT_REFERENCE.md) - Detailed component docs
- [BUILD_SYSTEM.md](./BUILD_SYSTEM.md) - Build system details
