# Fresh

A terminal-based text editor.

This is a completely free and open source project, not owned by any commerical company.

**Status:** Alpha preview, early adopters version -0.1

## Features

- **Easy** - Intuitive out of the box with command palette and discoverable menus
- **Huge file support** - Opens multi-gigabyte files in milliseconds
- **Lightweight** - Low memory footprint, minimal CPU usage, instant startup
- **TypeScript plugins** - Extend the editor with plugins that run in a sandboxed Deno environment
- **LSP integration** - Diagnostics, completion, and go-to-definition out of the box
- **Powerful editing** - Multi-cursor support, macros, split views, etc.

![Fresh Screenshot](docs/screenshot1.png)
![Fresh Screenshot](docs/screenshot2.png)
![Fresh Screenshot](docs/screenshot3.png)

## Installation

```bash
cargo build --release
./target/release/fresh [file]
```

## Documentation

- [User Guide](docs/USER_GUIDE.md)
- [Plugin Development](docs/PLUGIN_DEVELOPMENT.md)
- [Architecture](docs/ARCHITECTURE.md)

## License

Copyright (c) Noam Lewis

This project is licensed under the GNU General Public License v2.0 (GPL-2.0).
