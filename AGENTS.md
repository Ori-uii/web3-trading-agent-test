# AGENTS.md

这份文件用于帮助之后接手本项目的开发者或 AI 助手快速了解项目结构、运行方式、关键实现和协作注意事项。

## 项目概览

- 项目类型：Vue 3 + Vite 单页应用。
- 项目主题：Web3 金融交易 Agent 首页，品牌名为 `NeuroTrade`。
- 主要视觉：深色科技风首页、视频背景、滚动后视频缩小到第二屏、黑色胶囊导航栏、合作伙伴滚动条、最近更新轮播区。
- 当前没有接入真实接口，也没有真实交易逻辑，页面内容主要是静态展示和前端交互效果。

## 启动与构建

本项目使用 npm 管理依赖，Windows PowerShell 下建议使用 `npm.cmd`，避免 `npm.ps1` 执行策略问题。

```powershell
cd "F:\A codex\test"
npm.cmd install
npm.cmd run dev
npm.cmd run build
```

- `npm.cmd install`：安装依赖，生成或更新 `node_modules/`。
- `npm.cmd run dev`：启动 Vite 开发服务器，通常访问 `http://localhost:5173/`。
- `npm.cmd run build`：执行生产构建检查，输出目录是 `dist/`。

## 重要文件

```text
index.html
src/main.js
src/App.vue
src/style.css
src/components/PriceCard.vue
public/
package.json
package-lock.json
vite.config.js
.gitignore
```

### `index.html`

Vite 的 HTML 入口文件，只负责提供 Vue 挂载点：

```html
<div id="app"></div>
```

页面真正的内容由 `src/main.js` 挂载的 `App.vue` 渲染。

### `src/main.js`

Vue 应用入口：

- 引入 `App.vue`。
- 引入全局样式 `src/style.css`。
- 使用 `createApp(App).mount("#app")` 把 Vue 页面挂载到 `index.html` 的 `#app` 上。

### `src/App.vue`

当前主页面的核心文件，包含大部分页面结构、数据和交互逻辑。

主要内容：

- 顶部导航数据：`navLinks`。
- Resources 下拉预览数据：`resourcePreviewItems`。
- 合作伙伴滚动数据：`partners` 和 `scrollingPartners`。
- 最近更新轮播数据：`latestUpdates`。
- 首页视频缩放样式计算：`heroVideoStyle`。
- 首页内容淡出样式：`heroContentStyle`。
- 背景遮罩透明度：`shadeStyle`。
- 最近更新卡片轮播：`activeUpdateIndex`、`startLatestCarousel()`、`showLatestUpdate()`。
- 未展开卡片的鼠标跟随箭头：`railCursorVisible`、`railCursorDirection`。
- Resources 下拉框显示与关闭：`resourceMenuOpen`、`openResourceMenu()`、`closeResourceMenuLater()`。
- 首页向第二屏滚动吸附：`snapToWelcomeOnWheel()`。

如果要修改页面文案，优先看 `App.vue` 顶部的数据数组。

### `src/style.css`

当前项目几乎所有样式都在这里，包括：

- 全局字体、背景、盒模型。
- 首页视频、遮罩、大标题、合作伙伴滚动条。
- 固定导航栏和滚动后的黑色胶囊状态。
- Resources 下拉预览框。
- 第二屏欢迎区域。
- 第三屏最近更新轮播卡片。
- 响应式布局。
- 5 秒环形计时动画。

如果只调整页面外观，大多数情况下只需要修改 `src/style.css`。

### `src/components/PriceCard.vue`

这是之前创建的示例组件，用于演示 Vue 组件和 props 的写法。

目前主页面没有使用这个组件。除非后续重新加入行情卡片，否则不要因为它未使用就贸然删除。

## 视频资源规则

当前页面视频写法在 `src/App.vue`：

```html
<source src="/hero-video.mp4" type="video/mp4" />
```

这表示 Vite 会从 `public/hero-video.mp4` 读取视频。

注意：

- `public/` 里的文件会以网站根路径访问，例如 `/hero-video.mp4`。
- 视频文件通常很大，不建议提交到 GitHub。
- `.gitignore` 已经排除了 `*.mp4`、`*.MP4`、`*.mov`、`*.MOV`。
- 如果要更换视频，推荐把新视频命名为 `hero-video.mp4` 放进 `public/`，这样不用改代码。

## 关键交互说明

### 首页视频缩小

视频初始覆盖首页。向下滚动时，`heroVideoStyle` 会根据 `scrollProgress` 动态计算视频大小。

滚动到第二屏后，视频会固定在第二屏的中间文档位置，而不是继续跟着视口滚到第三屏。

### Resources 下拉框

Resources 菜单使用 `resourceMenuOpen` 控制显示。

下拉框相关 CSS 主要在：

```css
.resource-popover
.resource-popover::before
.is-scrolled .resource-popover
.is-scrolled .resource-popover::before
```

其中：

- `.resource-popover` 是整个可 hover 区域，包含透明区域和可见黑色面板。
- `.resource-popover::before` 是真正可见的黑色背景。
- `padding-top`、`min-height`、`::before top` 会影响导航和下拉框之间透明 hover 区域的大小。
- `closeResourceMenuLater()` 里当前关闭延迟是 `60ms`。

### 最近更新轮播

第三屏最近更新区使用 3 张卡片：

- 同一时间只有一张卡片展开。
- 未展开卡片缩小成浅灰色竖条。
- 点击未展开卡片可以展开。
- 每 5 秒自动切换下一张。
- 当前展开卡片右上角有一圈 5 秒计时动画。
- 鼠标放在未展开卡片上，会显示跟随鼠标移动的方向箭头。

## 样式修改建议

- 页面整体偏现代 Web3 金融项目主页，建议保持克制、干净、科技感。
- 不要随意加入框架或 UI 组件库，目前项目刻意保持轻量。
- 颜色尽量保持黑、白、灰、少量蓝紫光效的方向。
- 修改布局时同时检查桌面端和移动端媒体查询。
- 不要让文字溢出按钮、导航、卡片或小屏幕容器。

## Git 与文件排除

当前 `.gitignore` 已排除：

- `node_modules/`
- `dist/`
- `.env` 和 `.env.*`
- 日志文件
- 视频文件
- 系统和编辑器临时文件

提交代码前建议执行：

```powershell
git status --short --branch
npm.cmd run build
```

只提交源码和项目配置，不提交 `node_modules/`、`dist/`、视频文件或本地环境文件。

## 给后续开发者的注意事项

- 如果需要改页面内容，先看 `src/App.vue` 顶部的数据数组。
- 如果需要改视觉，优先看 `src/style.css`。
- 如果需要改视频，优先替换 `public/hero-video.mp4`，不要把大视频提交进 Git。
- 如果遇到 PowerShell 显示中文乱码，优先使用 `Get-Content -Encoding UTF8` 查看文件。
- 如果需要验证页面，先运行 `npm.cmd run dev`，再访问 Vite 输出的本地地址。
- 如果需要验证构建，运行 `npm.cmd run build`。

