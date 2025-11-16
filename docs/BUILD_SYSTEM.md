# Build System Documentation

This document explains Min's build system in detail, including how the build pipeline works, what each build script does, and how to create platform-specific builds.

## Table of Contents

- [Overview](#overview)
- [Build Pipeline](#build-pipeline)
- [Build Scripts](#build-scripts)
- [Watch Mode](#watch-mode)
- [Platform Builds](#platform-builds)
- [Build Optimization](#build-optimization)
- [Troubleshooting](#troubleshooting)

## Overview

Min uses a custom build system based on Node.js scripts. The build process:

1. **Concatenates** files that don't need bundling
2. **Bundles** code with Browserify for the renderer process
3. **Compiles** CSS files
4. **Processes** localization files
5. **Packages** the application for different platforms

### Build Directory Structure

```
min/
├── dist/                    # Build output
│   ├── main.build.js       # Main process bundle
│   ├── bundle.js           # Renderer process bundle
│   ├── bundle.css          # Compiled styles
│   ├── preload.js          # Preload script bundle
│   └── localization.build.js  # Translations
├── scripts/                 # Build scripts
│   ├── buildMain.js
│   ├── buildBrowser.js
│   ├── buildBrowserStyles.js
│   ├── buildPreload.js
│   ├── buildLocalization.js
│   ├── watch.js
│   └── build*.js (platform builds)
└── package.json            # npm scripts
```

## Build Pipeline

### Full Build Process

When you run `npm run build`:

```
1. buildMain
   - Concatenates main/*.js files
   - Output: dist/main.build.js

2. buildBrowser
   - Builds localization first
   - Concatenates legacy modules
   - Bundles with Browserify
   - Output: dist/bundle.js

3. buildBrowserStyles
   - Concatenates CSS files
   - Output: dist/bundle.css

4. buildPreload
   - Bundles preload scripts
   - Output: dist/preload.js
```

### Individual Build Commands

```bash
npm run buildMain          # Main process only
npm run buildBrowser       # Renderer process only
npm run buildBrowserStyles # Styles only
npm run buildPreload       # Preload scripts only
npm run build              # All of the above
```

## Build Scripts

### buildMain.js

**Purpose:** Bundles the main process code.

**Process:**
1. Reads all files in `main/` directory
2. Concatenates them in order
3. No module resolution (uses Node.js require)
4. Outputs to `dist/main.build.js`

**Code:**
```javascript
const fs = require('fs')
const path = require('path')

// List of main process files
const fileList = [
  'main/main.js',
  'main/viewManager.js',
  'main/menu.js',
  // ... etc
]

// Concatenate files
let output = ''
fileList.forEach(file => {
  output += fs.readFileSync(file, 'utf-8') + '\n'
})

// Write bundle
fs.writeFileSync('dist/main.build.js', output)
```

**Entry point in package.json:**
```json
{
  "main": "main.build.js"
}
```

### buildBrowser.js

**Purpose:** Bundles the renderer process code.

**Process:**
1. Builds localization first
2. Concatenates legacy modules (non-CommonJS)
3. Creates intermediate file
4. Bundles with Browserify
5. Applies electron-renderify transform
6. Outputs to `dist/bundle.js`

**Key features:**
- **Browserify**: Resolves CommonJS requires
- **electron-renderify**: Handles Electron renderer APIs
- **Excludes**: Heavy Node modules (chokidar, etc.)
- **Paths**: Resolves from root and js/ directories

**Code structure:**
```javascript
const browserify = require('browserify')
const renderify = require('electron-renderify')

// Legacy modules (concatenated)
const fileList = [
  'dist/localization.build.js',
  'js/default.js'  // Entry point
]

// Create intermediate file
let output = ''
fileList.forEach(script => {
  output += fs.readFileSync(script) + ';\n'
})
fs.writeFileSync('dist/build.js', output)

// Browserify bundle
const instance = browserify('dist/build.js', {
  paths: [rootDir, jsDir],
  node: true,
  detectGlobals: false
})

instance.transform(renderify)
instance.bundle()
  .pipe(fs.createWriteStream('dist/bundle.js'))
```

**Entry point:** `js/default.js`
- Requires all UI modules
- Sets up global objects
- Initializes UI components

### buildBrowserStyles.js

**Purpose:** Compiles CSS files.

**Process:**
1. Reads all CSS files from `css/` directory
2. Concatenates in order
3. No processing (no SASS/LESS)
4. Outputs to `dist/bundle.css`

**CSS files:**
```
css/
├── base.css          # Reset and base styles
├── theme.css         # Theming variables
├── navbar.css        # Navigation bar
├── tabBar.css        # Tab strip
├── searchbar.css     # Search interface
├── taskOverlay.css   # Task switcher
├── webviews.css      # Webview container
└── ...
```

**Code:**
```javascript
const fileList = [
  'css/base.css',
  'css/theme.css',
  'css/navbar.css',
  // ... etc
]

let output = ''
fileList.forEach(file => {
  output += fs.readFileSync(file, 'utf-8') + '\n'
})

fs.writeFileSync('dist/bundle.css', output)
```

### buildPreload.js

**Purpose:** Bundles preload scripts for web content.

**Process:**
1. Reads preload scripts from `js/preload/`
2. Bundles with Browserify
3. Outputs to `dist/preload.js`

**Preload scripts:**
- `textExtractor.js` - Extract page text
- `readerDetector.js` - Detect reader mode
- `passwordFill.js` - Password autofill

**Security note:** Preload scripts run with limited Node.js access in web content context.

### buildLocalization.js

**Purpose:** Compiles translation files.

**Process:**
1. Reads JSON files from `localization/languages/`
2. Detects user's language
3. Bundles translations into single file
4. Outputs to `dist/localization.build.js`

**Language files:**
```
localization/languages/
├── en-US.json
├── es.json
├── fr.json
├── de.json
├── zh-CN.json
└── ...
```

**Generated code:**
```javascript
// dist/localization.build.js
var languages = {
  'en-US': { 'ok': 'OK', ... },
  'es': { 'ok': 'Aceptar', ... },
  // ...
}

var userLanguage = detectLanguage()
var l = function(key) {
  return languages[userLanguage][key] || key
}
```

**Usage in code:**
```javascript
alert(l('confirmDeleteBookmark'))
```

## Watch Mode

### watch.js

**Purpose:** Auto-rebuild on file changes during development.

**How it works:**
1. Uses `chokidar` to watch file system
2. Detects changes to source files
3. Runs appropriate build script
4. Logs rebuild to console

**Code:**
```javascript
const chokidar = require('chokidar')

// Watch main process files
chokidar.watch('main/**/*.js').on('change', () => {
  console.log('Rebuilding main process...')
  require('./buildMain.js')()
})

// Watch renderer process files
chokidar.watch('js/**/*.js').on('change', () => {
  console.log('Rebuilding renderer process...')
  require('./buildBrowser.js')()
})

// Watch CSS files
chokidar.watch('css/**/*.css').on('change', () => {
  console.log('Rebuilding styles...')
  require('./buildBrowserStyles.js')()
})

// Watch localization files
chokidar.watch('localization/**/*.json').on('change', () => {
  console.log('Rebuilding localization...')
  require('./buildLocalization.js')()
})
```

**Usage:**
```bash
npm run watch
```

**Tips:**
- Runs in background
- Very fast rebuilds (< 1 second)
- Press `Ctrl+Alt+R` in Min to see changes
- Watch for console output to confirm rebuild

## Platform Builds

### Overview

Min can be packaged for multiple platforms:

- **Windows**: `.exe` installer
- **macOS**: `.app` bundle (Intel and ARM)
- **Linux**: `.deb`, `.rpm`, `.AppImage`

### buildWindows.js

**Purpose:** Create Windows installer.

**Process:**
1. Runs full build
2. Uses electron-builder
3. Creates NSIS installer
4. Outputs to `dist/`

**Requirements:**
- Windows OS
- Visual Studio (for native modules)

**Configuration:**
Uses electron-builder config in `package.json`:
```json
{
  "build": {
    "appId": "com.minbrowser.min",
    "productName": "Min",
    "win": {
      "target": "nsis",
      "icon": "icons/icon.ico"
    }
  }
}
```

**Run:**
```bash
npm run buildWindows
```

**Output:**
```
dist/
└── Min Setup 1.35.2.exe
```

### buildMac.js

**Purpose:** Create macOS application bundle.

**Process:**
1. Runs full build
2. Uses electron-builder
3. Creates .app bundle
4. Signs with code signature (if configured)
5. Creates .dmg installer

**Platforms:**
- Intel: `--platform=x86`
- Apple Silicon: `--platform=arm64`

**Requirements:**
- macOS
- Xcode command-line tools
- Apple Developer certificate (for signing)

**Configuration:**
```json
{
  "build": {
    "mac": {
      "category": "public.app-category.productivity",
      "icon": "icons/icon.icns",
      "target": ["dmg", "zip"]
    }
  }
}
```

**Run:**
```bash
npm run buildMacIntel   # Intel Macs
npm run buildMacArm     # Apple Silicon
```

**Output:**
```
dist/
├── Min-1.35.2.dmg
├── Min-1.35.2-mac.zip
└── mac/
    └── Min.app
```

**SDK version:**
May need to set SDK:
```bash
export SDKROOT=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX11.1.sdk
```

### buildDebian.js

**Purpose:** Create Debian/Ubuntu .deb package.

**Process:**
1. Runs full build
2. Uses electron-builder
3. Creates .deb package
4. Outputs to `dist/`

**Requirements:**
- Linux
- dpkg tools

**Configuration:**
```json
{
  "build": {
    "linux": {
      "target": "deb",
      "category": "Network",
      "icon": "icons/icon.png"
    }
  }
}
```

**Run:**
```bash
npm run buildDebian
```

**Output:**
```
dist/
└── min_1.35.2_amd64.deb
```

**Install:**
```bash
sudo dpkg -i min_1.35.2_amd64.deb
```

### buildRedhat.js

**Purpose:** Create RedHat/Fedora .rpm package.

**Process:**
1. Runs full build
2. Uses electron-builder
3. Creates .rpm package

**Requirements:**
- Linux
- rpm-build tools

**Run:**
```bash
npm run buildRedhat
```

**Output:**
```
dist/
└── min-1.35.2.x86_64.rpm
```

**Install:**
```bash
sudo rpm -i min-1.35.2.x86_64.rpm --ignoreos
```

### buildAppImage.js

**Purpose:** Create portable Linux AppImage.

**Process:**
1. Runs full build
2. Uses electron-builder
3. Creates .AppImage file

**Advantages:**
- Runs on any Linux distribution
- No installation needed
- Portable

**Run:**
```bash
npm run buildAppImage
```

**Output:**
```
dist/
└── Min-1.35.2.AppImage
```

**Usage:**
```bash
chmod +x Min-1.35.2.AppImage
./Min-1.35.2.AppImage
```

### Build All Platforms

**Warning:** This takes a long time and requires all platform prerequisites.

```bash
npm run buildAll
```

Builds:
- Windows
- macOS Intel
- macOS ARM
- Debian
- RedHat
- AppImage

## Build Optimization

### Speeding Up Builds

**1. Use watch mode during development:**
```bash
npm run watch
```
Much faster than full rebuilds.

**2. Build only what changed:**
```bash
npm run buildBrowser  # Just renderer
npm run buildMain     # Just main
```

**3. Disable source maps:**
In production builds, source maps are automatically disabled.

**4. Use npm ci for clean installs:**
```bash
npm ci  # Faster than npm install
```

### Reducing Bundle Size

**1. Exclude unnecessary modules:**

In `buildBrowser.js`:
```javascript
instance.exclude('heavy-module')
```

**2. Remove debug code:**

Use environment variables:
```javascript
if (process.env.NODE_ENV !== 'production') {
  console.log('Debug info')
}
```

**3. Minify in production:**

electron-builder automatically minifies.

**4. Analyze bundle:**
```bash
npm install -g bundle-buddy
browserify dist/build.js --full-paths | bundle-buddy
```

## Build Environment Variables

### Development vs Production

**Development:**
```bash
NODE_ENV=development npm start
```
- Verbose logging
- Source maps
- DevTools available

**Production:**
```bash
NODE_ENV=production npm run build
```
- Minified code
- No source maps
- Optimized performance

### Electron Build Config

Set in package.json or environment:

```bash
export DEBUG=electron-builder
npm run buildWindows
```

Shows detailed build logs.

## Troubleshooting

### Common Build Issues

**Issue: "Module not found"**

Solution:
```bash
npm ci  # Clean install
npm run build
```

**Issue: "ENOENT: no such file or directory, open 'dist/...'"**

Solution:
```bash
mkdir -p dist
npm run build
```

**Issue: Browserify errors**

Solution:
- Check syntax errors in JS files
- Ensure all requires are valid
- Run: `npm run buildBrowser` for detailed output

**Issue: Native module compilation fails**

Solution (Windows):
```bash
npm config set msvs_version 2019
npm install --force
```

Solution (macOS):
```bash
xcode-select --install
npm install
```

Solution (Linux):
```bash
sudo apt install build-essential
npm install
```

### Build Script Debugging

**Add logging:**
```javascript
console.log('Building file:', file)
```

**Check intermediate files:**
```bash
ls -la dist/
cat dist/build.js | head -50
```

**Test bundle manually:**
```bash
node dist/bundle.js
# Check for immediate errors
```

**Verbose Browserify:**
```javascript
instance.bundle()
  .on('error', function(e) {
    console.error('Build error:', e)
  })
  .on('log', function(msg) {
    console.log('Build log:', msg)
  })
```

### Platform-Specific Issues

**Windows:**
- Ensure Visual Studio is installed
- Set msvs_version: `npm config set msvs_version 2019`
- Run in Administrator mode if permission errors

**macOS:**
- Install Xcode: `xcode-select --install`
- Set SDKROOT if SDK errors
- Signing requires Apple Developer account

**Linux:**
- Install build tools: `sudo apt install build-essential`
- May need: `sudo apt install rpm` for RPM builds
- AppImage: Requires fuse

### Getting Help

If build issues persist:

1. Check GitHub issues: https://github.com/minbrowser/min/issues
2. Search Electron documentation: https://electronjs.org/
3. Ask on Discord: https://discord.gg/bRpqjJ4
4. Include:
   - Error message
   - Platform and version
   - Node.js version (`node --version`)
   - npm version (`npm --version`)
   - Build command used

## Build Scripts Quick Reference

### Development
```bash
npm install           # Install dependencies
npm run build         # Full build
npm run watch         # Watch and rebuild
npm run startElectron # Run application
npm start             # Build + watch + run
```

### Testing
```bash
npm test              # Lint code
npm run lint          # Lint and auto-fix
```

### Production Builds
```bash
npm run buildWindows     # Windows installer
npm run buildMacIntel    # macOS Intel
npm run buildMacArm      # macOS ARM
npm run buildDebian      # Debian/Ubuntu .deb
npm run buildRedhat      # RedHat/Fedora .rpm
npm run buildAppImage    # Linux AppImage
npm run buildAll         # All platforms
```

### Maintenance
```bash
npm run updateFilters    # Update ad block lists
npm run updateHttpsList  # Update HTTPS upgrade list
npm run updateSuffixes   # Update public suffix list
```

## Further Reading

- [ARCHITECTURE.md](./ARCHITECTURE.md) - System architecture
- [GETTING_STARTED.md](./GETTING_STARTED.md) - Setup guide
- [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md) - Development guide
- [electron-builder docs](https://www.electron.build/) - Packaging tool
- [Browserify docs](http://browserify.org/) - Module bundler
