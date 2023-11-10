# ChatGPT cross-platform app that run on Windows, macOS, and Linux built with the Electron framework

### Electron Quick Start
https://www.electronjs.org/docs/latest/tutorial/quick-start

## Requirements

```bash
# Platform built on V8 to build network applications
brew install node

# Convert SVG files to other image formats
brew install librsvg

# Lightweight and flexible command-line JSON processor
brew install jq
```

### Scaffold the project
Electron apps follow the same general structure as other Node.js projects. Start by creating a folder and initializing an npm package.

```bash
mkdir ChatGPT-app && cd ChatGPT-app

# npm init

cat > package.json <<'EOF'
{
  "name": "ChatGPT-app",
  "version": "1.0.0",
  "description": "ChatGPT is a free-to-use AI system. Use it for engaging conversations, gain insights, automate tasks, and witness the future of AI, all in one place.",
  "main": "main.js",
  "scripts": {
    "start": "electron ."
  }, 
  "keywords": [], 
  "author": {
    "name": "Your Name",
    "email": "your.email@example.com"
  },
  "license": "MIT"
}
EOF
```
### Install the Electron package into your app's devDependencies

```bash
# curl --silent https://registry.npmjs.org/electron | jq '."dist-tags".latest'
# npm search electron

npm install --save-dev electron
npm install electron-store
```

### Initialize the main script, file named main.js in the root folder of your project.

We created a main.js script that runs our main process, which controls our app and runs in a Node.js environment. In this script, we used Electron's app and BrowserWindow modules to create a browser window that displays web content in a separate process (the renderer).

```bash
cat > main.js <<'EOF'

// Modules to control application life and create native browser window
const { app, BrowserWindow } = require('electron')
const Store = require('electron-store')
const path = require('node:path')

function createWindow () {

  const store = new Store();

  // Create the browser window.
  const mainWindow = new BrowserWindow({
    show: false,
    width: 800,
    height: 600,
    webPreferences: {}
  })

  // and load the index.html of the app.
  mainWindow.loadURL('https://chat.openai.com')

  // Open the DevTools.
  // mainWindow.webContents.openDevTools()

  mainWindow.once('ready-to-show', () => {

    if (store.has('window_bounds')) {
      // set all bounds properties
      mainWindow.setBounds(store.get('window_bounds'))
      // console.log(store.get('window_bounds'))
    }
    mainWindow.show()
  })

  mainWindow.on('resize', () => {
    store.set('window_bounds', mainWindow.getBounds())
    // console.log(mainWindow.getBounds())
  })

  mainWindow.on('move', () => {
    store.set('window_bounds', mainWindow.getBounds())
    // console.log(mainWindow.getBounds())
  })
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.whenReady().then(() => {
  createWindow()

  app.on('activate', function () {
    // On macOS it's common to re-create a window in the app when the
    // dock icon is clicked and there are no other windows open.
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})

// Quit when all windows are closed, except on macOS. There, it's common
// for applications and their menu bar to stay active until the user quits
// explicitly with Cmd + Q.
app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
})

// app.on('before-quit', function () {
//   console.log('quit')
// })

// In this file you can include the rest of your app's specific main process
// code. You can also put them in separate files and require them here.

EOF
```

### Live test your App

```bash
# npm install
npm start
```

### Generating and setting an app icon

https://www.electronforge.io/guides/create-and-add-icons

```bash
mkdir images

curl --silent https://upload.wikimedia.org/wikipedia/commons/0/04/ChatGPT_logo.svg -o images/ChatGPT_logo.svg

rsvg-convert images/ChatGPT_logo.svg -w 1024            -f png -o images/icon.png
rsvg-convert images/ChatGPT_logo.svg -w $(( 1024 * 2 )) -f png -o images/icon@2x.png
rsvg-convert images/ChatGPT_logo.svg -w $(( 1024 * 3 )) -f png -o images/icon@3x.png
rsvg-convert images/ChatGPT_logo.svg -w $(( 1024 * 4 )) -f png -o images/icon@4x.png
rsvg-convert images/ChatGPT_logo.svg -w $(( 1024 * 5 )) -f png -o images/icon@5x.png
```

#### macOS app icon

```bash
mkdir icon.iconset

rsvg-convert images/ChatGPT_logo.svg -w 16   -f png -o icon.iconset/icon_16x16.png
rsvg-convert images/ChatGPT_logo.svg -w 32   -f png -o icon.iconset/icon_16x16@2x.png
rsvg-convert images/ChatGPT_logo.svg -w 32   -f png -o icon.iconset/icon_32x32.png
rsvg-convert images/ChatGPT_logo.svg -w 64   -f png -o icon.iconset/icon_32x32@2x.png
rsvg-convert images/ChatGPT_logo.svg -w 128  -f png -o icon.iconset/icon_128x128.png
rsvg-convert images/ChatGPT_logo.svg -w 256  -f png -o icon.iconset/icon_128x128@2x.png
rsvg-convert images/ChatGPT_logo.svg -w 256  -f png -o icon.iconset/icon_256x256.png
rsvg-convert images/ChatGPT_logo.svg -w 512  -f png -o icon.iconset/icon_256x256@2x.png
rsvg-convert images/ChatGPT_logo.svg -w 512  -f png -o icon.iconset/icon_512x512.png
rsvg-convert images/ChatGPT_logo.svg -w 1024 -f png -o icon.iconset/icon_512x512@2x.png

iconutil --convert icns --output images/icon.icns icon.iconset
```

### Package and distribute your application

https://www.electronjs.org/docs/latest/tutorial/quick-start#package-and-distribute-your-application
https://www.electronforge.io/

Append "config" json node to package.json

```bash
jq '. |= . + {

  "config": {
    "forge": {
      "packagerConfig": {
        "asar": true,
        "icon": "./images/icon"
      },
      "rebuildConfig": {},      
      "makers": [
        {
          "name": "@electron-forge/maker-squirrel",
          "config": {}
        },
        {
          "name": "@electron-forge/maker-zip",
          "platforms": [
            "darwin"
          ]
        },
        {
          "name": "@electron-forge/maker-deb",
          "config": {}
        },
        {
          "name": "@electron-forge/maker-rpm",
          "config": {}
        },
        {
          "name": "@electron-forge/maker-dmg",
          "config": {
            "options": {
              "icon": "./images/icon.icns"
            },
            "format": "ULFO"
          }
        }
      ]
    }
  }

}' package.json > package_temp.json

mv package_temp.json package.json
```

### Add Electron Forge as a development dependency of your app, and use its import command to set up Forge's scaffolding

```bash
npm install --save-dev @electron-forge/cli
npm install --save-dev @electron-forge/maker-dmg

npx electron-forge import

npm run make

open out/make

# npm run publish
```

### Location of runtime temporary application files (Cache, Cookies, ...)

```bash
ls -la  ~/Library/Application\ Support/ChatGPT-app
```
