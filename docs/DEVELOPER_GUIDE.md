# Developer Guide: Modifying Min Browser

This guide will help you make modifications to Min, whether you're fixing bugs, adding features, or customizing the browser for your needs.

## Table of Contents

- [Development Workflow](#development-workflow)
- [Code Organization](#code-organization)
- [Making Your First Change](#making-your-first-change)
- [Common Modification Tasks](#common-modification-tasks)
- [Testing Your Changes](#testing-your-changes)
- [Code Style and Standards](#code-style-and-standards)
- [Debugging](#debugging)
- [Performance Optimization](#performance-optimization)
- [Contributing Back](#contributing-back)

## Development Workflow

### Standard Development Loop

1. **Make changes** to source files
2. **Build** (automatic with watch mode)
3. **Reload** UI in running app (`Ctrl+Alt+R`)
4. **Test** your changes
5. **Commit** when working

### Setting Up Your Environment

```bash
# Clone and install
git clone https://github.com/minbrowser/min.git
cd min
npm install

# Start development
npm start
# This runs: build → watch → startElectron

# Or in separate terminals:
npm run watch       # Terminal 1: auto-rebuild
npm run startElectron  # Terminal 2: run app
```

### Development Mode Benefits

- Separate user data (won't affect your regular Min installation)
- Hot reload: `Ctrl+Alt+R` (Mac: `Cmd+Opt+R`)
- DevTools: `Ctrl+Shift+I` (Mac: `Cmd+Opt+I`)
- Console logging visible in terminal

### File Watching

The watch script automatically rebuilds when you save files:

- **JS files** in `js/` or `main/`: Rebuilds relevant bundle
- **CSS files**: Rebuilds styles
- **Localization files**: Rebuilds translations

**After rebuild:** Press `Ctrl+Alt+R` to see changes.

## Code Organization

Understanding where to make changes:

### Renderer Process (Browser UI)

**Location:** `js/` directory

Modify when:
- Changing UI appearance or behavior
- Adding searchbar features
- Modifying tab/task management
- Updating password manager integration
- Changing browser controls

**Entry point:** `js/default.js` (loaded in `index.html`)

**Key files:**
- `js/browserUI.js` - Core UI actions
- `js/navbar/tabBar.js` - Tab strip
- `js/searchbar/searchbar.js` - Search interface
- `js/taskOverlay/taskOverlay.js` - Task switcher
- `js/places/places.js` - History/bookmarks

### Main Process (Application Backend)

**Location:** `main/` directory

Modify when:
- Changing application lifecycle
- Modifying content filtering
- Updating download handling
- Changing permission management
- Working with native APIs
- Modifying view management

**Entry point:** `main/main.js`

**Key files:**
- `main/viewManager.js` - WebView management
- `main/menu.js` - Application menus
- `main/filtering.js` - Content blocking
- `main/download.js` - Download management
- `main/permissionManager.js` - Site permissions

### Styles

**Location:** `css/` directory

Modify when:
- Changing colors, fonts, spacing
- Updating layouts
- Adding animations
- Responsive design changes

**Build:** `npm run buildBrowserStyles`

### Preload Scripts

**Location:** `js/preload/` directory

Modify when:
- Adding new features that run in web pages
- Extracting data from pages
- Injecting code into web content

**Security note:** Preload scripts have limited Node.js access. Keep minimal and secure.

### Build Scripts

**Location:** `scripts/` directory

Modify when:
- Changing build process
- Adding new build targets
- Updating bundling logic

## Making Your First Change

Let's make a simple change to understand the workflow.

### Example: Change the New Tab Button Text

**1. Find the component:**

The "+" button is in `js/navbar/addTabButton.js`.

**2. Open the file:**

```bash
code js/navbar/addTabButton.js  # or your editor
```

**3. Look at the code:**

```javascript
var addTabButton = {
  create: function () {
    var button = document.createElement('button')
    button.className = 'add-tab-button'
    button.textContent = '+'
    // ...
```

**4. Make a change:**

```javascript
    button.textContent = '+ New'  // Changed from '+'
```

**5. The file watcher rebuilds automatically.**

**6. Reload the UI:**

In the running Min app, press `Ctrl+Alt+R` (Mac: `Cmd+Opt+R`).

**7. See your change:**

The button now says "+ New" instead of just "+".

### Example: Add a Console Log

Let's trace when tabs are created.

**1. Open `js/browserUI.js`:**

This file has the `addTab()` function.

**2. Add a log:**

```javascript
function addTab (tabId = tabs.add(), options = {}) {
  console.log('Creating new tab:', tabId, options)  // Add this
  
  // existing code...
```

**3. Reload UI:** `Ctrl+Alt+R`

**4. Check console:**

- Open DevTools: `Ctrl+Shift+I`
- Go to Console tab
- Create a new tab
- See your log message

## Common Modification Tasks

### Adding a Searchbar Plugin

Searchbar plugins provide results in the search dropdown.

**1. Create plugin file:**

Create `js/searchbar/myPlugin.js`:

```javascript
var myPlugin = {
  name: 'myPlugin',
  initialize: function () {
    // Setup if needed
  },
  showResults: function (text, input, event, suggestions) {
    // Only show for queries starting with "my:"
    if (!text.startsWith('my:')) {
      return
    }

    const query = text.slice(3) // Remove "my:" prefix
    
    // Create result items
    const results = [
      {
        title: 'My Result 1',
        descriptionBlock: 'Description for result 1',
        url: 'https://example.com/1',
        icon: 'icon-star'
      },
      {
        title: 'My Result 2', 
        descriptionBlock: 'Description for result 2',
        url: 'https://example.com/2',
        icon: 'icon-star'
      }
    ]

    // Add results to suggestions
    searchbarPlugins.addResults(this.name, results, input, event, suggestions)
  }
}

// Register plugin
searchbarPlugins.register(myPlugin)
```

**2. Load plugin in browser bundle:**

Edit `js/default.js` and add:

```javascript
require('searchbar/myPlugin.js')
```

**3. Rebuild and test:**

- Watch rebuilds automatically
- Press `Ctrl+Alt+R`
- Type "my:test" in searchbar
- See your results

### Modifying the Tab Bar

Let's add a tab counter to show how many tabs are open.

**1. Edit `js/navbar/tabBar.js`:**

Find the `updateAll` function and add:

```javascript
  updateAll: function () {
    // existing code...
    
    // Add tab counter
    var tabCount = tasks.getSelected().tabs.count()
    var counterElement = document.getElementById('tab-counter')
    if (!counterElement) {
      counterElement = document.createElement('div')
      counterElement.id = 'tab-counter'
      counterElement.className = 'tab-counter'
      tabBar.container.appendChild(counterElement)
    }
    counterElement.textContent = tabCount + ' tabs'
  }
```

**2. Add styles in `css/tabBar.css`:**

```css
.tab-counter {
  padding: 4px 8px;
  font-size: 11px;
  color: var(--theme-text-color);
  opacity: 0.7;
}
```

**3. Rebuild and reload:**

- CSS rebuilds automatically
- Press `Ctrl+Alt+R`
- See tab count displayed

### Adding a Keyboard Shortcut

Let's add `Ctrl+Shift+N` to create a new task.

**1. Edit `js/defaultKeybindings.js`:**

```javascript
  {
    keys: ['mod+shift+n'],
    fn: function () {
      browserUI.addTask()
      browserUI.addTab()
    }
  }
```

**Note:** `mod` = `Ctrl` on Windows/Linux, `Cmd` on Mac

**2. Reload UI:** `Ctrl+Alt+R`

**3. Test:** Press `Ctrl+Shift+N` to create a new task.

### Adding a Menu Item

Let's add a menu item to show the developer console.

**1. Edit `main/menu.js`:**

Find the `View` menu section and add:

```javascript
  {
    label: 'Developer Console',
    accelerator: 'CmdOrCtrl+Shift+I',
    click: function (item, window) {
      if (window) {
        getWindowWebContents(window).toggleDevTools()
      }
    }
  }
```

**2. Restart app:**

Menu changes require full restart, not just UI reload:
- Quit Min (`Ctrl+Q` or close window)
- Run `npm run startElectron` again

**3. Test:**

View menu now has "Developer Console" option.

### Customizing Content Filtering

Min's ad blocker can be customized.

**1. Edit filter lists in `main/filtering.js`:**

```javascript
// Add custom filter
FilterLists.push({
  url: 'https://mysite.com/custom-filters.txt',
  type: 'ads'
})
```

**2. Or block specific domains:**

```javascript
// In registerFiltering function, add:
details.url.includes('annoying-ads.com')
```

**3. Restart app to apply changes.**

### Adding a Setting

Let's add a setting to control whether tab count is shown.

**1. Add setting to `js/util/settings/settings.js`:**

```javascript
// In default settings object:
defaults: {
  // ... existing settings ...
  showTabCount: true  // Add this
}
```

**2. Use setting in `js/navbar/tabBar.js`:**

```javascript
  updateAll: function () {
    // existing code...
    
    if (settings.get('showTabCount')) {
      // Show tab counter (code from earlier example)
    }
  }
```

**3. Add UI control in settings page:**

Edit `pages/settings/index.html`:

```html
<div class="setting">
  <label>
    <input type="checkbox" id="setting-showTabCount">
    Show tab count in tab bar
  </label>
</div>
```

And `pages/settings/index.js`:

```javascript
// Bind to settings
bindSettingCheckbox('showTabCount')
```

**4. Rebuild, reload, and test in Preferences.**

## Testing Your Changes

### Manual Testing

**Basic checks:**
1. Does it build without errors?
2. Does the UI load correctly?
3. Does the feature work as expected?
4. Are there console errors?
5. Does it work on different platforms?

**Testing checklist for UI changes:**
- Light and dark themes
- Different window sizes
- Keyboard navigation
- Screen readers (if applicable)

### Testing Different Scenarios

**For tab/task changes:**
- Empty tabs
- Private tabs
- Multiple tasks
- Many tabs (100+)

**For search features:**
- Empty search
- No results
- Many results
- Special characters

**For navigation:**
- HTTP vs HTTPS sites
- About: pages
- File: URLs
- Invalid URLs

### Using Developer Tools

**Open DevTools:** `Ctrl+Shift+I` (Mac: `Cmd+Opt+I`)

**Console:**
- View log messages
- Check for errors
- Test functions interactively

**Elements:**
- Inspect DOM structure
- Modify styles live
- Check element properties

**Network:**
- Monitor requests
- Check filtering
- Debug loading issues

**Sources:**
- Set breakpoints
- Step through code
- Inspect variables

### Debugging Main Process

The main process doesn't have DevTools, but you can:

**1. Console logging:**

```javascript
console.log('Debug info:', variable)
```

Output appears in the terminal where you ran `npm run startElectron`.

**2. Remote debugging:**

Start with debugging flag:
```bash
electron . --development-mode --inspect=5858
```

Connect Chrome to `chrome://inspect`.

**3. Error handling:**

Add try-catch for better error messages:

```javascript
try {
  // risky code
} catch (error) {
  console.error('Error in myFunction:', error)
}
```

## Code Style and Standards

### JavaScript Style

Min uses [Standard](https://standardjs.com/) code style.

**Key points:**
- 2-space indentation
- No semicolons (except required cases)
- Single quotes for strings
- Spaces after keywords: `if (condition)`
- Space before function parens: `function name (args)`

**Check style:**
```bash
npm test  # Runs standard linter
```

**Auto-fix style:**
```bash
npm run lint  # Fixes style issues automatically
```

### Code Organization

**Module structure:**

```javascript
// Module dependencies
var dependency = require('dependency.js')

// Module code
var myModule = {
  property: value,
  
  method: function () {
    // implementation
  }
}

// Public API
module.exports = myModule
```

**Naming conventions:**
- **Variables/functions:** camelCase (`myVariable`, `myFunction`)
- **Classes:** PascalCase (`TabList`, `TaskList`)
- **Constants:** UPPER_SNAKE_CASE (`MAX_TABS`)
- **Private members:** Leading underscore (`_privateMethod`)

### Comments

**When to comment:**
- Complex algorithms
- Non-obvious code
- Workarounds for browser bugs
- Public APIs

**When NOT to comment:**
- Obvious code
- Redundant information
- Instead: make code self-documenting

**Good comments:**
```javascript
// Workaround for Electron issue #12345: views don't update correctly
// when window is minimized, so we force a repaint
webviews.update(tabId)

// Calculate frecency score: frequency * recency
// See: https://wiki.mozilla.org/User:Jesse/NewFrecency
var score = visitCount * Math.log(daysSinceLastVisit + 1)
```

**Bad comments:**
```javascript
// Increment counter
counter++

// Loop through tabs
tabs.forEach(function (tab) { ... })
```

### Commits

**Commit message format:**

```
Brief summary (50 chars or less)

More detailed explanation if needed. Wrap at 72 characters.
Explain what and why, not how.

- Bullet points are fine
- Include issue number if applicable: Fixes #123
```

**Good commits:**
- Single logical change per commit
- Working code (builds and runs)
- Tests pass
- Descriptive message

## Debugging

### Common Issues and Solutions

**Issue: Changes not appearing**

Solutions:
- Ensure watch script is running
- Press `Ctrl+Alt+R` to reload
- Check you edited the right file
- Rebuild manually: `npm run build`

**Issue: Build errors**

Solutions:
- Check syntax errors in code
- Look at terminal for error messages
- Try clean rebuild: delete `dist/`, run `npm run build`
- Check for missing dependencies

**Issue: Runtime errors**

Solutions:
- Check DevTools console (renderer)
- Check terminal output (main process)
- Add console.log() to trace execution
- Use debugger; statement to set breakpoint

**Issue: IPC not working**

Solutions:
- Verify event names match on both sides
- Check sender: main uses webContents.send()
- Check receiver: use ipc.on() in renderer
- Log messages to confirm sending/receiving

### Debugging Techniques

**1. Console logging:**

```javascript
console.log('Variable value:', variable)
console.dir(object)  // Detailed object inspection
console.trace()  // Stack trace
```

**2. Conditional breakpoints:**

```javascript
if (someCondition) {
  debugger  // Only breaks when condition is true
}
```

**3. Performance profiling:**

```javascript
console.time('operation')
// ... code to measure ...
console.timeEnd('operation')
```

**4. Network debugging:**

DevTools → Network tab shows:
- All requests
- Which are blocked
- Response times
- Headers

**5. Event debugging:**

```javascript
// Log all events
tasks.on('*', function (eventName, ...args) {
  console.log('Event:', eventName, args)
})
```

## Performance Optimization

### Best Practices

**1. Avoid unnecessary work:**

```javascript
// Bad: Rebuilds every time
function updateUI() {
  tabBar.updateAll()  // Expensive
}

// Good: Only update when needed
function updateUI() {
  if (tabsChanged) {
    tabBar.updateAll()
  }
}
```

**2. Batch updates:**

```javascript
// Bad: Multiple reflows
tabs.forEach(tab => {
  tabBar.update(tab)  // Causes reflow each time
})

// Good: Single reflow
var updates = tabs.map(prepareUpdate)
tabBar.batchUpdate(updates)
```

**3. Use requestIdleCallback:**

```javascript
// Do non-critical work when browser is idle
requestIdleCallback(function () {
  updateThumbnails()
})
```

**4. Debounce expensive operations:**

```javascript
// Don't search on every keystroke
var debouncedSearch = debounce(function (query) {
  places.search(query)
}, 300)
```

**5. Lazy loading:**

```javascript
// Don't load all tabs at startup
tabs.forEach(tab => {
  if (tab.selected) {
    loadTab(tab)
  } else {
    // Load later, when selected
    tab.loaded = false
  }
})
```

### Measuring Performance

**1. Chrome DevTools Profiler:**

- Open DevTools
- Go to Performance tab
- Click Record
- Perform action
- Stop recording
- Analyze timeline

**2. Console timing:**

```javascript
console.time('tab-load')
loadTab(tabId)
console.timeEnd('tab-load')
// Logs: tab-load: 45.123ms
```

**3. Memory profiling:**

- DevTools → Memory tab
- Take heap snapshot
- Perform action
- Take another snapshot
- Compare to find leaks

### Optimization Checklist

Before committing performance-related changes:

- [ ] Measure before and after
- [ ] Test with realistic data (100+ tabs)
- [ ] Check memory usage
- [ ] Profile CPU usage
- [ ] Test on slower machines
- [ ] Verify no regressions

## Contributing Back

### Before Creating a Pull Request

**1. Test thoroughly:**
- Works on your platform
- No console errors
- No broken functionality
- Follows existing patterns

**2. Check code style:**
```bash
npm test  # Runs linter
npm run lint  # Auto-fixes issues
```

**3. Update documentation:**
- Add comments for complex code
- Update relevant docs if needed
- Add to CHANGELOG if significant

**4. Create small, focused PRs:**
- One feature/fix per PR
- Easy to review
- Clear purpose

### Pull Request Process

**1. Fork and clone:**
```bash
git clone https://github.com/YOUR-USERNAME/min.git
cd min
git remote add upstream https://github.com/minbrowser/min.git
```

**2. Create feature branch:**
```bash
git checkout -b feature-name
```

**3. Make changes and commit:**
```bash
git add .
git commit -m "Add feature description"
```

**4. Push to your fork:**
```bash
git push origin feature-name
```

**5. Create PR on GitHub:**
- Go to your fork on GitHub
- Click "Pull Request"
- Describe your changes
- Reference any related issues

**6. Respond to review feedback:**
- Make requested changes
- Commit and push updates
- PR updates automatically

### What Makes a Good Contribution?

**Good contributions:**
- Solve real problems
- Follow existing patterns
- Include clear description
- Are well-tested
- Have clean code
- Update documentation

**Examples:**
- Bug fixes with test case
- New feature with usage docs
- Performance improvements with benchmarks
- UI improvements with screenshots
- Documentation improvements

### Getting Your PR Merged

**Tips for success:**
- Discuss large changes first (open issue)
- Keep PRs small and focused
- Respond promptly to feedback
- Be patient - reviews take time
- Follow project conventions

### Community Guidelines

- Be respectful and constructive
- Help others in issues/discussions
- Share knowledge
- Report bugs with details
- Suggest features thoughtfully

## Further Resources

- **Architecture**: [ARCHITECTURE.md](./ARCHITECTURE.md) - System design
- **Components**: [COMPONENT_REFERENCE.md](./COMPONENT_REFERENCE.md) - API details
- **Getting Started**: [GETTING_STARTED.md](./GETTING_STARTED.md) - Setup guide
- **Wiki**: https://github.com/minbrowser/min/wiki - More documentation
- **Discord**: https://discord.gg/bRpqjJ4 - Get help

## Quick Reference

### Build Commands
```bash
npm run build              # Full build
npm run buildMain          # Build main process
npm run buildBrowser       # Build renderer process
npm run buildBrowserStyles # Build CSS
npm run buildPreload       # Build preload scripts
npm run watch              # Watch and rebuild
```

### Run Commands
```bash
npm start                  # Build, watch, and run
npm run startElectron      # Run without building
npm test                   # Run linter
npm run lint               # Fix style issues
```

### Keyboard Shortcuts (in running app)
- `Ctrl+Alt+R` - Reload UI
- `Ctrl+Shift+I` - DevTools
- `Ctrl+Q` - Quit

### Key Files to Know
- `js/default.js` - Renderer entry point
- `main/main.js` - Main process entry point
- `js/browserUI.js` - Core UI actions
- `js/tabState/task.js` - Tab/task data model
- `main/viewManager.js` - WebView management
