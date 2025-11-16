# Getting Started with Min Browser

This guide will help you get Min up and running, either as a user or as a developer looking to contribute or modify the browser.

## Table of Contents

- [For End Users](#for-end-users)
- [For Developers](#for-developers)
- [Understanding Min's Interface](#understanding-mins-interface)
- [Basic Usage](#basic-usage)
- [Advanced Features](#advanced-features)

## For End Users

### Installation

#### Download Pre-built Binaries

The easiest way to get Min is to download a pre-built binary for your platform:

1. Visit the [Min releases page](https://github.com/minbrowser/min/releases)
2. Download the appropriate installer for your OS:
   - **Windows**: `.exe` installer
   - **macOS**: `.dmg` disk image (Intel or Apple Silicon)
   - **Linux**: `.deb`, `.rpm`, or `.AppImage`

#### Linux Installation

**Debian/Ubuntu (.deb):**
```bash
sudo dpkg -i min_*_amd64.deb
```

**RedHat/Fedora (.rpm):**
```bash
sudo rpm -i min-*.rpm --ignoreos
```

**AppImage:**
```bash
chmod +x Min-*.AppImage
./Min-*.AppImage
```

**Arch Linux:**
Install from AUR:
```bash
yay -S min-browser-bin
# or
paru -S min-browser-bin
```

**Raspberry Pi:**
Min is available through [Pi-Apps](https://github.com/Botspot/pi-apps).

### First Launch

When you first launch Min:

1. **Welcome screen** introduces basic features
2. **Privacy settings** can be configured
3. **Password manager** integration can be set up (optional)
4. **Search engine** preference can be selected

## For Developers

### Prerequisites

Before you can build and run Min from source, you need:

- **Node.js** (v18 or later recommended)
  - Download from [nodejs.org](https://nodejs.org/)
  - Or use a version manager like [nvm](https://github.com/nvm-sh/nvm)
- **npm** (comes with Node.js)
- **Git** for cloning the repository

**Platform-specific requirements:**

- **Windows**: Visual Studio (for native modules)
- **macOS**: Xcode command-line tools
- **Linux**: Build essentials (`build-essential` on Debian/Ubuntu)

### Setup Development Environment

#### 1. Clone the Repository

```bash
git clone https://github.com/minbrowser/min.git
cd min
```

#### 2. Install Dependencies

```bash
npm install
```

This will:
- Install all npm packages
- Run the postinstall script (`scripts/setupDevEnv.js`)
- Set up the development environment

**Note:** The first install may take several minutes as it downloads Electron and compiles native modules.

#### 3. Build the Application

```bash
npm run build
```

This runs multiple build steps:
- `buildMain`: Bundles main process code
- `buildBrowser`: Bundles renderer process code
- `buildBrowserStyles`: Compiles CSS
- `buildPreload`: Bundles preload scripts

Build output goes to the `dist/` directory:
- `dist/main.build.js` - Main process bundle
- `dist/bundle.js` - Renderer process bundle
- `dist/bundle.css` - Compiled styles
- `dist/preload.js` - Preload script bundle

#### 4. Run Min in Development Mode

```bash
npm start
```

This will:
1. Build the application
2. Start the file watcher (automatically rebuilds on changes)
3. Launch Min with development flags

**Alternative (separate terminals):**
```bash
# Terminal 1: Watch for changes
npm run watch

# Terminal 2: Run Electron
npm run startElectron
```

### Development Mode Features

When running in development mode (`--development-mode` flag):

- **Separate user data**: Uses `userData-development` directory
- **Hot reload**: Press `Ctrl+Alt+R` (or `Cmd+Opt+R` on Mac) to reload the UI
- **DevTools**: Access browser DevTools with `Ctrl+Shift+I` (or `Cmd+Opt+I`)
- **Console output**: See logs in the terminal
- **No installer checks**: Skips Windows installer logic

### File Watching

The watch script (`npm run watch`) monitors files and automatically rebuilds:

- **JavaScript changes** in `js/` or `main/`: Rebuilds affected bundles
- **CSS changes**: Rebuilds styles
- **Localization changes**: Rebuilds localization bundle

**Note:** After rebuild, press `Ctrl+Alt+R` to see changes in the running app.

## Understanding Min's Interface

### Main Components

```
┌─────────────────────────────────────────────────────────────┐
│ [←] [→]  [Address/Search Bar]           [☆] [⋮] [+]     │ ← Navbar
├─────────────────────────────────────────────────────────────┤
│ Tab: GitHub | Tab: Documentation | Tab: Min Browser    [×] │ ← Tab Bar
├─────────────────────────────────────────────────────────────┤
│                                                               │
│                                                               │
│                      Web Content                             │ ← WebView
│                                                               │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

**1. Navigation Bar (Top)**
   - Back/Forward buttons
   - Address/Search bar (combined)
   - Bookmark star (toggle bookmark)
   - Menu button (⋮)
   - Add tab button (+)

**2. Tab Bar**
   - Shows tabs in current task
   - Draggable for reordering
   - Audio indicators for playing tabs
   - Security indicators (lock icon)
   - Close buttons (×)

**3. Web Content Area**
   - Displays the active tab's content
   - Full-screen available
   - Reader view available for articles

### Hidden Components

**Task Overlay** (press `Shift+Ctrl+E` or `Shift+Cmd+E`):
- Shows all tasks and their tabs
- Create new tasks
- Switch between tasks
- Drag tabs between tasks

**Searchbar** (click address bar or press `Ctrl+L`):
- Search history and bookmarks
- DuckDuckGo instant answers
- Calculator
- Search suggestions
- Bang shortcuts (!g, !w, etc.)

## Basic Usage

### Browsing

**Open a URL:**
1. Click the address bar (or press `Ctrl+L`)
2. Type a URL or search term
3. Press Enter

**Navigate:**
- Back: Click ← button or press `Alt+←`
- Forward: Click → button or press `Alt+→`
- Refresh: Press `Ctrl+R` or `F5`

**New Tab:**
- Click + button
- Press `Ctrl+T`
- Middle-click a link (opens in new tab)

**Close Tab:**
- Click × on tab
- Press `Ctrl+W`

### Searching

Min's search bar is powerful:

**Search History:**
Just type - results from history appear automatically.

**Bookmarks:**
Results show bookmarks with star icon (☆).

**Web Search:**
If no history match, searches using default search engine.

**Calculator:**
Type math expressions: `2 + 2`, `sqrt(144)`

**Bang Shortcuts:**
- `!g search term` - Google
- `!w search term` - Wikipedia
- `!yt search term` - YouTube
- `!gh search term` - GitHub

**Create Custom Bangs:**
1. Open settings (☰ menu → Preferences)
2. Go to "Custom Bangs"
3. Add your shortcuts

### Bookmarks

**Add Bookmark:**
1. Navigate to page
2. Click star icon (☆) in navbar
3. Optionally add tags
4. Click "Save"

**Tags:**
Organize bookmarks with tags:
- Add multiple tags: `work, reference, javascript`
- Search by tag: Start typing tag name
- Auto-complete suggests existing tags

**Search Bookmarks:**
- Type in searchbar
- Bookmarks appear with ☆ icon
- Filter by tag: `#work`

### Tasks (Tab Groups)

Tasks are Min's way of organizing tabs:

**View All Tasks:**
Press `Shift+Ctrl+E` (or `Shift+Cmd+E` on Mac)

**Create Task:**
1. Open task overlay
2. Click "Add Task"
3. Name it (optional)
4. Add tabs

**Switch Tasks:**
1. Open task overlay
2. Click on a task
3. Or use keyboard navigation (arrows + Enter)

**Move Tabs Between Tasks:**
1. Open task overlay
2. Drag tab to different task
3. Drop to move

**Use Cases:**
- Separate work and personal browsing
- Organize research topics
- Keep related tabs together

### Reader View

For article pages, Min offers a clean reading experience:

**Enable Reader View:**
- Min automatically detects articles
- Click reader icon in address bar
- Or press `Ctrl+Shift+R`

**Features:**
- Removes ads and clutter
- Adjustable text size
- High contrast for readability
- Extracts article content only

### Downloads

**Downloading Files:**
1. Click download link
2. Choose location (or use default)
3. Download appears in download bar
4. Click to open when complete

**Download Manager:**
- View all downloads: Menu → Downloads
- Show in folder
- Cancel/Resume downloads

### Private Browsing

**Private Tabs:**
Private tabs don't save history or cookies:

1. Menu → New Private Tab
2. Or press `Ctrl+Shift+P`
3. Private tabs have purple indicator
4. Close tab to delete data

**Note:** Downloads and bookmarks from private tabs are still saved.

## Advanced Features

### Password Managers

Min integrates with external password managers:

**Supported:**
- Bitwarden
- 1Password
- System Keychain (macOS/Linux)

**Setup:**
1. Menu → Preferences
2. Password Manager section
3. Select your password manager
4. Configure path to CLI tool
5. Enable integration

**Usage:**
- Min detects login forms
- Offers to fill from password manager
- Can save new credentials

### Content Blocking

**Built-in Ad Blocking:**
Min blocks ads and trackers by default using EasyList and EasyPrivacy.

**Toggle for Site:**
1. Click shield icon in navbar
2. Toggle "Block Ads and Trackers"
3. Page reloads with new setting

**Custom Filters:**
Not currently supported via UI, but filter lists can be updated:
```bash
npm run updateFilters
```

### Full-Text Search

Min indexes the text of pages you visit:

**Search Page Content:**
Type in searchbar - results include page text matches, not just titles.

**How It Works:**
- Text extracted from each page
- Indexed in local database
- Searchable immediately
- Privacy-friendly (all local)

**Disable:**
Menu → Preferences → Privacy → "Save browsing history"

### Page Translation

Min includes built-in page translation:

**Translate Page:**
1. Menu → Translate Page
2. Select target language
3. Page translates in-place

**Supported:**
Uses Bergamot translator (local, privacy-preserving).

### Userscripts

Extend Min with custom JavaScript:

**Location:** `~/.config/Min/userscripts/` (Linux/Mac) or `%APPDATA%\Min\userscripts\` (Windows)

**Create Script:**
1. Create `.js` file in userscripts folder
2. Add userscript metadata header
3. Write script code

**Example:**
```javascript
// ==UserScript==
// @name         Example Script
// @match        https://example.com/*
// @run-at       document-end
// ==/UserScript==

console.log('Hello from userscript!')
document.body.style.backgroundColor = 'lightblue'
```

**Available APIs:**
- Standard web APIs
- Limited Min APIs (see wiki)

**Documentation:**
See [Min userscripts documentation](https://github.com/minbrowser/min/wiki/userscripts) for details.

### Keyboard Shortcuts

**Navigation:**
- `Ctrl+L` - Focus address bar
- `Ctrl+T` - New tab
- `Ctrl+W` - Close tab
- `Ctrl+Tab` - Next tab
- `Ctrl+Shift+Tab` - Previous tab
- `Alt+←` - Back
- `Alt+→` - Forward
- `Ctrl+R` or `F5` - Reload

**Tasks:**
- `Shift+Ctrl+E` - Task overlay
- `Ctrl+N` - New task

**Search:**
- `Ctrl+F` - Find in page
- `Ctrl+K` - Focus search bar

**Reader:**
- `Ctrl+Shift+R` - Reader view

**Other:**
- `Ctrl+Shift+I` - Developer tools (development mode)
- `Ctrl+Alt+R` - Reload UI (development mode)
- `F11` - Fullscreen

**Mac:** Replace `Ctrl` with `Cmd` and `Alt` with `Opt`

**Customization:**
Keyboard shortcuts can be customized by editing `js/defaultKeybindings.js` (requires rebuilding).

### Settings

Access settings via Menu → Preferences:

**Privacy:**
- Save browsing history
- Send usage statistics
- Enable DuckDuckGo Privacy Grade

**Search Engine:**
- Choose default search engine
- Add custom search engines

**Appearance:**
- Dark theme
- Separate titlebar
- Compact mode

**Password Manager:**
- Choose password manager
- Configure integration

**Advanced:**
- Enable JavaScript (default: on)
- Enable plugins
- Hardware acceleration
- Startup behavior

## Troubleshooting

### Build Issues

**"npm install" fails:**
- Ensure Node.js version is compatible (v18+)
- On Windows: Install Visual Studio Build Tools
- On Mac: Install Xcode command-line tools
- On Linux: Install `build-essential`

**"npm run build" fails:**
- Delete `node_modules` and `dist` folders
- Run `npm install` again
- Check for error messages in output

### Runtime Issues

**Min won't start:**
- Check terminal for error messages
- Try deleting user data: `~/.config/Min` (Linux/Mac) or `%APPDATA%\Min` (Windows)
- Reinstall dependencies: `npm ci`

**White screen on launch:**
- Press `Ctrl+Shift+I` to open DevTools
- Check console for errors
- Rebuild: `npm run build`

**Changes not appearing:**
- Ensure watch script is running
- Press `Ctrl+Alt+R` to reload UI
- Check that correct file was modified
- Rebuild if necessary

### Getting Help

- **Wiki**: [github.com/minbrowser/min/wiki](https://github.com/minbrowser/min/wiki)
- **Issues**: [github.com/minbrowser/min/issues](https://github.com/minbrowser/min/issues)
- **Discord**: [discord.gg/bRpqjJ4](https://discord.gg/bRpqjJ4)
- **Discussions**: [github.com/minbrowser/min/discussions](https://github.com/minbrowser/min/discussions)

## Next Steps

- Read [ARCHITECTURE.md](./ARCHITECTURE.md) to understand Min's design
- Check [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md) to start modifying Min
- Explore [COMPONENT_REFERENCE.md](./COMPONENT_REFERENCE.md) for detailed API docs
- Review existing issues to find areas to contribute

## Additional Resources

- **Main Repository**: https://github.com/minbrowser/min
- **Website**: https://minbrowser.org/
- **Feature Tour**: http://minbrowser.org/tour/
- **FAQ**: https://github.com/minbrowser/min/wiki/FAQ
