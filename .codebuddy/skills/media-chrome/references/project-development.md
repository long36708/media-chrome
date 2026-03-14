# Media Chrome 项目开发指南

本文档详细介绍如何为 Media Chrome 项目贡献代码、构建和测试。

## 开发环境设置

### 安装依赖

```bash
npm install
```

### 开发模式

```bash
# 监听模式开发（带热重载）
npm run dev
```

## 构建脚本

### 基础脚本

| 命令 | 说明 |
|------|------|
| `npm run clean` | 清理构建产物 |
| `npm run format` | 格式化代码 |
| `npm run lint` | Lint 检查 |
| `npm run build:types` | 类型检查并生成类型定义 |
| `npm run build` | 完整构建（所有格式） |

### 分模块构建

| 命令 | 说明 |
|------|------|
| `npm run build:esm` | 构建 ES Modules 格式 |
| `npm run build:cjs` | 构建 CommonJS 格式 |
| `npm run build:iife:dev` | 构建 IIFE 开发版本 |
| `npm run build:iife:prod` | 构建 IIFE 生产版本 |
| `npm run build:react` | 构建 React 包装器 |

### 测试脚本

```bash
# 运行所有测试
npm test

# 运行特定测试
npm test -- media-play-button
```

## 项目结构

```
media-chrome/
├── src/js/                    # 源代码目录
│   ├── media-*.ts            # 各个组件的实现
│   │   ├── media-controller.ts
│   │   ├── media-play-button.ts
│   │   ├── media-time-range.ts
│   │   └── ...
│   ├── constants.ts          # 常量定义（事件、属性等）
│   ├── utils/                # 工具函数
│   │   ├── server-safe-globals.ts
│   │   └── ...
│   └── media-store/          # 状态管理
│       ├── media-store.ts
│       └── state-mediator.ts
├── src/svgs/                 # SVG 图标
├── examples/                 # 示例代码
│   ├── vanilla/             # 原生 JS 示例
│   ├── nextjs-with-typescript/  # Next.js 示例
│   └── vite-react-with-typescript/  # Vite + React 示例
├── test/                    # 测试文件
│   └── unit/               # 单元测试
├── docs/                    # 文档文件
│   └── src/                # Astro 源文件
│       ├── pages/          # 页面
│       └── components/     # 组件
├── scripts/                 # 构建脚本
└── .codebuddy/            # CodeBuddy 配置
    └── skills/            # AI 技能包
        └── media-chrome/
            ├── SKILL.md
            └── references/
```

## 添加新组件流程

### 步骤 1：创建组件文件

在 `src/js/` 目录下创建新的组件文件，例如 `media-my-button.ts`：

```typescript
import { MediaChromeButton } from './media-chrome-button.js';
import { MediaUIEvents, MediaUIAttributes } from './constants.js';
import { globalThis } from './utils/server-safe-globals.js';

class MyCustomButton extends MediaChromeButton {
  static get observedAttributes() {
    return [
      ...super.observedAttributes,
      'custom-attr',
    ];
  }

  constructor() {
    super();
    // 初始化逻辑
  }

  // 实现必要的方法
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

  // 属性 getter/setter
  get mediaPaused(): boolean {
    return this.getAttribute(MediaUIAttributes.MEDIA_PAUSED) === 'true';
  }

  set mediaPaused(value: boolean) {
    this.setAttribute(MediaUIAttributes.MEDIA_PAUSED, String(value));
  }
}

if (!globalThis.customElements.get('my-custom-button')) {
  globalThis.customElements.define('my-custom-button', MyCustomButton);
}
```

### 步骤 2：在入口文件导出组件

在 `src/js/index.ts` 中添加导出：

```typescript
export { MyCustomButton } from './media-my-button.js';
```

### 步骤 3：添加测试

在 `test/unit/` 或 `test/` 目录下创建测试文件：

```typescript
import { expect } from '@open-wc/testing';
import { MyCustomButton } from '../../src/js/media-my-button.js';

describe('MyCustomButton', () => {
  it('should be defined', () => {
    const element = document.createElement('my-custom-button');
    expect(element).to.be.instanceOf(MyCustomButton);
  });

  it('should dispatch play request on click', async () => {
    const element = document.createElement('my-custom-button');
    document.body.appendChild(element);

    const listener = new Promise((resolve) => {
      element.addEventListener('media-play-request', resolve);
    });

    element.click();
    const event = await listener;

    expect(event.type).to.equal('media-play-request');

    document.body.removeChild(element);
  });
});
```

### 步骤 4：创建示例

在 `examples/vanilla/` 下创建示例 HTML 文件：

```html
<!DOCTYPE html>
<html>
<head>
  <title>My Custom Button Example</title>
  <script type="module" src="../../dist/index.js"></script>
</head>
<body>
  <media-controller>
    <video slot="media" src="video.mp4"></video>
    <media-control-bar>
      <my-custom-button></my-custom-button>
    </media-control-bar>
  </media-controller>
</body>
</html>
```

### 步骤 5：构建和测试

```bash
# 运行测试
npm test

# 构建
npm run build

# 本地验证示例
npm run dev
```

### 步骤 6：添加文档

在 `docs/src/pages/docs/en/` 下创建文档：

```mdx
---
title: My Custom Button
description: Documentation for My Custom Button component
layout: ../../../layouts/MainLayout.astro
---

# My Custom Button

Description of the component...

## Usage

...

## API

...
```

## 组件开发最佳实践

### 基类选择

| 基类 | 适用场景 |
|------|---------|
| `MediaChromeButton` | 按钮类组件（播放、静音、全屏等） |
| `MediaChromeRange` | 滑块类组件（进度条、音量等） |
| `MediaChromeRangeSlider` | 自定义滑块 |
| `GlobalMediaUIAttributes` | 需要访问全局媒体属性的组件 |
| `HTMLElement` | 完全自定义的组件 |

### 常量使用

从 `constants.ts` 导入事件和属性常量：

```typescript
import {
  MediaUIEvents,         // 事件名称
  MediaUIAttributes,      // 属性名称
  MediaStateReceiverAttributes,  // 状态接收属性
  nouns  // 文本标签
} from './constants.js';
```

### 状态管理

使用 `associateElement` 和 `disassociateElement` 关联到 media-controller：

```typescript
import { getMediaController } from './media-controller.js';

class MyComponent extends HTMLElement {
  mediaController: MediaControllerElement | null = null;

  connectedCallback() {
    this.mediaController = getMediaController(this);
    this.mediaController?.associateElement?.(this);
  }

  disconnectedCallback() {
    this.mediaController?.disassociateElement?.(this);
    this.mediaController = null;
  }
}
```

### 样式处理

使用 Shadow DOM 隔离样式：

```typescript
class MyComponent extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <style>
        :host {
          display: inline-block;
        }
      </style>
      <slot></slot>
    `;
  }
}
```

### 事件处理

正确使用 CustomEvent：

```typescript
import { globalThis } from './utils/server-safe-globals.js';

const event = new globalThis.CustomEvent(
  MediaUIEvents.MEDIA_PLAY_REQUEST,
  {
    composed: true,    // 穿越 Shadow DOM 边界
    bubbles: true,     // 事件冒泡
    detail: {          // 可选的额外数据
      someValue: 'data'
    }
  }
);
this.dispatchEvent(event);
```

## 测试指南

**📖 完整的测试指南，包括测试框架、测试示例和最佳实践，请参考：**
`references/testing-guide.md`

### 快速开始

```bash
# 运行所有测试
npm test

# 运行特定测试文件
npm test -- media-play-button

# 查看覆盖率报告
npm run test:coverage
```

### 测试指南包含：

- 测试框架介绍（@open-wc/testing）
- 基础测试示例
- 事件测试、属性测试、Shadow DOM 测试
- 状态管理测试
- 测试最佳实践
- Mock 和 Spy 使用
- 组件特定测试（按钮、滑块、控制器）
- 集成测试
- 调试技巧
- 常见测试问题和解决方案
- 测试清单

### 常用测试命令

| 命令 | 说明 |
|------|------|
| `npm test` | 运行所有测试 |
| `npm test -- <name>` | 运行特定测试文件 |
| `npm run test:watch` | 监听模式 |
| `npm run test:coverage` | 生成覆盖率报告 |

## 构建输出

项目构建后会生成以下产物：

```
dist/
├── index.js                  # ESM 入口
├── index.cjs                 # CommonJS 入口
├── index.d.ts                # TypeScript 类型定义
├── iife/
│   ├── index.js             # IIFE 开发版本
│   └── index.min.js         # IIFE 生产版本（压缩）
└── react/
    └── index.js             # React 包装器
```

## 调试技巧

### 开发者工具

1. **查看 Shadow DOM** - Chrome DevTools 支持查看 Shadow DOM
2. **事件监听** - 使用 `monitorEvents(element)` 监听所有事件
3. **自定义元素状态** - 检查 `customElements.get()` 的注册状态

### 常见问题

**问题 1：组件没有渲染**

检查：
- 组件是否正确注册（`customElements.define`）
- 是否在 `domcontentloaded` 之后定义
- 是否使用了 `customElements.whenDefined()`

**问题 2：事件没有触发**

检查：
- 是否使用了 `composed: true` 和 `bubbles: true`
- 事件名称是否正确（使用常量）
- 监听器是否正确添加

**问题 3：样式不生效**

检查：
- 是否正确使用了 Shadow DOM
- CSS 选择器是否正确
- 是否正确使用了 `:host` 伪类

## 贡献指南

### 提交代码

1. Fork 项目
2. 创建功能分支：`git checkout -b feature/my-feature`
3. 提交更改：`git commit -m 'Add some feature'`
4. 推送到分支：`git push origin feature/my-feature`
5. 创建 Pull Request

### 代码规范

- 遵循 ESLint 规则：`npm run lint`
- 遵循 Prettier 格式：`npm run format`
- 使用 TypeScript 编写新代码
- 添加必要的测试
- 更新相关文档

## 参考资源

### 项目内文件

**源代码：**
- `src/js/constants.ts` - 事件和属性常量定义
- `src/js/media-controller.ts` - MediaController 实现
- `src/js/media-store/state-mediator.ts` - 状态中介者
- `src/js/media-chrome-button.ts` - 按钮基类
- `src/js/media-chrome-range.ts` - 滑块基类

**测试：**
- `test/unit/media-play-button.test.ts` - 按钮组件测试
- `test/unit/media-time-range.test.ts` - 滑块组件测试

### 引用文档

- `references/testing-guide.md` - 完整的测试指南和示例

### 官方文档

- [Media Chrome 开发文档](https://media-chrome.org/docs)
- [Web Components 规范](https://web.dev/custom-elements/)
- [Shadow DOM](https://web.dev/shadowdom/)
