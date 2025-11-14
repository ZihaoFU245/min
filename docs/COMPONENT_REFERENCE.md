# Component Reference Guide

This document provides detailed information about Min's major components, their APIs, and how they interact with each other.

## Table of Contents

- [Task and Tab Management](#task-and-tab-management)
- [Places System](#places-system)
- [Webview Management](#webview-management)
- [Searchbar System](#searchbar-system)
- [Content Filtering](#content-filtering)
- [Password Manager Integration](#password-manager-integration)
- [Settings System](#settings-system)
- [IPC Communication](#ipc-communication)

## Task and Tab Management

### TaskList (`js/tabState/task.js`)

The `TaskList` class manages all tasks (tab groups) in the browser.

#### Properties

```javascript
tasks.tasks  // Array of task objects
```

Each task object:
```javascript
{
  id: String,              // Unique task ID
  name: String,            // Task name (optional)
  tabs: TabList,           // TabList instance
  tabHistory: TabStack,    // Closed tabs for undo
  collapsed: Boolean,      // UI collapsed state
  selectedInWindow: Number // Window ID where selected
}
```

#### Methods

**`tasks.add(task, index, emit)`**
- Creates a new task
- `task`: Optional task object with properties
- `index`: Optional position in task list
- `emit`: Whether to emit events (default: true)
- Returns: Task ID

```javascript
var taskId = tasks.add({
  name: 'Research'
})
```

**`tasks.get(id)`**
- Retrieves task by ID
- Returns: Task object or undefined

```javascript
var task = tasks.get(taskId)
console.log(task.name)  // 'Research'
```

**`tasks.destroy(id)`**
- Removes a task and all its tabs
- Emits 'task-destroyed' event

```javascript
tasks.destroy(taskId)
```

**`tasks.getSelected()`**
- Returns currently selected task
- Returns: Task object

**`tasks.setSelected(id)`**
- Sets the current task
- Updates UI to show task's tabs
- Emits 'task-selected' event

```javascript
tasks.setSelected(taskId)
```

**`tasks.getIndex(id)`**
- Gets position of task in list
- Returns: Index (number) or -1

**`tasks.getTaskContainingTab(tabId)`**
- Finds which task contains a tab
- Returns: Task object

```javascript
var task = tasks.getTaskContainingTab(tabId)
```

#### Events

Listen to events:
```javascript
tasks.on('event-name', function (data) {
  // Handle event
})
```

Available events:
- `task-added` - New task created
- `task-destroyed` - Task deleted
- `task-selected` - Task switched
- `task-updated` - Task properties changed
- `tab-added` - Tab created in any task
- `tab-updated` - Tab properties changed
- `tab-destroyed` - Tab closed
- `*` - All events (includes event name as first arg)

Example:
```javascript
tasks.on('tab-added', function (tabId, tab, options, taskId) {
  console.log('Tab added:', tabId, 'in task:', taskId)
})
```

### TabList (`js/tabState/tab.js`)

Manages tabs within a task.

#### Properties

```javascript
task.tabs.tabs  // Array of tab objects
```

Each tab object:
```javascript
{
  id: String,              // Unique tab ID
  url: String,             // Current URL
  title: String,           // Page title
  lastActivity: Number,    // Timestamp
  secure: Boolean,         // HTTPS?
  private: Boolean,        // Private tab?
  readerable: Boolean,     // Can use reader view?
  themeColor: String,      // Site theme color
  backgroundColor: String, // Background color
  scrollPosition: Number,  // Scroll offset
  selected: Boolean,       // Currently selected?
  muted: Boolean,          // Audio muted?
  loaded: Boolean,         // Content loaded?
  hasAudio: Boolean,       // Playing audio?
  previewImage: String     // Screenshot data URL
}
```

#### Methods

**`tabs.add(tab, options, emit)`**
- Creates a new tab
- `tab`: Optional tab properties
- `options`: { atEnd: Boolean } - position
- Returns: Tab ID

```javascript
var tabId = task.tabs.add({
  url: 'https://example.com',
  title: 'Example'
})
```

**`tabs.get(id)`**
- Retrieves tab by ID
- Returns: Tab object or undefined

**`tabs.update(id, data, emit)`**
- Updates tab properties
- `data`: Object with properties to update
- Emits 'tab-updated' event

```javascript
tabs.update(tabId, {
  title: 'New Title',
  url: 'https://newurl.com'
})
```

**`tabs.destroy(id)`**
- Removes tab
- Adds to tab history for undo
- Emits 'tab-destroyed' event

**`tabs.count()`**
- Returns number of tabs
- Returns: Number

**`tabs.getSelected()`**
- Gets currently selected tab ID
- Returns: Tab ID string

**`tabs.setSelected(id)`**
- Sets selected tab
- Updates previous tab's selected property
- Emits 'tab-selected' event

**`tabs.getSelectedIndex()`**
- Gets index of selected tab
- Returns: Number

**`tabs.getAtIndex(index)`**
- Gets tab at position
- Returns: Tab object

**`tabs.getIndex(id)`**
- Gets position of tab
- Returns: Number or -1

**`tabs.reorder(newOrder)`**
- Reorders tabs
- `newOrder`: Array of tab IDs in new order

```javascript
tabs.reorder([tab3, tab1, tab2])
```

**`tabs.moveBy(id, offset)`**
- Moves tab by offset positions
- `offset`: Positive (right) or negative (left)

```javascript
tabs.moveBy(tabId, -1)  // Move left
tabs.moveBy(tabId, 2)   // Move right 2
```

## Places System

### Places API (`js/places/places.js`)

Manages browsing history, bookmarks, and full-text search.

#### Methods

**`places.savePage(tabId, extractedText)`**
- Saves current page to history
- Called automatically on page load
- `extractedText`: Page content for search indexing

```javascript
places.savePage(tabId, pageText)
```

**`places.updatePlace(url, data, flags)`**
- Updates or creates a place entry
- Returns: Promise

```javascript
await places.invokeWithPromise({
  action: 'updatePlace',
  pageData: {
    url: 'https://example.com',
    title: 'Example Site',
    color: '#ffffff',
    extractedText: 'page content...'
  },
  flags: {
    isNewVisit: true
  }
})
```

**`places.searchPlaces(searchText, options)`**
- Searches history and bookmarks
- Returns: Promise<Array>

```javascript
var results = await places.invokeWithPromise({
  action: 'searchPlaces',
  searchText: 'javascript tutorial',
  options: {
    searchFTS: true  // Include full-text search
  }
})
```

**`places.deleteHistory(url)`**
- Removes URL from history
- Returns: Promise

```javascript
await places.invokeWithPromise({
  action: 'deleteHistory',
  url: 'https://example.com'
})
```

**`places.getSuggestedTags()`**
- Gets list of existing tags
- For autocomplete
- Returns: Promise<Array>

```javascript
var tags = await places.invokeWithPromise({
  action: 'getSuggestedTags'
})
```

**`places.searchPlacesFullText(searchText, options)`**
- Full-text search in page content
- More detailed than searchPlaces
- Returns: Promise<Array>

### PlacesService (`js/places/placesService.js`)

Background service that handles database operations.

#### Database Schema

Uses Dexie (IndexedDB) with these stores:

**places:**
```javascript
{
  url: String (primary key),
  title: String,
  color: String,
  visitCount: Number,
  lastVisit: Number,
  boost: Number,
  isBookmarked: Boolean,
  tags: Array<String>,
  metadata: Object,
  pageHTML: String,
  extractedText: String,
  searchIndex: String,
  searchIndex_stemmed: String
}
```

**readingList:**
```javascript
{
  url: String (primary key),
  title: String,
  visitCount: Number,
  time: Number
}
```

**statistics:**
```javascript
{
  path: String (primary key),
  data: Object
}
```

#### Internal Methods

These run in the service worker context:

**`updatePlace(pageData, flags)`**
- Updates or inserts place
- Updates visit count
- Indexes text for search

**`searchPlaces(searchText, options)`**
- Queries database
- Ranks results by relevance
- Supports full-text search

**`searchPlacesFullText(searchText, options)`**
- Full-text only search
- Uses stemmed index
- Returns snippets

### Full-Text Search (`js/places/fullTextSearch.js`)

#### Methods

**`fullTextSearch.index(url, title, extractedText)`**
- Indexes page content
- Uses stemming for better matches
- Returns: Search index string

```javascript
var searchIndex = fullTextSearch.index(
  url,
  'Page Title',
  'Page content text here...'
)
```

**`fullTextSearch.search(searchIndex, searchTerm, threshold)`**
- Searches indexed content
- `threshold`: Minimum score (default: 0.25)
- Returns: Score (0-1) or null

```javascript
var score = fullTextSearch.search(
  searchIndex,
  'javascript',
  0.25
)
```

## Webview Management

### Webviews (`js/webviews.js`)

Manages WebContentsView instances from renderer side.

#### Properties

```javascript
webviews.selectedId  // Currently visible tab ID
```

#### Methods

**`webviews.add(tabId, existingViewId)`**
- Creates view for tab
- Sends IPC to main process
- `existingViewId`: Reuse popup view (optional)

```javascript
webviews.add(tabId)
```

**`webviews.setSelected(tabId, options)`**
- Makes tab's view visible
- Hides other views
- `options`: { focus: Boolean }

```javascript
webviews.setSelected(tabId, { focus: true })
```

**`webviews.update(tabId, url)`**
- Navigates view to URL
- Updates tab state

```javascript
webviews.update(tabId, 'https://example.com')
```

**`webviews.destroy(tabId)`**
- Removes view
- Sends IPC to main process

```javascript
webviews.destroy(tabId)
```

**`webviews.callAsync(tabId, method, args, callback)`**
- Calls method on view's webContents
- Via IPC to main process
- `callback`: Receives (error, result)

```javascript
webviews.callAsync(tabId, 'executeJavaScript', 
  'document.title',
  function (err, title) {
    console.log('Title:', title)
  }
)
```

**`webviews.hide(tabId)`**
- Hides view without destroying
- View remains loaded

**`webviews.show(tabId)`**
- Shows previously hidden view

**`webviews.focus(tabId)`**
- Gives focus to view

### ViewManager (`main/viewManager.js`)

Main process view management.

#### Methods (IPC handlers)

**`createView(options)`**
- Creates WebContentsView
- `options`: { id, webPreferences, boundsString, events }
- Sets up event forwarding

**`destroyView(id)`**
- Destroys view
- Cleans up resources

**`setViewBounds(id, bounds)`**
- Sets view position/size
- `bounds`: { x, y, width, height }

**`loadURL(options)`**
- Loads URL in view
- `options`: { id, url, options }

**`goBack(id)` / `goForward(id)`**
- Navigation

**`reload(id)`**
- Reloads page

**`stop(id)`**
- Stops loading

**`callViewMethod(options)`**
- Calls method on webContents
- Generic handler for any method

## Searchbar System

### Searchbar (`js/searchbar/searchbar.js`)

Main search interface.

#### Methods

**`searchbar.show()`**
- Opens searchbar
- Focuses input
- Shows plugins

```javascript
searchbar.show()
```

**`searchbar.hide()`**
- Closes searchbar
- Clears input

**`searchbar.getValue()`**
- Gets current input text
- Returns: String

**`searchbar.setValue(text)`**
- Sets input text
- Triggers plugins

```javascript
searchbar.setValue('search query')
```

**`searchbar.openURL(url, options)`**
- Opens URL from search
- `options`: { background: Boolean }

```javascript
searchbar.openURL('https://example.com', {
  background: false
})
```

### Searchbar Plugins (`js/searchbar/searchbarPlugins.js`)

Plugin system for search results.

#### Registering a Plugin

```javascript
var myPlugin = {
  name: 'uniqueName',
  
  // Called once on startup
  initialize: function () {
    // Setup code
  },
  
  // Called for each search query
  showResults: function (text, input, event, suggestions) {
    // text: search query string
    // input: searchbar input element
    // event: triggering event
    // suggestions: container element
    
    // Generate results
    var results = [
      {
        title: 'Result Title',
        descriptionBlock: 'Description text',
        url: 'https://example.com',
        icon: 'icon-star',
        click: function () {
          // Custom click handler (optional)
        }
      }
    ]
    
    // Add to searchbar
    searchbarPlugins.addResults(
      this.name,
      results,
      input,
      event,
      suggestions
    )
  }
}

searchbarPlugins.register(myPlugin)
```

#### Plugin Methods

**`searchbarPlugins.register(plugin)`**
- Registers plugin
- Calls initialize()

**`searchbarPlugins.addResults(pluginName, results, input, event, container)`**
- Adds result items to searchbar
- Handles rendering and events

**`searchbarPlugins.run(text, input, event)`**
- Runs all plugins for query
- Called automatically on input

#### Result Object Format

```javascript
{
  title: String,              // Main text
  secondaryText: String,      // Optional subtitle
  descriptionBlock: String,   // Detailed description
  attribution: String,        // Source attribution
  url: String,                // Target URL
  icon: String,               // CSS class for icon
  image: String,              // Image URL
  click: Function,            // Custom handler
  delete: Function,           // Delete handler
  classList: Array<String>    // Additional CSS classes
}
```

### Built-in Plugins

**placesPlugin:**
- Searches history and bookmarks
- Shows visit count
- Displays last visit time

**bangsPlugin:**
- DuckDuckGo bang shortcuts
- `!g` for Google, `!w` for Wikipedia, etc.

**customBangs:**
- User-defined shortcuts
- Managed in settings

**instantAnswerPlugin:**
- DuckDuckGo instant answers
- Weather, definitions, calculations

**calculatorPlugin:**
- Basic math expressions
- `2 + 2`, `sqrt(16)`, etc.

**searchSuggestionsPlugin:**
- Search engine suggestions
- Uses configured search engine

## Content Filtering

### Filtering (`main/filtering.js`)

Content blocking in main process.

#### Methods

**`registerFiltering(session)`**
- Sets up request filtering
- Called during app initialization
- Uses ABP filter parser

```javascript
// In main.js startup:
filtering.registerFiltering(session.defaultSession)
```

#### Filter Lists

Defined in module:
```javascript
var FilterLists = [
  {
    url: 'https://easylist.to/easylist/easylist.txt',
    type: 'ads'
  },
  {
    url: 'https://easylist.to/easylist/easyprivacy.txt',
    type: 'trackers'
  }
]
```

#### Blocking Logic

```javascript
session.webRequest.onBeforeRequest((details, callback) => {
  // Check if URL matches filter
  if (shouldBlock(details.url)) {
    callback({ cancel: true })
  } else {
    callback({})
  }
})
```

#### HTTPS Upgrade

Automatically upgrades HTTP to HTTPS for known sites:

```javascript
// Uses HTTPS Everywhere rulesets
if (canUpgradeToHTTPS(url)) {
  callback({ redirectURL: httpsURL })
}
```

#### Per-Site Toggle

Users can disable filtering per site:

```javascript
// Stored in settings
var disabledSites = settings.get('filtering.disabledSites') || []

if (disabledSites.includes(hostname)) {
  // Don't filter this site
}
```

## Password Manager Integration

### Password Manager (`js/passwordManager/passwordManager.js`)

Base interface for password managers.

#### Methods

**`passwordManager.getCredentials(domain)`**
- Retrieves saved credentials
- Returns: Promise<Array>

```javascript
var creds = await passwordManager.getCredentials('example.com')
// [{ username: 'user', password: 'pass' }]
```

**`passwordManager.saveCredential(domain, username, password)`**
- Saves new credential
- Returns: Promise

**`passwordManager.getAllCredentials()`**
- Lists all saved credentials
- Returns: Promise<Array>

### Bitwarden (`js/passwordManager/bitwarden.js`)

Bitwarden CLI integration.

#### Setup

Requires Bitwarden CLI installed and configured:
```bash
bw login
bw unlock
```

#### Methods

Implements password manager interface:
- `getCredentials(domain)`
- `saveCredential(domain, username, password)`
- `getAllCredentials()`

Uses CLI commands:
```javascript
execSync('bw list items --search example.com')
```

### 1Password (`js/passwordManager/onePassword.js`)

1Password CLI integration.

Similar to Bitwarden, uses `op` CLI.

### Keychain (`js/passwordManager/keychain.js`)

System keychain integration (macOS/Linux).

Uses Node.js keytar module.

### Password Capture (`js/passwordManager/passwordCapture.js`)

Detects and captures login forms.

#### How It Works

1. Preload script detects form submission
2. Sends credentials to renderer
3. Prompts user to save
4. Saves to configured password manager

## Settings System

### Settings (`js/util/settings/settings.js`)

Persistent settings storage.

#### Methods

**`settings.get(key)`**
- Retrieves setting value
- Returns: Value or undefined

```javascript
var darkMode = settings.get('darkMode')
```

**`settings.set(key, value)`**
- Sets setting value
- Persists to disk
- Emits change event

```javascript
settings.set('darkMode', true)
```

**`settings.on(key, callback)`**
- Listens for setting changes
- `callback`: Receives new value

```javascript
settings.on('darkMode', function (enabled) {
  updateTheme(enabled)
})
```

#### Default Settings

Defined in module:
```javascript
var defaults = {
  searchEngine: 'duckduckgo',
  darkMode: false,
  useSeparateTitlebar: false,
  saveHistory: true,
  enableAutoplay: false,
  // ... more defaults
}
```

#### Storage

Settings saved to:
- **Linux/Mac**: `~/.config/Min/userSettings.json`
- **Windows**: `%APPDATA%\Min\userSettings.json`

Format:
```json
{
  "darkMode": true,
  "searchEngine": "google",
  "customBangs": [...]
}
```

## IPC Communication

### Main → Renderer

Send from main process:
```javascript
// Single window
window.webContents.send('event-name', data)

// All windows
BrowserWindow.getAllWindows().forEach(window => {
  window.webContents.send('event-name', data)
})
```

Receive in renderer:
```javascript
ipc.on('event-name', function (event, data) {
  console.log('Received:', data)
})
```

### Renderer → Main

Send from renderer:
```javascript
ipc.send('action-name', data)
```

Receive in main:
```javascript
ipc.on('action-name', function (event, data) {
  console.log('Received:', data)
  
  // Reply to sender
  event.sender.send('reply-event', result)
})
```

### View Events

Views emit events to renderer:

```javascript
// In viewManager.js (main)
view.webContents.on('did-finish-load', function () {
  window.webContents.send('view-event', {
    tabId: id,
    event: 'did-finish-load',
    args: []
  })
})

// In webviews.js (renderer)
ipc.on('view-event', function (event, data) {
  if (data.event === 'did-finish-load') {
    onPageLoad(data.tabId)
  }
})
```

### Remote Actions

Renderer can call main process methods:

```javascript
// Renderer
ipc.send('callRemoteAction', {
  action: 'showSaveDialog',
  args: [options]
})

// Main (remoteActions.js)
ipc.on('callRemoteAction', function (event, data) {
  var result = actions[data.action](...data.args)
  event.sender.send('remoteActionResult', result)
})
```

## Utility Functions

### URL Parser (`js/util/urlParser.js`)

**`urlParser.parse(url)`**
- Parses and validates URL
- Returns: Normalized URL

**`urlParser.isURL(text)`**
- Checks if text is valid URL
- Returns: Boolean

**`urlParser.getSourceURL(url)`**
- Extracts original URL from internal URLs
- Example: `min://app/reader/index.html?url=https://example.com` → `https://example.com`

### Search Engine (`js/util/searchEngine.js`)

**`searchEngine.getSearchURL(searchEngine, query)`**
- Generates search URL
- Returns: String

```javascript
var url = searchEngine.getSearchURL('google', 'cats')
// https://www.google.com/search?q=cats
```

**`searchEngine.getCurrent()`**
- Gets configured search engine
- Returns: Engine object

### Database (`js/util/database.js`)

**`db.get(storeName)`**
- Returns Dexie table
- For direct database access

```javascript
var placesTable = db.get('places')
var results = await placesTable
  .where('url')
  .startsWith('https://github')
  .toArray()
```

## Component Interaction Patterns

### Opening a URL Flow

```
User enters URL
  ↓
searchbar.openURL()
  ↓
browserUI.navigate()
  ↓
webviews.update(tabId, url)
  ↓
IPC → viewManager.loadURL()
  ↓
WebContentsView loads page
  ↓
did-start-navigation event
  ↓
IPC → webviews.js
  ↓
tabs.update(tabId, { loading: true })
  ↓
UI updates (progress bar, etc.)
  ↓
did-finish-load event
  ↓
Extract text, save to Places
  ↓
tabs.update(tabId, { loading: false })
```

### Searching History Flow

```
User types in searchbar
  ↓
searchbar input event
  ↓
searchbarPlugins.run(text)
  ↓
placesPlugin.showResults()
  ↓
places.searchPlaces(text)
  ↓
IPC → placesService
  ↓
Database query (IndexedDB)
  ↓
Full-text search if needed
  ↓
Results ranked by relevance
  ↓
IPC → placesPlugin
  ↓
searchbarPlugins.addResults()
  ↓
Results displayed in searchbar
```

### Content Filtering Flow

```
WebContentsView makes request
  ↓
onBeforeRequest handler
  ↓
Check URL against filters
  ↓
ABP parser matches rules
  ↓
If blocked: callback({ cancel: true })
  ↓
Request cancelled
  ↓
If allowed: callback({})
  ↓
Request proceeds
```

## Best Practices

### Performance

1. **Batch updates**: Update UI once, not per item
2. **Debounce**: Delay expensive operations
3. **Lazy load**: Load views only when needed
4. **Cache**: Store computed values
5. **Measure**: Profile before optimizing

### Security

1. **Sanitize**: Never trust user input
2. **Validate URLs**: Check before navigating
3. **Isolate**: Keep web content sandboxed
4. **Limit**: Minimize preload script access
5. **Audit**: Review security implications

### Maintainability

1. **Small functions**: One responsibility
2. **Clear names**: Self-documenting code
3. **Comments**: Explain why, not what
4. **Consistent**: Follow existing patterns
5. **Test**: Verify changes work

## Further Reading

- [ARCHITECTURE.md](./ARCHITECTURE.md) - System design
- [GETTING_STARTED.md](./GETTING_STARTED.md) - Setup guide
- [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md) - How to modify Min
- [Min Wiki](https://github.com/minbrowser/min/wiki) - More documentation
