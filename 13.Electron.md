### 进程

#### main进程

- app模块

app模块是electron应用的骨架。他掌管着整个应用的生命周期钩子，以及很多其他事件钩子。app常用生命周期钩子如下：
```
will-finish-launching 在应用完成基本启动进程后触发
ready 当electron完成初始化后触发
window-all-closed 所有窗口都关闭的时候触发，在windows和linux里，所有窗口都退出的时候通常是应用退出的时候
before-quit 退出应用之前的时候触发
will-quit 即将退出应用的时候触发
quit 应用退出的时候触发
```
**注意**：我们常在`ready`的时候执行`创建窗口`，`创建应用菜单`，`创建应用快捷键`等初始化操作，而在`will-quit`以及`quit`的时候执行一些清空操作，比如：`解绑快捷键`

- BrowserWindow

**常用配置**

```js
let window
function createWindow (){
  window = new BrowserWindow({
    height:900, //高
    width:400, //宽
    show:false, //创建后是否显示
    frame:false, //是否创建frameless窗口
    fullscreenable:false, //是否允许全屏
    center:true, //是否出现在屏幕居中的位置
    backgroundColor:'#fff', //背景色，用于transparent和frameless窗口
    titleBarStyle:'hidden', //标题栏的样式，有hidden、hiddenInset、customButtonsOnHover等
    resizable:false, //是否允许拉伸大小
    transparent:false, //是否是透明窗口（仅macOS）
    vibrancy:'ultra-dark', //窗口模糊的样式（仅macOS）
    webPreferences:{
      backgroundThrottling:false //当页面被置于非激活窗口的时候是否停止动画和计时器
    }
    //...以及其他可选配置
  })

  window.loadURL(url)

  window.on('closed',() => {window = null})
}
```

**frame**：默认为`true`，`false`情况下创建一个`没有顶部工具栏`，`没有border`的窗口。这也是我们在`windows`系统下自定义顶部工具栏的基础
**常用钩子**

```
closed 窗口被关闭的时候
focus 窗口被激活的时候
show 窗口显示的时候
hide 窗口隐藏的时候
maxsize 窗口最大化的时候
minsize 窗口最小化的时候
```

**常用方法**

```
BrowserWindow.getFocusedWindow() [静态方法]获取激活的窗口
win.close() [实例方法，下同]关闭窗口
win.focus() 激活窗口
win.show() 显示窗口
win.hide() 隐藏窗口
win.maximize() 最大化窗口
win.minimize() 最小化窗口
win.restore() 从最小化窗口恢复
```

- Tray(图标组件)

需要注意的是，windows 和 macOS 里，图标的大小都是 16*16px。macOS 下顶部栏的图标通常都是走黑白路线，所以可以为两种系统分别准备不同的图标

- menu

#### render进程

使用路由`hash`模式

### titlebar实现

因为默认的 tilebar 样式一般不会符合我们的设计，所以很多时候我们会自定义这一部分内容

```
titleBarStyle: 'hidden'
```

不过在平时的使用中，我们要注意，一般我们鼠标按住 titlebar 的时候是可以拖动窗口的。但是如果我们在不加可拖拽的属性之前，我们自己写的 titlebar 是不具备这样的特性的。要加上这个特性也很简单：

```css
.handle-bar {
  -webkit-app-region:no-drag;
}
```

### drag&drop的避免

通常我们用`Chrome`的时候，有个特性是比如你往`Chrome`里拖入一个`pdf`，它就会自动用内置的`pdf`阅读器打开。你往`Chrome`里拖入一张图片，它就会打开这张图片。由于我们的`electron`应用的`BrowserWindow`其实内部也是一个浏览器，所以这样的特性依然存在。而这也是很多人没有注意的地方。也就是当你开发完一个`electron`应用之后，往里拖入一张图片，一个`pdf`等等，如果不是一个可拖拽区域（比如`PicGo`的上传区），那么它就不应该打开这张图、这个`pdf`，而是将其排除在外。

所以我们将在全局监听`drag`和`drop`事件，当用户拖入一个文件但是又不是拖入可拖拽区域的时候，应该将其屏蔽掉。因为所有的页面都应该要有这样的特性，因此合理使用`vue`的`mixin`很重要

```js
export default {
  mounted() {
    this.disableDragEvent();
  },
  methods: {
    disableDragEvent() {
      window.addEventListener("dragenter", this.disableDrag, false);
      window.addEventListener("dragover", this.disableDrag);
      window.addEventListener("drop", this.disableDrag);
    },
    disableDrag(e) {
      const dropzone = document.getElementById("upload-area"); // 这个是可拖拽的上传区
      if (dropzone === null || !dropzone.contains(e.target)) {
        e.preventDefault();
        e.dataTransfer.effectAllowed = "none";
        e.dataTransfer.dropEffect = "none";
      }
    },
  },
  beforeDestroy() {
    window.removeEventListener("dragenter", this.disableDrag, false);
    window.removeEventListener("dragover", this.disableDrag);
    window.removeEventListener("drop", this.disableDrag);
  },
}
```

### 进程间通信

`electron`里的不同进程间的通信是通过`ipcMain`和`ipcRenderer`来实现的。其中`ipcMain`是在`main`进程里使用的，而`ipcRenderer`是在`renderer`进程里使用的。

官网的例子：

```js
// In main process.
const { ipcMain } = require("electron");
ipcMain.on("asynchronous-message", (event, arg) => {
  console.log(arg); // prints "ping"
  event.sender.send("asynchronous-reply", "pong");
});

ipcMain.on("synchronous-message", (event, arg) => {
  console.log(arg); // prints "ping"
  event.returnValue = "pong";
});
```

```js
// In renderer process (web page).
const { ipcRenderer } = require("electron");
console.log(ipcRenderer.sendSync("synchronous-message", "ping")); // prints "pong"

ipcRenderer.on("asynchronous-reply", (event, arg) => {
  console.log(arg); // prints "pong"
});
ipcRenderer.send("asynchronous-message", "ping");
```

其中`ipcMain`只有监听来自`ipcRenderer`的某个事件后才能返回给`ipcRenderer`值。而`ipcRenderer`既可以收，也可以发。
`ipcMain`无法主动发消息给`ipcRenderer`，使用`webContents`

### webContents

`webContents`其实是`BrowserWindow`实例的一个属性。也就是如果我们需要在`main`进程里给某个窗口某个页面发送消息，则必须通过`win.webContents.send()`方法来发送。
代码如下：

```js
// In main process
let win = new BrowserWindow({...})
win.webContents.send('img-files', imgs)


// In renderer process
ipcRenderer.on('img-files', (event, files) => {
  console.log(files)
})
```

### js数据库

#### lowdb

- 初始化

**注意**：`lowdb`每次调用数据的时候需要显式调用其`read`方法，保证各个进程获取到的数据一致性
**tips**：为了操作`fs`更方便，不妨安装一个[fs-extra](https://github.com/jprichardson/node-fs-extra)
创建一个`datastore.js`文件：

```js
/**
 * lowdb 设置
 */

import Datastore from "lowdb";
import LodashId from "lodash-id";
import FileSync from "lowdb/adapters/FileSync";
import path from "path";
import fs from "fs-extra";
import { remote, app } from "electron";

if (process.env.NODE_ENV !== "development") {
  global.__static = path.join(__dirname, "/static").replace(/\\/g, "\\\\");
}
if (process.env.DEBUG_ENV === "debug") {
  global.__static = path.join(__dirname, "../../static").replace(/\\/g, "\\\\");
}

const APP = process.type === "renderer" ? remote.app : app;
const STORE_PATH = APP.getPath("userData") + "/hdCloud"; // 定义存储地址根路径

if (!fs.pathExistsSync(STORE_PATH)) {
  // 是否存在根路径，不存在则创建
  fs.mkdirpSync(STORE_PATH);
}

const scheme = Datastore(new FileSync(path.join(STORE_PATH, "/hdScheme.json"))); // 创建数据库文件
const config = Datastore(new FileSync(path.join(STORE_PATH, "/hdConfig.json")));

scheme._.mixin(LodashId);
config._.mixin(LodashId); // 使用lodashId为每一个新增的记录生成随机数组成的id

const db = {
  scheme,
  config,
};

if (!db.config.has("formualList").value()) {
  // 如果不存在根节点创建
  db.config.set("formualList", []).write();
}

if (!db.config.has("templateList").value()) {
  db.config.set("templateList", []).write();
}

export default db;
```

- 基本操作

**创建**：主要通过`set()`或者`defaults()`方法。其中`defaults()`专门针对空`JSON文件`进行初始化。（不过用`set`也是可以实现类似的，如上一小节说到的初始化）

```js
db.defaults({ posts: [], user: {}, count: 0 }).write(); // 一定要显式调用write方法将数据存入JSON
```

**注意任何写的操作，都必须显式的使用 write()方法来保存。**
**获取数据库：**

```js
// 获取数据库
getDB () {
  db[this.dbName].read()
  return db[this.dbName].get(this.tableName) // 放回当前实例方便链式调用其他方法
}
```

**读取：**

```js
getList () {
  return this.getDB().value()
}
```

还可以用`lodash`的一些方法来查询你的`JSON`。

```js
// 根据字段查找
db.get('posts').find({ id: 1 }).value()

getInfoByInfo (info) { // {id: 1}
  return this.getDB().find(info).value()
}

// 根据Id查找
getInfoById (id) {
  return this.getDB().getById(id).value()
}
```

**注意任何读的操作，都必须显式使用value()方法来获取值。**

**新增：**

```js
// 针对对象就用赋值，针对数组就用push()或者insert()（lowdb-id提供的方法）
// 添加信息
addInfo (info) {
  this.getDB().insert(info).write()
  return this // 一般新增完毕会返回更新后数据，因此返回实例
}
```

**更新：**更新的时候我们可以根据不同的结构来更新

```js
// 针对对象可以直接用set()来更新：
db.set('user.name', 'typicode').write() // 通过set方法来对对象操作

// 也可以：
db.set('user', {
  name: 'typicode'
}).write()

// 针对原有的数据进行更新的可以用update。
db.update('count', n => n + 1).write() // update方法使用已存在的值来操作

// 针对对象可以使用 assign 写入
// 更新信息
updateInfo (info) {
  this.getDB().getById(info.id).assign(info).write()
  return this
}
```

**删除：**

```js
// 根据字段进行查找删除
db.get('posts').remove({ title: 'low!' }).write()

// 可以通过unset来删除一个属性：
db.unset('user.name').write()

// 也可以通过lodash-id提供的removeById()来删除指定id的项：
deleteInfoById (id) {
  this.getDB().removeById(id).write()
  return this
}
```

### 跨平台兼容措施

主要的工具是通过 process.platform 来判断不同的平台。当前可能的值有：

```
"aix"
"darwin"--macOs
"freebsd"
"linux"
"openbsd"
"sunos"
"win32"--windows
```

### excle操作

我们使用`node-xlsx`插件进行相关操作

- 写入：

```js
const buffer = xlsx.build(this.list);
const _path = path.join(__dirname, ".", "result.xlsx");
fs.writeFile(_path, buffer, function(err) {
  if (err) {
    throw err;
  }
  console.log("Write to xls has finished");
});
```

- 读取：

```js
const _path = path.join(__dirname, ".", "result.xlsx");
// 读xlsx
var obj = xlsx.parse(_path);
console.log(JSON.stringify(obj, null, 2));
```

**注意：**`node-xlsx`对于`excle`里面的行列合并采用的是填空的方式处理

### 使用node api

`electron`同时在主进程和渲染进程中对`node.js`暴露了所有的接口，需要注意：

- 所有在 node 中可以使用的 api，在 electron 中依旧可以使用，例如：

```js
const fs = require("fs");
const root = fs.readdirSync("/");

// 这会打印出磁盘根级别的所有文件
// 同时包含'/'和'C:\'。
console.log(root);
```

- 读取：

```js
const APP = process.type === "renderer" ? remote.app : app;
const STORE_PATH = APP.getPath("userData") + "/hdCloud/scheme"; // 定义根路径

if (!fs.pathExistsSync(STORE_PATH)) {
  // 没有则创建
  fs.mkdirpSync(STORE_PATH);
}

schemeModule.readFile = function(fileName) {
  return new Promise((resolve, reject) => {
    const _path = path.join(STORE_PATH, ".", `${fileName}.json`);
    if (!fs.pathExistsSync(_path)) {
      fs.writeFile(_path, JSON.stringify({}));
      resolve({});
    } else {
      fs.readFile(_path, "utf8", (err, data) => {
        if (!err) {
          resolve(JSON.parse(data));
        } else {
          reject(err);
        }
      });
    }
  });
};
```

- 写入：

```js
schemeModule.writeFile = function(file, fileName) {
  const _path = path.join(STORE_PATH, ".", `${fileName}.json`);
  fs.writeFile(_path, JSON.stringify(file), function(err) {
    if (!err) return "写入成功";
  });
};
```

### 利用 electron-builder 打包

```js
"build": {
    "productName": "HDcalculator", // 打包出来的软件名
    "appId": "huidian.calculator.com", // appid
    "directories": { // 打包输出地址
      "output": "dist"
    },
    "files": [ // 打包排除的文件
      "dist/electron/**/*",
      "node_modules/",
      "package.json"
    ],
    "nsis": { // 生成安装包的相关配置
      "oneClick": false, // 是否一键安装
      "allowElevation": true, // 允许请求提升。 如果为false，则用户必须使用提升的权限重新启动安装程序。
      "allowToChangeInstallationDirectory": true, // 允许修改安装目录
      "installerIcon": "build/icons/icon.ico", // 安装图标
      "uninstallerIcon": "build/icons/icon.ico", //卸载图标
      "installerHeaderIcon": "build/icons/icon.ico", // 安装时头部图标
      "createDesktopShortcut": true, // 创建桌面图标
      "createStartMenuShortcut": true, // 创建开始菜单图标
      "shortcutName": "HDcalculator" // 图标名称
    },
    "dmg": {
      "contents": [
        {
          "x": 410,
          "y": 150,
          "type": "link",
          "path": "/Applications"
        },
        {
          "x": 130,
          "y": 150,
          "type": "file"
        }
      ]
    },
    "mac": {
      "icon": "build/icons/icon.icns" // mac下的图标
    },
    "win": {
      "icon": "build/icons/icon.ico",
      "target": [
        {
          "target": "nsis", // 打包对象
          "arch": [
            "ia32"
          ]
        }
      ]
    },
    "linux": {
      "icon": "build/icons" // linux下的图标
    }
  }
```

**script设置：**

```
"build:package": "node .electron-vue/build-builder.js && electron-builder --win", // 指定打包windows环境 前面那一段很重要  不加的话会出现打包app打开白屏的情况
```

### Electron常用api

[官方文档](https://www.electronjs.org/zh/docs/latest/)
[API 手册项目](https://github.com/demopark/electron-api-demos-Zh_CN)

#### 主进程

1. 应用程序进程

```js
const { BrowserWindow } = require("electron");
const win = new BrowserWindow();

win.setProgressBar(0.5);
```

2. 任务栏弹出列表

```js
const { app } = require("electron");
app.setUserTasks([
  {
    program: process.execPath,
    arguments: "--new-window",
    iconPath: process.execPath,
    iconIndex: 0,
    title: "New Window",
    description: "Create a new window",
  },
]);
```

3. 任务栏缩略图工具栏

```js
const { BrowserWindow } = require('electron')
const path = require('path')

const win = new BrowserWindow()

win.setThumbarButtons([
  {
    tooltip: 'button1',
    icon: path.join(__dirname, 'button1.png'),
    click () { console.log('button1 clicked') }
  }, {
    tooltip: 'button2',
    icon: path.join(__dirname, 'button2.png'),
    flags: ['enabled', 'dismissonclick'],
    click () { console.log('button2 clicked.') }
  }
])
```

**注意:** 调用`BrowserWindow.setThumbarButtons`并传入空数组即可清空缩略图工具栏：

```js
const { BrowserWindow } = require('electron')

const win = new BrowserWindow()
win.setThumbarButtons([])
```

4. 本地快捷键：我们可以使用`menu`模块来配置快捷键，但是只有当`app`在处于焦点时才能够进行触发，在创建`MenuItem`时必须指定一个`accelerator`属性。

```js
const { Menu, MenuItem } = require('electron')
const menu = new Menu()

menu.append(new MenuItem({
  label: 'Print',
  accelerator: 'CmdOrCtrl+P',
  click: () => { console.log('time to print stuff') }
}))
```

可以在操作系统中配置不同的组合键

```js
menu.append(new MenuItem({
  label: 'Print',
  accelerator: process.platform === 'darwin' ? 'Alt+Cmd+I' : 'Ctrl+Shift+I'
  click: () => { console.log('time to print stuff') }
}))
```

5. 全局快捷键：当应用程序不处于焦点状态时，你可以使用`globalShortcut`模块来检测键盘事件

```js
const { app, globalShortcut } = require("electron");

app.on("ready", () => {
  globalShortcut.register("CommandOrControl+X", () => {
    console.log("CommandOrControl+X is pressed");
  });
});
```

#### 渲染进程

1. 通知（Notification）

```js
let myNotification = new Notification("标题", {
  body: "通知正文内容",
}); // 声明并显示一个通知

myNotification.onclick = () => {
  console.log("通知被点击");
}; // 通知被点击的时候执行
```

### 遇到的问题

#### electron拖拽问题

`electron`存在和浏览器一样拖入文件直接打开的问题
`electron3.0`之下版本需要手动禁用浏览器相关默认行为

```js
document.addEventListener("dragenter", function( event ) {
  ...
}, false);
document.addEventListener("dragover", function( event ) {
  ...
}, false);
document.addEventListener("drop", function( event ) {
  event.preventDefault();//禁止浏览器默认行为
  ...
  return false;//禁止浏览器默认行为
}, false);
document.addEventListener("dragend", function( event ) {
  ...
}, false);
```

或者通过定义`mixins`解决：

```js
export default {
  mounted() {
    this.disableDragEvent();
  },
  methods: {
    disableDragEvent() {
      window.addEventListener("dragenter", this.disableDrag, false);
      window.addEventListener("dragover", this.disableDrag);
      window.addEventListener("drop", this.disableDrag);
    },
    disableDrag(e) {
      const dropzone = document.getElementById("upload-area"); // 这个是可拖拽的上传区
      if (dropzone === null || !dropzone.contains(e.target)) {
        e.preventDefault();
        e.dataTransfer.effectAllowed = "none";
        e.dataTransfer.dropEffect = "none";
      }
    },
  },
  beforeDestroy() {
    window.removeEventListener("dragenter", this.disableDrag, false);
    window.removeEventListener("dragover", this.disableDrag);
    window.removeEventListener("drop", this.disableDrag);
  },
};
```

#### 使窗口具备桌面端拖拽移动位置的功能

```css
-webkit-app-region: drag;
```

**注意：**这样设置会导致区域的点击事件失效，解决办法：在需要点击的区域定义

```css
-webkit-app-region: no-drag;
```

或者自己编写`mouse`事件

#### electron-vue中使用element-ui部分组件无法渲染

[参考](https://blog.csdn.net/yuqiuyao/article/details/91980109)

常见的一个`element-table`表格组件和`button`，但是`table`组件并没有渲染，`electron-vue`文档上提出使用`.electron-vue/webpack.renderer.config.js`文件进行处理。
白名单里的外部组件：
我们可以将特定的模块列入白名单，而不是将它视为`webpack`的`externals`，大多数情况下，我们不需要使用这个功能，但是某些情况下，比如提供了原始`.vue`组件的`vueui`库，我们需要将他们列入白名单，以至于`vue-loader`能够进行编译，另一个情况是使用`webpack`的`alias`。

```js
let whiteListedModules = ['vue', 'element-ui', 'vue-router', 'axios', 'vuex', 'vue-electron']
// 白名单内的内容才会被打包进程序，否则可能出现找不到依赖的情况
```

#### dialog(打开文件或者保存文件)打开后禁用其他位置点击事件(类模态窗效果)

```
dialog-showOpenDialog/showSaveDialog 第一个参数为browserWindow,该参数指定选定窗口，无法使用主进程export方式获取，也无法通过赋值方法保存

获取 > BrowserWindow.getFocusedWindow()
```