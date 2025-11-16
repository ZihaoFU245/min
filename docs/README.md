# Min Browser Documentation

Welcome to the Min Browser documentation! This collection of guides will help you understand, use, and modify Min.

## Documentation Overview

### For Users

**[GETTING_STARTED.md](./GETTING_STARTED.md)**
- Installation instructions
- First-time setup
- Basic usage guide
- Keyboard shortcuts
- Advanced features
- Troubleshooting

Start here if you're new to Min or want to learn how to use its features effectively.

### For Developers

**[ARCHITECTURE.md](./ARCHITECTURE.md)**
- High-level system design
- Component relationships
- Data flow diagrams
- Process architecture (main/renderer/web content)
- Key concepts (Tasks, Places, etc.)
- Threading model
- Security architecture

Read this to understand how Min is structured and how its components work together.

**[DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md)**
- Development workflow
- Making your first change
- Common modification tasks
- Testing and debugging
- Code style and standards
- Contributing guidelines

Follow this guide when you want to modify Min's code or contribute to the project.

**[COMPONENT_REFERENCE.md](./COMPONENT_REFERENCE.md)**
- Detailed API documentation
- Task and tab management
- Places system
- Webview management
- Searchbar plugins
- Content filtering
- Password manager integration
- Settings system
- IPC communication

Use this as a reference when working with specific components or APIs.

**[BUILD_SYSTEM.md](./BUILD_SYSTEM.md)**
- Build pipeline overview
- Build script details
- Watch mode
- Platform-specific builds
- Build optimization
- Troubleshooting

Consult this when working with the build system or creating distribution packages.

## Quick Navigation

### I want to...

**...install and use Min**
→ [GETTING_STARTED.md](./GETTING_STARTED.md)

**...understand how Min works**
→ [ARCHITECTURE.md](./ARCHITECTURE.md)

**...make changes to Min**
→ [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md)

**...look up a specific API**
→ [COMPONENT_REFERENCE.md](./COMPONENT_REFERENCE.md)

**...build Min from source**
→ [BUILD_SYSTEM.md](./BUILD_SYSTEM.md)

**...add a searchbar plugin**
→ [DEVELOPER_GUIDE.md#adding-a-searchbar-plugin](./DEVELOPER_GUIDE.md#adding-a-searchbar-plugin)

**...understand tab management**
→ [COMPONENT_REFERENCE.md#task-and-tab-management](./COMPONENT_REFERENCE.md#task-and-tab-management)

**...debug a build issue**
→ [BUILD_SYSTEM.md#troubleshooting](./BUILD_SYSTEM.md#troubleshooting)

## Documentation Organization

```
docs/
├── README.md                    # This file - Documentation index
├── GETTING_STARTED.md           # User and developer setup guide
├── ARCHITECTURE.md              # System design and architecture
├── DEVELOPER_GUIDE.md           # How to modify Min
├── COMPONENT_REFERENCE.md       # Detailed API documentation
├── BUILD_SYSTEM.md              # Build system details
└── statistics.md                # Usage statistics info
```

## Key Concepts

Before diving into the documentation, here are some key concepts to understand:

### Tasks
Tasks are Min's way of organizing tabs into groups (similar to tab groups or workspaces). Each task:
- Contains its own set of tabs
- Can be named
- Maintains independent history
- Can be saved and restored

### Places
The Places system is Min's integrated database for:
- Browsing history
- Bookmarks with tags
- Full-text search of page content

### WebViews
Min uses Electron's `WebContentsView` to display web pages. Each tab has its own isolated view that:
- Runs in a separate process
- Is fully sandboxed
- Has limited access to Node.js APIs
- Can be shown/hidden without reloading

### Searchbar Plugins
The searchbar is extensible through a plugin system. Plugins can:
- Provide search results
- Access history and bookmarks
- Integrate external services
- Add custom UI

### Content Filtering
Built-in ad and tracker blocking using:
- Filter lists (EasyList, EasyPrivacy)
- HTTPS upgrades
- Per-site toggle

## Architecture at a Glance

Min is built on Electron and follows this structure:

```
┌─────────────────────────────────────────────────────────────┐
│                        Main Process                          │
│  • Application lifecycle                                     │
│  • Window management                                         │
│  • WebView management                                        │
│  • Content filtering                                         │
│  • Native integrations                                       │
└─────────────────────────────────────────────────────────────┘
                            ↕ IPC
┌─────────────────────────────────────────────────────────────┐
│                      Renderer Process                        │
│  • Browser UI (tabs, searchbar, menus)                      │
│  • Tab/task management                                       │
│  • Places database                                           │
│  • User interactions                                         │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                    Web Content Views                         │
│  • Sandboxed web pages                                       │
│  • Each tab in separate process                             │
│  • Limited preload scripts                                   │
└─────────────────────────────────────────────────────────────┘
```

See [ARCHITECTURE.md](./ARCHITECTURE.md) for detailed information.

## Common Tasks

### Setup for Development

```bash
# Clone repository
git clone https://github.com/minbrowser/min.git
cd min

# Install dependencies
npm install

# Build and run
npm start
```

See [GETTING_STARTED.md](./GETTING_STARTED.md) for detailed setup instructions.

### Making Changes

1. Edit source files in `js/` or `main/`
2. Watch mode rebuilds automatically
3. Press `Ctrl+Alt+R` to reload UI
4. Test your changes

See [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md) for development workflow.

### Building for Distribution

```bash
# Windows
npm run buildWindows

# macOS Intel
npm run buildMacIntel

# macOS ARM (Apple Silicon)
npm run buildMacArm

# Linux .deb
npm run buildDebian

# Linux .rpm
npm run buildRedhat

# Linux AppImage
npm run buildAppImage
```

See [BUILD_SYSTEM.md](./BUILD_SYSTEM.md) for platform build details.

## Component Hierarchy

### Main Process Components

```
main.js (Entry point)
├── viewManager.js (WebView management)
├── menu.js (Application menus)
├── filtering.js (Content blocking)
├── download.js (Download handling)
├── permissionManager.js (Site permissions)
├── remoteActions.js (IPC handlers)
└── ... (other modules)
```

### Renderer Process Components

```
default.js (Entry point)
├── browserUI.js (Core UI orchestration)
├── tabState/
│   ├── task.js (TaskList)
│   └── tab.js (TabList)
├── navbar/
│   ├── tabBar.js (Tab strip)
│   ├── tabEditor.js (URL input)
│   └── ... (other navbar components)
├── searchbar/
│   ├── searchbar.js (Search interface)
│   ├── searchbarPlugins.js (Plugin system)
│   └── ... (plugin implementations)
├── places/
│   ├── places.js (API)
│   ├── placesService.js (Database service)
│   └── fullTextSearch.js (Search engine)
├── webviews.js (WebView management)
└── ... (other modules)
```

See [ARCHITECTURE.md](./ARCHITECTURE.md) for complete component breakdown.

## Development Workflow

### Standard Loop

1. **Make changes** to source files
2. **Watch rebuilds** automatically
3. **Reload UI** with `Ctrl+Alt+R`
4. **Test** changes
5. **Commit** when working
6. **Create PR** to contribute

### Testing Changes

- **Manual testing** in running app
- **DevTools** (`Ctrl+Shift+I`) for debugging
- **Console logs** for tracing
- **Linting** with `npm test`

See [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md) for detailed workflow.

## Code Style

Min uses [Standard](https://standardjs.com/) JavaScript style:

```bash
# Check style
npm test

# Auto-fix issues
npm run lint
```

Key points:
- 2-space indentation
- No semicolons
- Single quotes
- Space after keywords

See [DEVELOPER_GUIDE.md#code-style-and-standards](./DEVELOPER_GUIDE.md#code-style-and-standards) for details.

## File Organization

### Source Files

```
min/
├── main/           # Main process (Node.js)
├── js/             # Renderer process (Browser UI)
│   ├── navbar/     # Navigation bar components
│   ├── searchbar/  # Search interface and plugins
│   ├── tabState/   # Tab and task management
│   ├── places/     # History/bookmark database
│   ├── preload/    # Preload scripts for web content
│   └── util/       # Shared utilities
├── css/            # Stylesheets
├── pages/          # Special pages (settings, etc.)
├── localization/   # Translation files
└── icons/          # Icon resources
```

### Build Output

```
dist/
├── main.build.js       # Main process bundle
├── bundle.js           # Renderer bundle
├── bundle.css          # Compiled styles
├── preload.js          # Preload bundle
└── localization.build.js  # Translations
```

## Keyboard Shortcuts

### Development

- `Ctrl+Alt+R` - Reload UI (development mode)
- `Ctrl+Shift+I` - DevTools
- `Ctrl+Q` - Quit

### User

- `Ctrl+L` - Focus address bar
- `Ctrl+T` - New tab
- `Ctrl+W` - Close tab
- `Ctrl+Tab` - Next tab
- `Shift+Ctrl+E` - Task overlay
- `Ctrl+F` - Find in page

See [GETTING_STARTED.md#keyboard-shortcuts](./GETTING_STARTED.md#keyboard-shortcuts) for complete list.

## Resources

### Official

- **Repository**: https://github.com/minbrowser/min
- **Website**: https://minbrowser.org/
- **Releases**: https://github.com/minbrowser/min/releases
- **Wiki**: https://github.com/minbrowser/min/wiki

### Community

- **Discord**: https://discord.gg/bRpqjJ4
- **Issues**: https://github.com/minbrowser/min/issues
- **Discussions**: https://github.com/minbrowser/min/discussions
- **Sponsors**: https://github.com/sponsors/PalmerAL

### Related

- **Electron**: https://electronjs.org/
- **Chromium**: https://www.chromium.org/
- **Standard JS**: https://standardjs.com/

## Contributing

We welcome contributions! Please:

1. Read [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md)
2. Check existing [issues](https://github.com/minbrowser/min/issues)
3. Discuss large changes first
4. Follow code style guidelines
5. Test thoroughly
6. Create focused pull requests

See [DEVELOPER_GUIDE.md#contributing-back](./DEVELOPER_GUIDE.md#contributing-back) for details.

## Getting Help

### Documentation

Start with these docs, especially:
- [GETTING_STARTED.md](./GETTING_STARTED.md) for setup issues
- [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md) for development questions
- [BUILD_SYSTEM.md](./BUILD_SYSTEM.md) for build problems

### Community Support

- **Discord**: Quick help from community
- **GitHub Issues**: Bug reports and feature requests
- **GitHub Discussions**: Questions and ideas

### Bug Reports

When reporting bugs, include:
- Min version
- Operating system
- Steps to reproduce
- Expected vs actual behavior
- Screenshots if applicable
- Console errors (if any)

## License

Min is open source under the Apache-2.0 License.

See [LICENSE.txt](../LICENSE.txt) for full details.

## Acknowledgments

Min is made possible by:
- **Contributors**: Everyone who has contributed code, docs, or translations
- **Sponsors**: Supporting ongoing development
- **Community**: Users providing feedback and ideas
- **Dependencies**: Electron, Chromium, and many npm packages

Thank you all!

---

**Need help?** Join our [Discord](https://discord.gg/bRpqjJ4) or open an [issue](https://github.com/minbrowser/min/issues).

**Ready to contribute?** Read the [DEVELOPER_GUIDE.md](./DEVELOPER_GUIDE.md) and dive in!
