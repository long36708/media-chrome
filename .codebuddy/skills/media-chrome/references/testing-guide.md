# Media Chrome 测试指南

本文档详细介绍如何为 Media Chrome 组件编写测试、运行测试和调试测试。

## 测试框架

Media Chrome 使用 `@open-wc/testing` 进行单元测试，这是一个专门为 Web Components 设计的测试框架。

### 安装测试依赖

```bash
npm install --save-dev @open-wc/testing
```

### 主要工具

`@open-wc/testing` 包含以下工具：

- **@open-wc/testing** - Web Components 测试辅助工具
- **@web/test-runner** - 测试运行器
- **sinon** - Mock 和 spy 工具
- **@open-wc/karma-esm** - ESM 支持（如需要）

## 测试结构

### 目录结构

```
test/
├── unit/                    # 单元测试
│   ├── media-play-button.test.ts
│   ├── media-time-range.test.ts
│   ├── media-controller.test.ts
│   └── ...
└── integration/             # 集成测试（可选）
    └── ...
```

### 测试文件命名

- 单元测试：`*.test.ts`
- 集成测试：`*.integration.test.ts`

## 基础测试示例

### 简单组件测试

```typescript
import { expect } from '@open-wc/testing';
import { MyCustomButton } from '../../src/js/media-my-button.js';

describe('MyCustomButton', () => {
  it('should be defined', () => {
    const element = document.createElement('my-custom-button');
    expect(element).to.be.instanceOf(MyCustomButton);
  });

  it('should have correct default values', () => {
    const element = document.createElement('my-custom-button') as MyCustomButton;
    expect(element.someProperty).to.equal('default-value');
  });

  it('should have correct default attributes', () => {
    const element = document.createElement('my-custom-button');
    expect(element.hasAttribute('media-paused')).to.be.true;
  });
});
```

### 事件测试

```typescript
import { expect } from '@open-wc/testing';
import { MyCustomButton } from '../../src/js/media-my-button.js';

describe('MyCustomButton - Events', () => {
  it('should dispatch play request on click', async () => {
    const element = document.createElement('my-custom-button');
    document.body.appendChild(element);

    // 等待事件触发
    const listener = new Promise((resolve) => {
      element.addEventListener('media-play-request', resolve);
    });

    // 触发点击
    element.click();

    // 等待并验证事件
    const event = await listener as Event;
    expect(event.type).to.equal('media-play-request');
    expect(event.bubbles).to.be.true;
    expect(event.composed).to.be.true;

    // 清理
    document.body.removeChild(element);
  });

  it('should pause on second click', async () => {
    const element = document.createElement('my-custom-button');
    document.body.appendChild(element);

    // 设置初始状态为播放
    element.mediaPaused = false;

    const listener = new Promise((resolve) => {
      element.addEventListener('media-pause-request', resolve);
    });

    element.click();
    const event = await listener as Event;

    expect(event.type).to.equal('media-pause-request');

    document.body.removeChild(element);
  });
});
```

### 属性变化测试

```typescript
import { expect } from '@open-wc/testing';
import { MyComponent } from '../../src/js/my-component.js';

describe('MyComponent - Attributes', () => {
  it('should update attribute when property changes', async () => {
    const element = document.createElement('my-component') as MyComponent;
    document.body.appendChild(element);

    // 设置属性
    element.someAttribute = 'new-value';

    // 等待 DOM 更新
    await element.updateComplete;

    // 验证属性
    expect(element.getAttribute('some-attribute')).to.equal('new-value');

    document.body.removeChild(element);
  });

  it('should reflect attribute changes to property', async () => {
    const element = document.createElement('my-component') as MyComponent;
    document.body.appendChild(element);

    // 设置 attribute
    element.setAttribute('some-attribute', 'attribute-value');

    // 等待 DOM 更新
    await element.updateComplete;

    // 验证 property
    expect(element.someAttribute).to.equal('attribute-value');

    document.body.removeChild(element);
  });
});
```

### Shadow DOM 测试

```typescript
import { expect } from '@open-wc/testing';
import { MyComponent } from '../../src/js/my-component.js';

describe('MyComponent - Shadow DOM', () => {
  it('should have shadow root', () => {
    const element = document.createElement('my-component') as MyComponent;
    expect(element.shadowRoot).to.not.be.null;
  });

  it('should contain expected elements', () => {
    const element = document.createElement('my-component') as MyComponent;
    const button = element.shadowRoot?.querySelector('button');

    expect(button).to.not.be.null;
    expect(button?.textContent).to.equal('Click me');
  });

  it('should apply styles correctly', async () => {
    const element = document.createElement('my-component');
    document.body.appendChild(element);

    await element.updateComplete;

    const computedStyle = getComputedStyle(element);
    expect(computedStyle.display).to.equal('inline-block');

    document.body.removeChild(element);
  });
});
```

### 状态管理测试

```typescript
import { expect } from '@open-wc/testing';
import { MediaController } from '../../src/js/media-controller.js';

describe('MediaController - State Management', () => {
  it('should update media-paused attribute', async () => {
    const controller = document.createElement('media-controller') as MediaController;
    const video = document.createElement('video');

    controller.appendChild(video);
    document.body.appendChild(controller);

    // 触发播放
    video.play();

    // 等待状态更新
    await new Promise(resolve => setTimeout(resolve, 100));

    expect(controller.hasAttribute('media-paused')).to.be.false;

    document.body.removeChild(controller);
  });

  it('should forward video events to attributes', async () => {
    const controller = document.createElement('media-controller') as MediaController;
    const video = document.createElement('video');

    controller.appendChild(video);
    document.body.appendChild(controller);

    // 模拟 timeupdate 事件
    video.dispatchEvent(new Event('timeupdate'));

    await new Promise(resolve => setTimeout(resolve, 50));

    expect(controller.hasAttribute('media-current-time')).to.be.true;

    document.body.removeChild(controller);
  });
});
```

## 测试最佳实践

### 1. 测试隔离

每个测试应该是独立的，不依赖于其他测试的执行顺序：

```typescript
describe('MyComponent', () => {
  let element: MyComponent;

  beforeEach(() => {
    // 每个测试前创建新实例
    element = document.createElement('my-component') as MyComponent;
    document.body.appendChild(element);
  });

  afterEach(() => {
    // 每个测试后清理
    document.body.removeChild(element);
    element = null as any;
  });

  // 测试...
});
```

### 2. 异步测试处理

对于异步操作，使用 `async/await`：

```typescript
it('should handle async operations', async () => {
  const element = document.createElement('my-component');
  document.body.appendChild(element);

  // 等待组件更新完成
  await element.updateComplete;

  // 等待异步事件
  const promise = new Promise(resolve => {
    element.addEventListener('some-event', resolve);
  });

  element.triggerAsyncOperation();
  await promise;

  expect(element.state).to.equal('completed');

  document.body.removeChild(element);
});
```

### 3. DOM 清理

始终清理创建的 DOM 元素，避免内存泄漏：

```typescript
afterEach(() => {
  // 清理所有添加到文档的元素
  while (document.body.firstChild) {
    document.body.removeChild(document.body.firstChild);
  }
});
```

### 4. 使用描述性的测试名称

测试名称应该清楚地描述测试的目的：

```typescript
// ✅ 好的测试名称
it('should dispatch play request when clicked');
it('should update paused attribute after pause method is called');
it('should show loading spinner while media is buffering');

// ❌ 不好的测试名称
it('works');
it('test 1');
it('click test');
```

### 5. 测试边界情况

不要只测试正常情况，还要测试边界和错误情况：

```typescript
describe('MyComponent', () => {
  it('should handle empty src');
  it('should handle invalid media format');
  it('should handle network errors');
  it('should handle rapid play/pause toggles');
  it('should handle missing attributes');
});
```

## Mock 和 Spy

### Mock DOM 元素

```typescript
import { stub } from 'sinon';

describe('MyComponent with Mock', () => {
  it('should handle mocked media element', () => {
    const mockVideo = {
      play: stub().resolves(),
      pause: stub(),
      paused: true,
      addEventListener: stub(),
      removeEventListener: stub(),
    };

    const element = document.createElement('my-component');
    // 注入 mock
    (element as any).media = mockVideo;

    element.play();

    expect(mockVideo.play.calledOnce).to.be.true;
  });
});
```

### 监听函数调用

```typescript
import { spy } from 'sinon';

describe('MyComponent Spy', () => {
  it('should call custom hook on click', () => {
    const element = document.createElement('my-component');
    const hookSpy = spy(element, 'onClickHook');

    element.click();

    expect(hookSpy.calledOnce).to.be.true;
    expect(hookSpy.calledWith(new MouseEvent('click'))).to.be.true;

    hookSpy.restore();
  });
});
```

## 运行测试

### 运行所有测试

```bash
npm test
```

### 运行特定测试文件

```bash
npm test -- media-play-button
```

### 运行特定测试套件

```bash
npm test -- --grep "MyComponent"
```

### 监听模式（文件变化自动运行）

```bash
npm run test:watch
```

### 覆盖率报告

```bash
npm run test:coverage
```

## 测试覆盖率

### 配置覆盖率

在 `web-test-runner.config.js` 中配置：

```javascript
import { defaultReporter } from '@web/test-runner';

export default {
  files: ['test/**/*.test.js'],
  reporters: [
    defaultReporter({ reportTestResults: true, reportTestErrors: true }),
  ],
  coverageConfig: {
    include: ['src/**/*.js'],
    exclude: ['src/**/*.test.js'],
  },
};
```

### 查看覆盖率报告

```bash
npm run test:coverage

# 查看覆盖率报告
open coverage/index.html
```

## 组件特定测试

### 按钮组件测试

```typescript
describe('MediaPlayButton', () => {
  it('should toggle between play and pause', async () => {
    const controller = document.createElement('media-controller');
    const video = document.createElement('video');
    const button = document.createElement('media-play-button');

    controller.appendChild(video);
    controller.appendChild(button);
    document.body.appendChild(controller);

    // 初始状态应该是 play 按钮
    expect(button.getAttribute('aria-label')).to.equal('Play');

    // 点击后应该变成 pause 按钮
    button.click();
    await new Promise(resolve => setTimeout(resolve, 50));

    expect(button.getAttribute('aria-label')).to.equal('Pause');

    document.body.removeChild(controller);
  });
});
```

### 滑块组件测试

```typescript
describe('MediaTimeRange', () => {
  it('should update current time on input', async () => {
    const controller = document.createElement('media-controller');
    const video = document.createElement('video');
    const range = document.createElement('media-time-range');

    video.src = 'test-video.mp4';
    controller.appendChild(video);
    controller.appendChild(range);
    document.body.appendChild(controller);

    // 等待元数据加载
    await new Promise(resolve => {
      video.addEventListener('loadedmetadata', resolve);
    });

    const slider = range.shadowRoot?.querySelector('input[type="range"]');
    if (slider) {
      slider.value = '50';
      slider.dispatchEvent(new Event('input'));

      await new Promise(resolve => setTimeout(resolve, 50));

      expect(video.currentTime).to.be.closeTo(50, 0.1);
    }

    document.body.removeChild(controller);
  });
});
```

### 控制器组件测试

```typescript
describe('MediaController', () => {
  it('should find media element in media slot', async () => {
    const controller = document.createElement('media-controller');
    const video = document.createElement('video');

    video.slot = 'media';
    controller.appendChild(video);
    document.body.appendChild(controller);

    // 等待控制器初始化
    await new Promise(resolve => setTimeout(resolve, 100));

    expect(controller.media).to.equal(video);

    document.body.removeChild(controller);
  });

  it('should forward media events to control components', async () => {
    const controller = document.createElement('media-controller');
    const video = document.createElement('video');
    const button = document.createElement('media-play-button');

    video.slot = 'media';
    controller.appendChild(video);
    controller.appendChild(button);
    document.body.appendChild(controller);

    await new Promise(resolve => setTimeout(resolve, 100));

    // 模拟播放
    video.dispatchEvent(new Event('play'));

    await new Promise(resolve => setTimeout(resolve, 50));

    expect(button.hasAttribute('media-paused')).to.be.false;

    document.body.removeChild(controller);
  });
});
```

## 集成测试

### 端到端场景测试

```typescript
describe('Media Player Integration', () => {
  it('should complete full play/pause cycle', async () => {
    const html = `
      <media-controller>
        <video slot="media" src="test-video.mp4"></video>
        <media-control-bar>
          <media-play-button></media-play-button>
          <media-time-range></media-time-range>
        </media-control-bar>
      </media-controller>
    `;

    document.body.innerHTML = html;
    const controller = document.body.querySelector('media-controller');
    const video = controller?.querySelector('video');

    expect(video).to.not.be.null;

    // 等待加载
    await new Promise(resolve => {
      video?.addEventListener('loadedmetadata', resolve);
    });

    // 播放
    video?.play();
    await new Promise(resolve => setTimeout(resolve, 100));

    expect(video?.paused).to.be.false;

    // 暂停
    video?.pause();
    await new Promise(resolve => setTimeout(resolve, 50));

    expect(video?.paused).to.be.true;

    // 清理
    document.body.innerHTML = '';
  });
});
```

## 调试测试

### 查看失败测试的详细信息

```bash
npm test -- --reporter=verbose
```

### 只运行失败的测试

```bash
npm test -- --reporter=verbose --only-failures
```

### 在浏览器中调试

在测试代码中添加 `debugger` 语句：

```typescript
it('should work', () => {
  debugger;  // 测试会在此处暂停
  const element = document.createElement('my-component');
  expect(element).to.exist;
});
```

### 输出调试信息

```typescript
it('should have correct state', () => {
  const element = document.createElement('my-component');
  document.body.appendChild(element);

  // 输出调试信息
  console.log('Element state:', element.state);
  console.log('Element attributes:', Array.from(element.attributes));

  expect(element.state).to.equal('expected-state');

  document.body.removeChild(element);
});
```

## 常见测试问题

### 问题 1：自定义元素未定义

**症状**：测试失败，错误 "custom element not defined"

**解决方案**：确保在测试前导入并注册自定义元素

```typescript
import { MyComponent } from '../../src/js/my-component.js';

describe('MyComponent', () => {
  before(() => {
    // 确保元素已定义
    customElements.whenDefined('my-component');
  });

  // 测试...
});
```

### 问题 2：Shadow DOM 元素未找到

**症状**：`expect(element.shadowRoot?.querySelector('button')).to.be.null`

**解决方案**：等待组件更新完成

```typescript
it('should contain button', async () => {
  const element = document.createElement('my-component');
  document.body.appendChild(element);

  // 等待 Shadow DOM 渲染
  await element.updateComplete;

  const button = element.shadowRoot?.querySelector('button');
  expect(button).to.not.be.null;

  document.body.removeChild(element);
});
```

### 问题 3：事件未触发

**症状**：事件监听器没有被调用

**解决方案**：确保使用 `composed: true` 和 `bubbles: true`

```typescript
const event = new CustomEvent('my-event', {
  bubbles: true,
  composed: true,
});
element.dispatchEvent(event);
```

### 问题 4：异步操作未完成

**症状**：测试通过但实际功能不工作

**解决方案**：使用 Promise 等待异步操作

```typescript
it('should update state after async operation', async () => {
  const element = document.createElement('my-component');
  document.body.appendChild(element);

  // 等待异步操作完成
  await new Promise(resolve => setTimeout(resolve, 100));

  expect(element.state).to.equal('updated');

  document.body.removeChild(element);
});
```

## 测试清单

在提交代码前，确保：

- [ ] 所有测试通过
- [ ] 测试覆盖率至少达到 80%
- [ ] 添加了新功能的测试
- [ ] 测试覆盖了边界情况
- [ ] 测试名称描述清晰
- [ ] 测试相互独立，不依赖执行顺序
- [ ] 清理了所有创建的 DOM 元素
- [ ] 测试不依赖外部资源（网络、文件等）

## 参考资源

### 测试框架文档

- [@open-wc/testing](https://open-wc.org/docs/testing/) - Web Components 测试指南
- [Web Test Runner](https://modern-web.dev/docs/test-runner/) - 测试运行器文档
- [Sinon.JS](https://sinonjs.org/) - Mock 和 spy 库

### 测试示例

- `test/unit/media-play-button.test.ts` - 按钮组件测试
- `test/unit/media-time-range.test.ts` - 滑块组件测试
- `test/unit/media-controller.test.ts` - 控制器组件测试

### 最佳实践

- [Testing Library](https://testing-library.com/) - 测试最佳实践
- [JavaScript Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)
