<img width="1920" height="991" alt="demo" src="https://github.com/user-attachments/assets/576e243c-af25-4530-b424-a0119b395330" />

# 漫画阅读器

一个简洁高效的 Web 漫画阅读器，单文件无需构建，直接在浏览器中打开即可使用。

## 技术栈

- **纯前端**：HTML + CSS + JavaScript，无任何外部依赖
- **File System Access API**：原生浏览器 API 实现文件夹选择
- **Canvas API**：缩略图动态生成与放大镜实现
- **CSS Flexbox/Grid**：响应式布局

## 核心技术

### 1. 文件系统访问

使用 `showDirectoryPicker()` API 获取用户授权，直接读取本地文件夹：

```javascript
const dir = await window.showDirectoryPicker();
for await (const e of dir.values()) {
    if (e.kind === 'file') entries.push(e);
}
```

### 2. 智能页码解析

通过正则表达式解析文件名，提取页码和特殊标记：

```javascript
function parseNum(name) {
    const m = name.match(/(\d+)([a-zA-Z])?(?:-(\d+))?\.jpe?g$/i);
    // 返回: { p: 页码, s: 字母后缀, r: 结束页码(合图), sp: 是否合图 }
}
```

支持格式：
- `railgan_01_001.jpg` → 第1页
- `railgan_01_999a.jpg` → 封面/封底
- `railgan_01_006-007.jpg` → 双页合图

### 3. 页面分组算法

将相邻两页配对为阅读组，特殊合图（如 006-007.jpg）单独处理：

```javascript
function makeGroups(files) {
    for (let i = 0; i < files.length; i++) {
        if (pg.sp) {
            g.push({ idx: [i], label: pg.p + '-' + pg.r, spread: true });
        } else {
            g.push({ idx: [i, i+1], label: (i+1) + '-' + (i+2), spread: false });
            i++;
        }
    }
}
```

### 4. 放大镜实现

通过 Canvas 的 `drawImage()` 截取原图局部，放大后显示：

```javascript
function moveZoom(e) {
    const rx = (e.clientX - rect.left) / rect.width;  // 鼠标在图片的相对位置
    const ry = (e.clientY - rect.top) / rect.height;
    const scaledW = img.naturalWidth * S.zoom;        // 放大后的尺寸
    const offsetX = rx * scaledW - half;             // 计算偏移量
    E.lensImg.style.left = -offsetX + 'px';          // 负偏移实现截取
}
```

### 5. 日漫排版

双页模式使用 `flex-direction: row-reverse` 实现从右到左的阅读顺序：

```css
.reader.s2 .pages { flex-direction: row-reverse; }
```

## 操作指南

### 启动

1. 双击打开 `index.html`
2. 点击「选择文件夹」按钮
3. 选择漫画图片所在文件夹

### 快捷键

| 按键 | 功能 |
|------|------|
| `←` / `X` | 上一页 |
| `→` / `C` | 下一页 |
| `A` | 打开/关闭目录 |
| `Z` | 按住放大镜模式，滚轮调整放大倍数 |
| `Home` | 跳转到首页 |
| `End` | 跳转到末页 |
| `ESC` | 关闭目录 |

### 目录功能

- 点击目录中的缩略图可跳转对应页面
- 鼠标悬停缩略图超过1秒，显示页面预览
- 跳转后目录保持显示，再次点击目录后隐去

### 放大镜

- 按住 `Z` 键激活放大镜
- 鼠标在页面上移动，放大镜跟随
- 滚轮上下滚动调整放大倍数（1.5x ~ 6x）
- 右侧滑块可调整放大镜直径（150px ~ 500px）

### 阅读模式

- **单页模式**：一次显示一页
- **双页模式**（默认）：一次显示两页，日漫从右到左排版

### 浏览器兼容性

需要支持 File System Access API 的浏览器：
- Chrome 86+
- Edge 86+
- Opera 72+

不支持 Firefox、Safari。

---

## AI 生成提示词

如果让我用 AI 来生成这个项目，我会使用以下提示词：

```
创建一个单文件 Web 漫画阅读器应用，要求：

## 基本功能
- 使用 File System Access API 实现文件夹选择，用户选择一个漫画文件夹后自动加载其中所有 jpg/jpeg/png 图片
- 图片按文件名中的数字排序（如 railgan_01_001.jpg → 第1页）
- 支持单页和双页两种阅读模式，双页模式为日漫从右到左排版

## 文件名解析规则
- 普通文件：railgan_01_001.jpg → 第1页
- 带字母后缀：railgan_01_999a.jpg → 封面/封底（排在最后）
- 双页合图：railgan_01_006-007.jpg → 合图单独一组，不参与两页配对

## 页面分组逻辑
- 正常情况下每两页配对为一组显示
- 合图（文件名含 - 如 006-007.jpg）单独一组
- 第一页如果在双数位置（如第2、4、6页），则单独成组

## 导航功能
- 左侧目录面板显示所有页面的缩略图（显示组标签而非单页）
- 缩略图点击跳转到对应页面
- 键盘快捷键：←/X 上一页，→/C 下一页，A 目录，Home/End 首末页
- 点击屏幕左右边缘翻页

## 放大镜功能
- 按住 Z 键激活放大镜，鼠标位置居中放大显示
- 滚轮调整放大倍数（1.5x ~ 6x）
- 右侧滑块调整放大镜直径（150px ~ 500px）

## UI 要求
- 暗色主题，简洁现代
- 右上角显示快捷键说明和放大镜设置
- 目录悬停1秒显示该页预览大图
- 通过目录跳转后目录保持显示，鼠标离开目录后才自动关闭

## 技术限制
- 纯 HTML + CSS + JavaScript，无外部依赖
- 单文件实现
- 使用 Canvas 动态生成缩略图
- 所有图片懒加载节省内存
```
