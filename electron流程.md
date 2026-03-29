# 鬼追人 - 打包为 EXE 流程

## 前置条件

- 已安装 [Node.js](https://nodejs.org/)（v18+）
- 游戏 HTML 文件已准备好（鬼追人.html、游戏.html、游戏说明.html、游戏设置.html）

---

## 1. 创建 Electron 项目结构

在游戏根目录下创建 `ghostman` 文件夹，结构如下：

```
ghostman/
├── main.js          # Electron 主进程
├── package.json     # 项目配置
└── game/            # 游戏文件（复制进来）
    ├── 鬼追人.html
    ├── 游戏.html
    ├── 游戏说明.html
    └── 游戏设置.html
```

## 2. main.js 内容

```javascript
const { app, BrowserWindow } = require('electron');
const path = require('path');

function createWindow() {
    const win = new BrowserWindow({
        width: 900,
        height: 700,
        title: '鬼追人',
        autoHideMenuBar: true,
        resizable: true,
        webPreferences: {
            nodeIntegration: false,
            contextIsolation: true
        }
    });

    win.loadFile(path.join(__dirname, 'game', '鬼追人.html'));
    win.setMenu(null);
}

app.whenReady().then(createWindow);
app.on('window-all-closed', () => { app.quit(); });
```

## 3. package.json 内容

```json
{
  "name": "ghost-chase",
  "version": "1.0.0",
  "description": "鬼追人 - 迷宫追逐游戏",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "build": "electron-packager . 鬼追人 --platform=win32 --arch=x64 --out=dist --overwrite --no-prune"
  },
  "devDependencies": {
    "electron": "^28.0.0",
    "electron-packager": "^17.1.2"
  }
}
```

## 4. 复制游戏文件

将 4 个 HTML 文件复制到 `ghostman/game/` 目录：

```powershell
mkdir ghostman\game
copy 鬼追人.html ghostman\game\
copy 游戏.html ghostman\game\
copy 游戏说明.html ghostman\game\
copy 游戏设置.html ghostman\game\
```

## 5. 安装依赖

```powershell
cd ghostman
npm install
```

## 6. 测试运行（可选）

```powershell
npx electron .
```

> **说明**：
> - `npx` 是 npm 自带的工具，用于执行 node_modules 中的包
> - `electron` 是要执行的包名
> - `.` 表示当前目录（即 `ghostman` 文件夹），Electron 会读取该目录下的 `package.json` 找到 `main.js` 并运行

## 7. 打包为 EXE

```powershell
npx electron-packager . 鬼追人 --platform=win32 --arch=x64 --out=dist --overwrite --no-prune
```

## 8. 输出结果

打包完成后，exe 位于：

```
ghostman/dist/鬼追人-win32-x64/鬼追人.exe
```

双击 **鬼追人.exe** 即可运行游戏。整个 `鬼追人-win32-x64` 文件夹可以压缩为 zip 发给别人，无需安装任何环境。

---

## 🔄 修改游戏后重新打包（快速流程）

> 如果你已经完成过首次打包（第 1~8 步），之后只需按以下步骤操作即可。

### 步骤一：修改根目录的 HTML 文件

直接编辑项目根目录下的 HTML 文件（如 `鬼追人.html`、`游戏.html`、`游戏说明.html`、`游戏设置.html`），保存修改。

### 步骤二：复制修改后的文件到 ghostman/game/

将修改过的 HTML 文件重新复制到 `ghostman/game/` 目录，覆盖旧文件：

```powershell
# 在项目根目录下执行（跑跑抓游戏 文件夹）
copy /Y 鬼追人.html ghostman\game\
copy /Y 游戏.html ghostman\game\
copy /Y 游戏说明.html ghostman\game\
copy /Y 游戏设置.html ghostman\game\
```

> `/Y` 参数表示自动覆盖，不需要确认。
> 也可以只复制你修改过的那个文件，不用全部复制。

### 步骤三：（可选）先测试运行，确认效果

```powershell
cd ghostman
npx electron .
```

> `.` 表示当前目录，确保你已经在 `ghostman` 文件夹中执行此命令

测试没问题后关闭窗口，继续打包。

### 步骤四：重新打包为 EXE

```powershell
cd ghostman
npx electron-packager . 鬼追人 --platform=win32 --arch=x64 --out=dist --overwrite --no-prune
```

> `--overwrite` 会自动覆盖上次的打包结果，无需手动删除。

### 步骤五：打包完成

新的 exe 还是在原来的位置：

```
ghostman/dist/鬼追人-win32-x64/鬼追人.exe
```

双击即可运行修改后的游戏。

---

## 📦 打包体积优化

Electron 打包后约 **150-500MB**，对于简单的 HTML 游戏来说确实很大。以下是几种优化方案：

### 方案 1：使用 `--prune=true` 清理开发依赖（轻度优化）

修改打包命令，自动删除 devDependencies：

```powershell
npx electron-packager . 鬼追人 --platform=win32 --arch=x64 --out=dist --overwrite --prune=true
```

> 可节省约 50-100MB

### 方案 2：使用 electron-builder + asar 打包（中度优化）

1. 安装 electron-builder：
```powershell
npm install --save-dev electron-builder
```

2. 修改 `package.json`，添加 build 配置：
```json
{
  "build": {
    "appId": "com.ghostman.game",
    "productName": "鬼追人",
    "asar": true,
    "files": ["main.js", "game/**/*"],
    "win": {
      "target": "portable"
    }
  }
}
```

3. 打包命令：
```powershell
npx electron-builder --win portable
```

> asar 压缩后可节省 30-40%，且加载更快

### 方案 3：切换到轻量级框架（重度优化，推荐）

#### 使用 Tauri（推荐）- 打包后仅 **3-10MB**

Tauri 使用系统自带浏览器内核，体积极小：

```powershell
# 安装 Rust（一次性操作）
# 下载：https://rustup.rs/

# 创建 Tauri 项目
npm create tauri-app
```

#### 使用 Neutralinojs - 打包后仅 **3-5MB**

更简单，无需 Rust：

```powershell
npm install -g @neutralinojs/neu
neu create ghostman-neu
```

### 方案 4：Web 版（0MB，直接网页运行）

最简单的方案：直接用浏览器打开 HTML 文件，无需打包！

```powershell
# 本地测试
start 鬼追人.html
```

如果要分发给朋友，可以：
- 打包成 zip，让对方解压后双击 HTML
- 部署到 GitHub Pages（免费托管）

---

## ⚠️ 注意事项

1. **修改游戏后需重新打包**：修改 HTML 文件后，需重新复制到 `game/` 目录并重新执行打包步骤（见上方快速流程）
2. **不要直接改 ghostman/game/ 里的文件**：建议始终修改根目录的 HTML，然后复制过去，避免下次复制时覆盖丢失修改
3. **文件夹较大**：Electron 打包后约 150-500MB（因为包含了 Chromium 内核），可参考上方优化方案
4. **推荐轻量方案**：对于简单 HTML 游戏，建议直接用浏览器打开或使用 Tauri/Neutralinojs
