# Electron 101

## 1. Global Variables

The main process has full access to both Node.js globals and Electron modules. These are alse available in renderer process if nodeIntegration is set to true.

> [!NOTE]
> In CommonJS style, Electron module globals are typically required explicitly. But under certain conditions (like REPL or loosely loaded scripts), they may appear global-like because of Node's module caching.
> 
> ```js
> const { app, BrowserWindow, ipcMain } = require('electron');
> ```

| Global     | Description                                     |
| ---------- | ----------------------------------------------- |
| __dirname  | Directory name of the current module            |
| __filename | Absolute path of the current module file        |
| global     | Node.js global scope (like `window` in browser) |
| process    | Node.js process object                          |
| Buffer     | Node.js buffer object                           |
| require    | Node.js module system                           |

The renderer process runs in a Chromium environment (sandboxed browser), so browser-like globals are available

| Object                         | Description                  |
| ------------------------------ | ---------------------------- |
| window                         | Main browser context         |
| document                       | HTML document object         |
| navigator                      | User agent and platform info |
| location                       | URL of the current page      |
| console                        | Logging interface            |
| setTimeout, clearTimeout, etc. | Timers                       |

The preload script runs before the renderer is loaded. It has access to Node.js and Electron + limited browser globals. This is the safe bridge for exposing functionality to the renderer.

### Electron Module Globals

Electron exposes a suite of core modules, these modules are used to build every core feature of an Electron app, from window creation, IPC communication, menus, system tray access, file dialogs, and more.