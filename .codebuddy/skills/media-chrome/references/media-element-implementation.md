# 自定义媒体元素实现指南

本文档详细说明如何实现一个与 Media Chrome 兼容的自定义媒体元素。

## 概述

Media Chrome 将与任何暴露与 HTML 媒体元素（`<video>` 和 `<audio>`）相同 API 的元素一起工作。这意味着你可以用自己的自定义元素替换这些元素，只要它们符合相同的 API。

了解更多关于 [HTMLMediaElement API on MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement)。

## Media Slot 工作原理

最简单的 media slot 是浏览器原生的 `<video>` 标签：

```html
<media-controller>
  <video slot="media" src="https://....mp4" >
</media-controller>
```

在这个例子中，Media Controller 将：

* 监听 `<video>` 元素发出的事件
* 使用已知的属性理解 `<video>` 元素的当前状态
* 调用 `<video>` 元素上的方法，如 `play()`、`pause()` 等

## 最小媒体元素（Minimal Media Element）

### 概念

**重要**：slot 中的媒体元素**不需要**支持完整的 `HTMLMediaElement` API。每个 Media Chrome 组件需要不同的 API，例如要使播放按钮工作，媒体元素的要求可能只需要：

- 提供 `play()` 和 `pause()` 方法
- 触发 `play`、`playing` 和 `pause` 事件
- 提供 `paused` getter

### 最小实现示例

这是一个只有播放/暂停功能的最小自定义媒体元素：

```html
<script>
  class PlayPauseElement extends HTMLElement {
    constructor() {
      super();

      this.attachShadow({ mode: 'open' });
      this.shadowRoot.innerHTML = `
        <style>
          :host {
            pointer-events: none;
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            background: #333;
          }

          :host::before {
            content: 'Paused';
          }

          :host([unpaused])::before {
            content: 'Playing';
          }
        </style>
      `;
    }

    get paused() {
      return !this.hasAttribute('unpaused');
    }

    play() {
      this.toggleAttribute('unpaused', true);
      this.dispatchEvent(new Event('play'));
      this.dispatchEvent(new Event('playing'));
      return Promise.resolve();
    }

    pause() {
      this.toggleAttribute('unpaused', false);
      this.dispatchEvent(new Event('pause'));
    }
  }

  if (!customElements.get('play-pause')) {
    customElements.define('play-pause', PlayPauseElement);
  }
</script>

<media-controller>
  <play-pause slot="media"></play-pause>
  <media-control-bar>
    <media-play-button></media-play-button>
  </media-control-bar>
</media-controller>
```

### 使用场景

最小媒体元素适用于：

- 快速原型开发
- 测试 Media Chrome 控制组件
- 只需要部分播放功能的场景
- 作为理解 Media Chrome 工作原理的起点

## 扩展原生媒体元素

### 使用 custom-media-element 库

如果你想在原生的 `<video>` 或 `<audio>` 元素上添加一些额外行为，可以使用 [custom-media-element](https://github.com/muxinc/custom-media-element) 库。

`HTMLVideoElement` API 会自动添加到任何从 `CustomVideoElement` 继承的自定义元素中。

### 安装

```bash
npm install custom-media-element
```

### 基本用法

```javascript
import { CustomVideoElement } from 'custom-media-element';

class MyCustomVideoElement extends CustomVideoElement {
  constructor() {
    super();
  }

  // 重写 play 方法
  play() {
    console.log('Custom play method called');
    return super.play();
  }

  // 重写 src getter & setter
  get src() {
    return super.src;
  }

  set src(src) {
    console.log('Setting source to:', src);
    super.src = src;
  }
}

if (!customElements.get('my-custom-video')) {
  customElements.define('my-custom-video', MyCustomVideoElement);
}

export default MyCustomVideoElement;
```

### 在 HTML 中使用

```html
<media-controller>
  <my-custom-video
    slot="media"
    src="https://stream.mux.com/A3VXy02VoUinw01pwyomEO3bHnG4P32xzV7u1j1FSzjNg/low.mp4"
  ></my-custom-video>
  <media-control-bar>
    <media-play-button></media-play-button>
  </media-control-bar>
</media-controller>
```

### custom-media-element 提供的功能

- 自动继承所有 `HTMLVideoElement` 的方法和属性
- 保持与原生 video 元素相同的事件行为
- 简化自定义媒体元素的创建
- 支持所有 Media Chrome 控制组件

[hls-video-element](https://github.com/muxinc/hls-video-element) 是使用此库的一个很好的例子。

## 完整自定义媒体元素

当需要完全自定义媒体引擎时（例如支持新的流媒体协议），你需要从头实现。以下是需要满足的完整要求。

### 核心 HTMLMediaElement 接口要求

自定义媒体元素必须实现 `MediaStateOwner` 类型，包含以下必需方法：

```typescript
export type MediaStateOwner = Partial<HTMLVideoElement> &
  Pick<
    HTMLMediaElement,
    'play' | 'paused' | 'addEventListener' | 'removeEventListener'
  > & {
    // 扩展属性...
  };
```

### 必需方法

- `play()` - 播放媒体（应返回 Promise）
- `addEventListener()` / `removeEventListener()` - 监听事件

### 必需属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `paused` | boolean | 暂停状态 |
| `duration` | number | 总时长（秒） |
| `currentTime` | number | 当前播放时间（秒） |
| `volume` | number | 音量（0-1） |
| `muted` | boolean | 是否静音 |
| `playbackRate` | number | 播放速率 |
| `loop` | boolean | 是否循环播放 |
| `readyState` | number | 就绪状态（0-4） |
| `error` | MediaError \| null | 错误对象 |

### readyState 常量

```typescript
const HAVE_NOTHING = 0;
const HAVE_METADATA = 1;
const HAVE_CURRENT_DATA = 2;
const HAVE_FUTURE_DATA = 3;
const HAVE_ENOUGH_DATA = 4;
```

## 必须触发的标准媒体事件

`state-mediator.ts` 中定义的关键事件，状态变更必须在相关事件**之前**触发：

| 事件类型 | 用途 | 触发时机 |
|---------|------|---------|
| `play` | 播放开始 | 调用 `play()` 方法且媒体开始播放时 |
| `pause` | 暂停 | 调用 `pause()` 方法时 |
| `playing` | 正在播放 | 媒体实际开始播放时 |
| `emptied` | 媒体清空 | 媒体加载被中断时 |
| `loadedmetadata` | 元数据加载完成 | 获取到 duration、videoWidth 等元数据时 |
| `timeupdate` | 播放时间更新 | currentTime 变化时（频繁触发） |
| `durationchange` | 时长变化 | duration 属性变化时 |
| `volumechange` | 音量变化 | volume 或 muted 属性变化时 |
| `ratechange` | 播放速率变化 | playbackRate 属性变化时 |
| `waiting` | 等待数据 | 缓冲不足需要等待时 |
| `seeked` | 跳转完成 | seek 操作完成时 |
| `seeking` | 正在跳转 | seek 操作开始时 |
| `ended` | 播放结束 | 媒体播放到末尾时 |
| `error` | 错误发生 | 发生播放错误时 |
| `resize` | 视频尺寸变化 | videoWidth 或 videoHeight 变化时 |
| `progress` | 缓冲进度 | buffered 范围变化时 |
| `canplay` | 可以播放 | 有足够数据开始播放时 |
| `canplaythrough` | 可以流畅播放 | 缓冲足够可以流畅播放到结束时 |

### 事件触发示例

```typescript
// 正确：先更新状态，再触发事件
this.paused = false;
this.dispatchEvent(new Event('play'));
setTimeout(() => {
  this.dispatchEvent(new Event('playing'));
}, 0);

// 错误：先触发事件再更新状态会导致监听器获取到旧状态
```

## Text Tracks 支持（必需）

### textTracks 属性

```typescript
// 必须支持 textTracks 属性
mediaElement.textTracks: TextTrackList
```

### 必须触发的轨道事件

- `addtrack` - 添加轨道时
- `removetrack` - 移除轨道时
- `change` - 轨道状态变化时

### Text Track 实现

```typescript
class MyMediaElement extends HTMLElement {
  private _textTracks: TextTrackList;

  constructor() {
    super();
    this._textTracks = [] as any; // 或使用真实的 TextTrackList
  }

  get textTracks(): TextTrackList {
    return this._textTracks;
  }

  addTrack(track: TextTrack) {
    this._textTracks.addTrack(track);
    this.dispatchEvent(new TrackEvent('addtrack', { track }));
  }

  removeTrack(track: TextTrack) {
    this._textTracks.removeTrack(track);
    this.dispatchEvent(new TrackEvent('removetrack', { track }));
  }
}
```

## Video Renditions 支持（推荐）

### videoRenditions 属性

```typescript
mediaElement.videoRenditions: VideoRenditionList
```

### 触发画质切换事件

- `addrendition` - 添加画质选项时
- `removerendition` - 移除画质选项时
- `change` - 画质变化时

### Video Rendition 实现

```typescript
interface VideoRendition {
  id: number;
  width: number;
  height: number;
  bandwidth: number;
  framerate: number;
}

class MyMediaElement extends HTMLElement {
  private _videoRenditions: VideoRenditionList;

  get videoRenditions(): VideoRenditionList {
    return this._videoRenditions;
  }

  addRendition(rendition: VideoRendition) {
    this._videoRenditions.addRendition(rendition);
    this.dispatchEvent(new Event('addrendition'));
  }
}
```

## Audio Tracks 支持（可选）

### audioTracks 属性

```typescript
mediaElement.audioTracks: AudioTrackList
```

### 触发音轨事件

- `addtrack` - 添加音轨时
- `removetrack` - 移除音轨时
- `change` - 音轨变化时

## 扩展属性（推荐）

### 流类型

```typescript
streamType?: 'live' | 'on-demand' | 'unknown';
```

用于区分直播流和点播内容，影响 UI 显示（如显示实时直播指示器）。

### 实时直播相关

```typescript
targetLiveWindow?: number;  // 直播延迟窗口（秒）
liveEdgeStart?: number;    // 直播边缘开始时间
```

### 远程播放（如 AirPlay、Cast）

```typescript
remote?: {
  state: string;           // 'disconnected' | 'connecting' | 'connected'
  prompt?: () => Promise<void>;  // 触发远程播放对话框
};
```

### WebKit 特定属性

```typescript
webkitDisplayingFullscreen?: boolean;
webkitPresentationMode?: 'fullscreen' | 'picture-in-picture' | 'inline';
webkitEnterFullscreen?: () => void;
webkitExitFullscreen?: () => void;
```

## 画中画支持（推荐）

```typescript
{
  requestPictureInPicture?: () => Promise<PictureInPictureWindow>;
  exitPictureInPicture?: () => Promise<void>;
  disablePictureInPicture?: boolean;
}
```

## 全屏支持（推荐）

### 标准 API

```typescript
{
  requestFullscreen?: () => Promise<void>;
  exitFullscreen?: () => Promise<void>;
  fullscreenElement?: Element | null;
  fullscreenEnabled?: boolean;
}
```

### WebKit API

```typescript
{
  webkitEnterFullscreen?: () => void;
  webkitExitFullscreen?: () => void;
  webkitDisplayingFullscreen?: boolean;
}
```

## Video 特定属性（推荐）

```typescript
{
  videoWidth: number;        // 视频宽度
  videoHeight: number;       // 视频高度
  poster: string;            // 海报图片 URL
  playsInline: boolean;      // 内联播放（iOS）
  crossOrigin: string;       // CORS 设置
}
```

## 可搜索/可跳转范围

```typescript
{
  seekable: TimeRanges;      // 可跳转的时间范围
  buffered: TimeRanges;       // 已缓冲的时间范围
  played: TimeRanges;        // 已播放的时间范围
}
```

## 完整实现示例

```typescript
class MyMediaElement extends HTMLElement {
  static get observedAttributes() {
    return ['src', 'autoplay', 'playsinline'];
  }

  // 必需属性
  paused = true;
  duration = NaN;
  currentTime = 0;
  volume = 1;
  muted = false;
  playbackRate = 1;
  loop = false;
  readyState = 0;
  error = null;

  // 推荐属性
  videoWidth = 0;
  videoHeight = 0;
  poster = '';
  playsInline = false;
  crossOrigin = '';
  autoplay = false;
  disablePictureInPicture = false;
  streamType: 'live' | 'on-demand' | 'unknown' = 'unknown';

  // TimeRanges
  seekable: TimeRanges = { length: 0, start: () => 0, end: () => 0 };
  buffered: TimeRanges = { length: 0, start: () => 0, end: () => 0 };
  played: TimeRanges = { length: 0, start: () => 0, end: () => 0 };

  // 轨道
  private _textTracks: TextTrackList;
  private _videoRenditions: VideoRenditionList;

  private _mediaEngine: any; // 实际的媒体引擎
  private _timeUpdateInterval: number | null = null;

  constructor() {
    super();
    this._textTracks = new TextTrackList() as any;
    this._videoRenditions = [] as any;
  }

  connectedCallback() {
    // 初始化媒体引擎
    this.loadMedia();
  }

  attributeChangedCallback(name: string, oldValue: string, newValue: string) {
    if (name === 'src' && oldValue !== newValue) {
      this.loadMedia();
    }
  }

  // 属性 getter/setter
  get src(): string {
    return this.getAttribute('src') || '';
  }

  set src(value: string) {
    if (value !== this.src) {
      this.setAttribute('src', value);
    }
  }

  get textTracks(): TextTrackList {
    return this._textTracks;
  }

  get videoRenditions(): VideoRenditionList {
    return this._videoRenditions;
  }

  // 必需方法
  async play(): Promise<void> {
    try {
      await this._mediaEngine.play();
      this.paused = false;
      this.dispatchEvent(new Event('play'));
      // playing 事件可以延迟触发
      requestAnimationFrame(() => {
        this.dispatchEvent(new Event('playing'));
      });
    } catch (error) {
      this.error = error as MediaError;
      this.dispatchEvent(new Event('error'));
      throw error;
    }
  }

  pause(): void {
    this._mediaEngine.pause();
    this.paused = true;
    this.dispatchEvent(new Event('pause'));
  }

  load(): void {
    // 重置状态
    this.paused = true;
    this.currentTime = 0;
    this.readyState = 0;
    this.duration = NaN;
    this.error = null;

    this.dispatchEvent(new Event('emptied'));
    this.loadMedia();
  }

  canPlayType(type: string): string {
    // 实现类型检测
    return 'maybe' | 'probably' | '';
  }

  // 事件监听器管理
  addEventListener(type: string, listener: any) {
    return super.addEventListener(type, listener);
  }

  removeEventListener(type: string, listener: any) {
    return super.removeEventListener(type, listener);
  }

  // 私有方法
  private loadMedia() {
    const src = this.src;
    if (!src) return;

    // 加载媒体逻辑
    this._mediaEngine = new YourMediaEngine(src);

    // 监听引擎事件
    this._mediaEngine.on('loadedmetadata', () => {
      this.duration = this._mediaEngine.duration;
      this.videoWidth = this._mediaEngine.videoWidth;
      this.videoHeight = this._mediaEngine.videoHeight;
      this.readyState = 1; // HAVE_METADATA
      this.dispatchEvent(new Event('loadedmetadata'));
      this.dispatchEvent(new Event('durationchange'));
    });

    this._mediaEngine.on('canplay', () => {
      this.readyState = 4; // HAVE_ENOUGH_DATA
      this.dispatchEvent(new Event('canplay'));
    });

    this._mediaEngine.on('timeupdate', () => {
      this.currentTime = this._mediaEngine.currentTime;
      this.dispatchEvent(new Event('timeupdate'));
    });

    this._mediaEngine.on('buffered', (ranges: TimeRanges) => {
      this.buffered = ranges;
      this.dispatchEvent(new Event('progress'));
    });

    this._mediaEngine.on('ended', () => {
      this.dispatchEvent(new Event('ended'));
      if (this.loop) {
        this.play();
      } else {
        this.paused = true;
      }
    });

    this._mediaEngine.on('error', (error: MediaError) => {
      this.error = error;
      this.dispatchEvent(new Event('error'));
    });
  }

  // 画中画
  async requestPictureInPicture(): Promise<PictureInPictureWindow> {
    if (typeof documentPictureInPicture !== 'undefined') {
      return documentPictureInPicture.requestWindow();
    }
    // 或者使用原生的 Picture-in-Picture API
    return (this as any).requestPictureInPicture();
  }

  // 全屏
  async requestFullscreen(): Promise<void> {
    return this.requestFullscreen();
  }
}

customElements.define('my-media-element', MyMediaElement);
```

## 关键实现要点

### 1. 事件顺序很重要

```typescript
// ✅ 正确：先更新状态，再触发事件
this.paused = false;
this.dispatchEvent(new Event('play'));

// ❌ 错误：先触发事件再更新状态
this.dispatchEvent(new Event('play'));
this.paused = false;
```

### 2. 属性可观察

使用 `observedAttributes` 用于响应属性变化：

```typescript
static get observedAttributes() {
  return ['src', 'autoplay', 'playsinline'];
}

attributeChangedCallback(name: string, oldValue: string, newValue: string) {
  if (oldValue !== newValue) {
    // 处理属性变化
  }
}
```

### 3. 异步操作

`play()` 等方法应返回 Promise：

```typescript
async play(): Promise<void> {
  try {
    await this._mediaEngine.play();
    this.paused = false;
    this.dispatchEvent(new Event('play'));
  } catch (error) {
    this.error = error;
    this.dispatchEvent(new Event('error'));
    throw error;
  }
}
```

### 4. 错误处理

必须设置 `error` 属性并触发 `error` 事件：

```typescript
this.error = {
  code: 4,  // MEDIA_ELEMENT_ERR_SRC_NOT_SUPPORTED
  message: 'Source not supported'
};
this.dispatchEvent(new Event('error'));
```

### 5. Web Component 兼容

作为自定义元素，可以被 `customElements.whenDefined()` 等待：

```typescript
await customElements.whenDefined('my-media-element');
```

### 6. TimeRanges 支持

确保 `seekable` 和 `buffered` 返回标准的 TimeRanges 对象：

```typescript
// 简单实现
const createTimeRanges = (ranges: Array<{start: number, end: number}>) => ({
  length: ranges.length,
  start: (index: number) => ranges[index]?.start ?? 0,
  end: (index: number) => ranges[index]?.end ?? 0,
  item: (index: number) => ranges[index] ?? null
});

this.buffered = createTimeRanges([
  { start: 0, end: 10 },
  { start: 15, end: 20 }
]);
```

## 测试 checklist

实现完成后，使用以下 checklist 验证：

- [ ] 所有必需属性都已实现
- [ ] 所有必需方法都已实现
- [ ] `play()` 返回 Promise
- [ ] `paused` 状态正确反映播放状态
- [ ] 所有标准媒体事件都被正确触发
- [ ] 事件触发顺序正确
- [ ] textTracks 属性存在且可访问
- [ ] TextTrack 事件被正确触发
- [ ] duration 在元数据加载后被正确设置
- [ ] currentTime 随播放正确更新
- [ ] seekable 和 buffered 返回有效的 TimeRanges
- [ ] 错误情况下设置 error 属性并触发 error 事件
- [ ] 自定义元素可以正常注册和使用

## 可用的自定义媒体元素

以下是一些已经构建的、与 Media Chrome 兼容的自定义媒体元素：

### 官方实现

- **[hls-video-element](https://github.com/muxinc/hls-video-element)**
  - 支持 HLS (HTTP Live Streaming) 流媒体
  - 基于 hls.js
  - 使用 `custom-media-element` 库

- **[youtube-video-element](https://github.com/muxinc/youtube-video-element)**
  - 支持 YouTube 视频播放
  - 基于 YouTube IFrame API
  - 使用 `custom-media-element` 库

- **[dash-video-element](https://github.com/muxinc/dash-video-element)**
  - 支持 DASH (Dynamic Adaptive Streaming over HTTP)
  - 基于 dash.js
  - 使用 `custom-media-element` 库

- **[vimeo-video-element](https://github.com/muxinc/vimeo-video-element)**
  - 支持 Vimeo 视频播放
  - 基于 Vimeo Player API
  - 使用 `custom-media-element` 库

### 社区实现

- **[jwplayer-video-element](https://github.com/muxinc/jwplayer-video-element)**
  - 支持 JW Player 集成

- **[videojs-video-element](https://github.com/muxinc/videojs-video-element)**
  - 支持 Video.js 集成

- **[mux-video-element](https://github.com/muxinc/mux-video-element)**
  - 支持 Mux Video 播放

- **[plyr-video-element](https://github.com/muxinc/plyr-video-element)**
  - 支持 Plyr 集成

### 创建自己的媒体元素

参考以下项目作为起点：

- [custom-media-element](https://github.com/muxinc/custom-media-element) - 基础库
- [hls-video-element](https://github.com/muxinc/hls-video-element) - HLS 实现
- [youtube-video-element](https://github.com/muxinc/youtube-video-element) - YouTube 实现

## 参考资源

- [HTMLMediaElement 规范](https://html.spec.whatwg.org/multipage/media.html#htmlmediaelement)
- [Media Controller 源码](https://github.com/muxinc/media-chrome/blob/main/src/js/media-controller.ts)
- [State Mediator 源码](https://github.com/muxinc/media-chrome/blob/main/src/js/media-store/state-mediator.ts)
- [hls-video-element 示例](https://github.com/muxinc/hls-video-element)
- [youtube-video-element 示例](https://github.com/muxinc/youtube-video-element)
