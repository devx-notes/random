# Electron Process Model

Electron applications consist of two main process types—the Main Process and one or more Renderer Processes—plus an optional Preload Process for secure bridging.

| Process        | Technology        | Runs In                 | Purpose                                                     |
| -------------- | ----------------- | ----------------------- | ----------------------------------------------------------- |
| Main Process   | Node.js           | OS-level native process | Manages windows, menus, IPC, file I/O                       |
| Renderer       | Chromium + JS     | Web sandbox             | Renders app UI (HTML/CSS/JS)                                |
| Preload Script | Node.js (limited) | In WebView, before UI   | Secure bridge, defines allowed APIs between Main & Renderer |

## 1. Security Recommendation

```js
const win = new BrowserWindow({
    webPreferences: {
        nodeIntegration: false, // default behavior since v5.0.0
        contextIsolation: true, // default behavior since v12.0.0
        preload: path.join(__dirname, 'preload.js')
    }
})
```

### 1.1. Process Sandboxing

### 1.2. NodeJS Integration

### 1.3. Context Isolation

This feature allows developers to run code in preload scripts and in Electron APIs in a dedicated JavaScript context

## 2. Bridging

Electron provides the `contextBridge` module to create a safe, bi-directional, synchronous bridge across isolated contexts

```js
// preload.js
const { contextBridge, ipcRenderer } = require('electron')

contextBridge.exposeInMainWorld('electron', {
    doThing: () => ipcRenderer.send('do-a-thing')
})

// renderer
window.electron.doThing()
```

## 3. IPC Channels

In Electron, processes communicate by passing messages through developer-defined channels with the `ipcMain` and `ipcRenderer` modules