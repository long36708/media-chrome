---
name: media-chrome
description: Media Chrome 是一个基于 Web Components 的媒体播放器控件库。此技能用于集成、配置和使用 media-chrome 组件，创建自定义视频/音频播放器，支持原生 HTML5 video/audio 以及 HLS、DASH、YouTube、Vimeo 等多种媒体源，并与 React、Vue、Angular 等框架兼容。当用户需要使用或扩展 media-chrome 功能时使用此技能。
---

# Media Chrome 集成与使用技能

## 概述

Media Chrome 提供了一套可自定义的 Web Components 媒体播放器控件，可以轻松集成到任何项目中。本技能涵盖组件的安装、基本使用、高级配置、自定义组件开发以及在不同框架中的集成方式。

## 安装与引入

### CDN 方式（快速开始）

```html
<!-- ES Modules -->
<script type="module" src="https://cdn.jsdelivr.net/npm/media-chrome@4/+esm"></script>

<!-- 或者使用 IIFE 版本 -->
<script src="https://cdn.jsdelivr.net/npm/media-chrome@4/dist/iife/index.js"></script>
```

### NPM 包方式

```bash
npm install media-chrome
```

```javascript
// ES Modules
import 'media-chrome';

// 或者按需导入
import { MediaController, MediaPlayButton } from 'media-chrome';
```

### 项目构建方式（开发此项目）

```bash
# 安装依赖
npm install

# 开发模式（带热重载）
npm run dev

# 构建
npm run build

# 运行测试
npm test
```

## 基础使用

### 视频播放器基础结构

使用 `<media-controller>` 作为容器，`slot="media"` 放置媒体元素：

```html
<media-controller>
  <video
    slot="media"
    src="https://example.com/video.mp4"
    playsinline
    crossorigin
  ></video>
  <media-control-bar>
    <media-play-button></media-play-button>
    <media-mute-button></media-mute-button>
    <media-volume-range></media-volume-range>
    <media-time-range></media-time-range>
    <media-fullscreen-button></media-fullscreen-button>
  </media-control-bar>
</media-controller>
```

### 音频播放器基础结构

添加 `audio` 属性到 `<media-controller>`：

```html
<media-controller audio>
  <audio
    slot="media"
    src="https://example.com/audio.mp3"
  ></audio>
  <media-control-bar>
    <media-play-button></media-play-button>
    <media-time-display showduration></media-time-display>
    <media-time-range></media-time-range>
    <media-playback-rate-button></media-playback-rate-button>
    <media-mute-button></media-mute-button>
    <media-volume-range></media-volume-range>
  </media-control-bar>
</media-controller>
```

## 核心组件

### MediaController 基础属性

| 属性 | 说明 | 示例值 |
|------|------|--------|
| `audio` | 音频模式，移除视频特定样式 | `audio` |
| `defaultsubtitles` | 默认启用字幕 | `defaultsubtitles` |
| `hotkeys` | 启用键盘快捷键（默认启用） | `hotkeys` |
| `nohotkeys` | 禁用键盘快捷键 | `nohotkeys` |
| `keyboardforwardseekoffset` | 前进跳转秒数 | `"15"` |
| `keyboardbackwardseekoffset` | 后退跳转秒数 | `"5"` |
| `defaultstreamtype` | 默认流类型 | `"on-demand"` 或 `"live"` |

### 常用控制组件

- `<media-play-button>` - 播放/暂停按钮
- `<media-mute-button>` - 静音按钮
- `<media-volume-range>` - 音量滑块
- `<media-time-range>` - 进度条（支持缩略图预览）
- `<media-time-display>` - 当前时间显示
- `<media-duration-display>` - 总时长显示
- `<media-fullscreen-button>` - 全屏按钮
- `<media-pip-button>` - 画中画按钮
- `<media-airplay-button>` - AirPlay 按钮
- `<media-cast-button>` - 投屏按钮
- `<media-captions-button>` - 字幕按钮
- `<media-playback-rate-button>` - 倍速按钮
- `<media-poster-image>` - 海报图片
- `<media-loading-indicator>` - 加载指示器

### MediaController Slots

| Slot 名称 | 说明 |
|-----------|------|
| `media` | 媒体元素（video/audio 或自定义媒体元素） |
| `poster` | 海报图片 |
| `centered-chrome` | 居中的控制元素（如加载指示器） |
| `top-chrome` | 顶部控制栏 |
| `bottom-chrome` | 底部控制栏 |

## 媒体源支持

### HTML5 Video/Audio

```html
<video slot="media" src="video.mp4" playsinline crossorigin>
  <track kind="captions" label="中文" src="captions.vtt" srclang="zh" default>
</video>
```

### HLS 流媒体

需要先加载 `hls-video-element`：

```html
<script type="module" src="https://cdn.jsdelivr.net/npm/hls-video-element@1.1/+esm"></script>

<media-controller>
  <hls-video
    slot="media"
    src="https://example.com/stream.m3u8"
    stream-type="on-demand"
    preload="metadata"
    playsinline
    crossorigin
  ></hls-video>
  <!-- 控制组件 -->
</media-controller>
```

### YouTube

```html
<script type="module" src="https://cdn.jsdelivr.net/npm/youtube-video-element@1.1/+esm"></script>

<media-controller>
  <youtube-video
    slot="media"
    video-id="VIDEO_ID"
    playsinline
  ></youtube-video>
  <!-- 控制组件 -->
</media-controller>
```

### 其他媒体源

参考 `examples/media-elements/` 目录下的示例：
- DASH (`dash-video-element`)
- Vimeo (`vimeo-video-element`)
- Video.js
- JW Player
- Mux Video

## 样式配置

### 基础样式

```css
/* 防止未定义的元素显示 */
:not(:defined) {
  display: none;
}

/* 防止布局偏移（CLS） */
media-controller:not([audio]) {
  display: block;
  max-width: 960px;
  aspect-ratio: 16 / 9;
}

video {
  width: 100%;
}
```

### CSS 变量

Media Chrome 组件支持多种 CSS 变量进行样式自定义：

```css
media-controller {
  --media-control-background: rgba(0, 0, 0, 0.7);
  --media-control-hover-background: rgba(0, 0, 0, 0.9);
  --media-primary-color: #3b82f6;
  --media-text-color: #ffffff;
}
```

### 组件特定样式

```css
/* 播放按钮 */
media-play-button {
  --media-play-button-display: flex;
  --media-play-button-width: 40px;
  --media-play-button-height: 40px;
}

/* 进度条 */
media-time-range {
  --media-time-range-height: 6px;
}
```

查看 `references/css-variables.md` 获取完整的 CSS 变量列表。

## 框架集成

### React 集成

使用官方 React 包装器：

```bash
npm install media-chrome
```

```tsx
import { MediaController, MediaPlayButton, MediaMuteButton } from 'media-chrome/react';

function VideoPlayer() {
  return (
    <MediaController>
      <video slot="media" src="video.mp4" playsinline />
      <MediaPlayButton />
      <MediaMuteButton />
    </MediaController>
  );
}
```

或直接使用 Web Components：

```tsx
import { useEffect, useRef } from 'react';

function VideoPlayer() {
  const videoRef = useRef<HTMLVideoElement>(null);

  useEffect(() => {
    import('media-chrome');
  }, []);

  return (
    <media-controller>
      <video ref={videoRef} slot="media" src="video.mp4" playsinline />
      <media-play-button />
      <media-mute-button />
    </media-controller>
  );
}
```

### Vue 集成

```vue
<script setup>
import { onMounted, ref } from 'vue';

onMounted(() => {
  import('media-chrome');
});
</script>

<template>
  <media-controller>
    <video slot="media" src="video.mp4" playsinline />
    <media-play-button />
    <media-mute-button />
  </media-controller>
</template>
```

### Angular 集成

```typescript
import { Component, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-video-player',
  template: `
    <media-controller>
      <video slot="media" src="video.mp4" playsinline></video>
      <media-play-button></media-play-button>
      <media-mute-button></media-mute-button>
    </media-controller>
  `
})
export class VideoPlayerComponent implements AfterViewInit {
  ngAfterViewInit() {
    import('media-chrome');
  }
}
```

## 高级功能

### 字幕菜单

需要导入菜单组件：

```html
<script type="module" src="media-chrome/dist/menu/index.js"></script>

<media-controller>
  <video slot="media" src="video.mp4">
    <track kind="captions" label="中文" src="zh.vtt" srclang="zh">
    <track kind="captions" label="English" src="en.vtt" srclang="en">
  </video>
  <media-control-bar>
    <media-play-button></media-play-button>
    <media-captions-button></media-captions-button>
    <media-fullscreen-button></media-fullscreen-button>
  </media-control-bar>
  <media-captions-menu anchor="auto" hidden></media-captions-menu>
</media-controller>
```

### 设置菜单（速度、画质、音频）

```html
<media-settings-menu anchor="auto" hidden>
  <media-settings-menu-item>
    Speed
    <media-playback-rate-menu slot="submenu" hidden>
      <div slot="title">Speed</div>
    </media-playback-rate-menu>
  </media-settings-menu-item>
  <media-settings-menu-item>
    Quality
    <media-rendition-menu slot="submenu" hidden>
      <div slot="title">Quality</div>
    </media-rendition-menu>
  </media-settings-menu-item>
</media-settings-menu>
```

### 缩略图预览

在 `media-time-range` 中添加缩略图：

```html
<video slot="media" src="video.mp4">
  <track kind="metadata" label="thumbnails" src="thumbnails.vtt" default>
</video>

<media-time-range>
  <media-preview-thumbnail slot="preview"></media-preview-thumbnail>
</media-time-range>
```

### 键盘快捷键

默认支持的快捷键：

- `Space/K` - 播放/暂停
- `M` - 静音
- `F` - 全屏
- `左右箭头` - 快进/快退 10 秒
- `上下箭头` - 调整音量
- `>` / `<` - 增加/减少播放速度

自定义快捷键：

```html
<media-controller keyboardforwardseekoffset="15" keyboardbackwardseekoffset="5">
  <!-- ... -->
</media-controller>
```

## 自定义媒体元素开发

Media Chrome 将与任何暴露与 HTML 媒体元素（`<video>` 和 `<audio>`）相同 API 的元素一起工作。这意味着你可以用自己的自定义元素替换这些元素，只要它们符合相同的 API。

了解 [HTMLMediaElement API on MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement)。

### Media Slot 工作原理

Media Controller 会自动：
- 监听 slot="media" 中元素发出的事件
- 使用已知的属性理解媒体元素的当前状态
- 调用媒体元素上的方法，如 `play()`、`pause()` 等

### 三种实现方式

| 方式 | 适用场景 | 复杂度 |
|------|---------|--------|
| **最小媒体元素** | 快速原型、测试、只需要部分功能 | ⭐ 简单 |
| **扩展原生元素** | 在原生基础上添加功能 | ⭐⭐ 中等 |
| **完全自定义实现** | 支持新的流媒体协议 | ⭐⭐⭐ 复杂 |

---

### 方式一：最小媒体元素

**关键要点**：不需要支持完整的 `HTMLMediaElement` API，只需实现组件所需的部分

例如，要使播放按钮工作，只需要：
- 提供 `play()` 和 `pause()` 方法
- 触发 `play`、`playing` 和 `pause` 事件
- 提供 `paused` getter

---

### 方式二：扩展原生元素

使用 [custom-media-element](https://github.com/muxinc/custom-media-element) 库，在原生 `<video>` 或 `<audio>` 元素上添加功能。

**优点**：
- ✅ 自动继承完整的 `HTMLVideoElement` API
- ✅ 保持与原生元素相同的事件行为
- ✅ 简化实现，只需关注扩展功能
- ✅ 支持所有 Media Chrome 控制组件

---

### 方式三：完全自定义实现

当需要完全自定义媒体引擎（如支持新的流媒体协议）时，必须满足完整的 HTMLMediaElement 接口要求。

### 完整要求快速参考

| 类别 | 必需项 |
|------|--------|
| **方法** | `play()`、`addEventListener()`、`removeEventListener()` |
| **属性** | `paused`、`duration`、`currentTime`、`volume`、`muted`、`playbackRate`、`readyState`、`error` |
| **事件** | `play`、`pause`、`playing`、`timeupdate`、`durationchange`、`ended`、`error` 等 |
| **轨道** | `textTracks`、`addtrack`、`removetrack`、`change` 事件 |

---

### 详细实现指南

**📖 所有实现方式的完整代码示例、详细说明和测试清单，请参考：**
`references/media-element-implementation.md`

该指南包含：
- 三种实现方式的完整代码示例（可运行）
- 完整的接口规范和类型定义
- 所有必需事件的详细说明和触发时机
- Text Tracks、Video Renditions、Audio Tracks 的实现示例
- 画中画、全屏、远程播放等高级功能的实现
- 实现要点和常见陷阱
- 测试 checklist

### 参考实现

#### 官方媒体元素

- **[hls-video-element](https://github.com/muxinc/hls-video-element)** - HLS 流媒体（使用 custom-media-element）
- **[youtube-video-element](https://github.com/muxinc/youtube-video-element)** - YouTube 视频（使用 custom-media-element）
- **[dash-video-element](https://github.com/muxinc/dash-video-element)** - DASH 流媒体（使用 custom-media-element）
- **[vimeo-video-element](https://github.com/muxinc/vimeo-video-element)** - Vimeo 视频（使用 custom-media-element）

#### 社区媒体元素

- **[jwplayer-video-element](https://github.com/muxinc/jwplayer-video-element)** - JW Player 集成
- **[videojs-video-element](https://github.com/muxinc/videojs-video-element)** - Video.js 集成
- **[mux-video-element](https://github.com/muxinc/mux-video-element)** - Mux Video 播放
- **[plyr-video-element](https://github.com/muxinc/plyr-video-element)** - Plyr 集成

#### 工具库

- **[custom-media-element](https://github.com/muxinc/custom-media-element)** - 扩展原生媒体元素的基础库

#### 项目源码

- `src/js/media-controller.ts` - MediaController 的媒体元素查找逻辑
- `src/js/media-store/state-mediator.ts` - 状态中介者，监听所有媒体事件

### 如何选择实现方式

| 需求 | 推荐方式 |
|------|---------|
| 快速原型/测试 | 最小媒体元素 |
| 在原生基础上扩展 | custom-media-element |
| 支持 HLS/DASH | 使用对应的官方元素 |
| 支持第三方播放器 | 使用对应的社区元素 |
| 全新流媒体协议 | 完全自定义实现

## 自定义组件开发

当需要创建自定义媒体控制组件时，参考项目源码中的现有组件实现：

1. 继承 `MediaChromeButton`、`MediaChromeRange` 等基类
2. 使用 `MediaUIEvents` 和 `MediaUIAttributes` 常量
3. 实现 `observedAttributes` 和相关 getter/setter
4. 使用 `associateElement(this)` 关联到 media-controller
5. 通过 CustomEvent 发送和接收媒体事件

参考文件：
- `src/js/media-play-button.ts` - 基础按钮组件
- `src/js/media-time-range.ts` - 进度条组件
- `src/js/constants.ts` - 事件和属性常量

示例结构：

```typescript
import { MediaChromeButton } from './media-chrome-button.js';
import { MediaUIEvents, MediaUIAttributes } from './constants.js';
import { globalThis } from './utils/server-safe-globals.js';

class MyCustomButton extends MediaChromeButton {
  handleClick(): void {
    const evt = new globalThis.CustomEvent(
      MediaUIEvents.MEDIA_PLAY_REQUEST,
      {
        composed: true,
        bubbles: true,
      }
    );
    this.dispatchEvent(evt);
  }
}

if (!globalThis.customElements.get('my-custom-button')) {
  globalThis.customElements.define('my-custom-button', MyCustomButton);
}
```

## 事件系统

### 媒体请求事件

组件通过派发事件与媒体元素通信：

- `media-play-request` - 请求播放
- `media-pause-request` - 请求暂停
- `media-mute-request` - 请求静音
- `media-unmute-request` - 请求取消静音
- `media-seek-request` - 请求跳转时间
- `media-fullscreen-request` - 请求全屏
- `media-airplay-request` - 请求 AirPlay
- `media-cast-request` - 请求投屏

### 媒体状态属性

通过 `MediaUIAttributes` 监听媒体状态：

- `media-paused` - 是否暂停
- `media-muted` - 是否静音
- `media-duration` - 总时长
- `media-current-time` - 当前时间
- `media-volume` - 音量
- `media-playing` - 是否播放中

## 项目构建与开发

**📝 注意**：本节内容仅适用于为 Media Chrome 项目贡献代码的开发者。如果只是使用 Media Chrome 库，可以跳过此部分。

**📖 完整的开发指南、构建脚本和测试指南，请参考：**
`references/project-development.md`

### 快速开始

```bash
# 安装依赖
npm install

# 监听模式开发
npm run dev

# 运行测试
npm test

# 完整构建
npm run build
```

### 开发指南包含：

- 详细的构建脚本说明
- 项目结构和目录说明
- 添加新组件的完整流程
- 组件开发最佳实践
- 测试指南和示例
- 调试技巧和常见问题
- 贡献指南

### 常用命令

| 命令 | 说明 |
|------|------|
| `npm run dev` | 开发模式（带热重载） |
| `npm test` | 运行测试 |
| `npm run build` | 完整构建 |
| `npm run lint` | Lint 检查 |
| `npm run format` | 格式化代码 |

## 参考资源

### 官方文档
- [Media Chrome 官方网站](https://media-chrome.org)
- [API 文档](https://media-chrome.org/docs)
- [组件文档](https://media-chrome.org/docs/en/components)

### 项目内参考文件

**详细指南：**
- `references/media-element-implementation.md` - 自定义媒体元素实现指南（完整代码示例）
- `references/project-development.md` - 项目开发指南（构建、贡献）
- `references/testing-guide.md` - 测试指南（测试框架、示例、最佳实践）

**示例和源码：**
- `references/css-variables.md` - CSS 变量完整列表
- `examples/vanilla/basic.html` - 基础示例
- `examples/vanilla/advanced.html` - 高级示例
- `examples/vanilla/mobile.html` - 移动端示例
- `src/js/constants.ts` - 常量定义

### 技术规格
- [Web Components 规范](https://web.dev/custom-elements/)
- [Shadow DOM](https://web.dev/shadowdom/)
- [Custom Elements](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements)

## 常见问题

### Q: 如何禁用某些控件？
A: 使用 CSS 隐藏不可用的控件：
```css
media-airplay-button[mediaairplayunavailable] {
  display: none;
}
```

### Q: 如何自定义图标？
A: 使用 slot 自定义图标：
```html
<media-play-button>
  <svg slot="icon" viewBox="0 0 24 24">
    <!-- 自定义图标路径 -->
  </svg>
</media-play-button>
```

### Q: 如何实现响应式布局？
A: 使用 CSS 容器查询或 breakpoint 属性：
```html
<media-control-bar>
  <media-play-button desktop></media-play-button>
  <media-play-button mobile hidden></media-play-button>
</media-control-bar>
```

### Q: 支持哪些浏览器？
A: 支持所有现代浏览器（Chrome、Firefox、Safari、Edge），需要 Custom Elements 和 Shadow DOM 支持。
