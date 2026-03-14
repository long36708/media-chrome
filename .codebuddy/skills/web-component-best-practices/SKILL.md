---
name: web-component-best-practices
description: This skill should be used when creating, implementing, or refactoring Web Components. Provides comprehensive best practices for Web Component development including architecture design, TypeScript integration, attribute management, lifecycle handling, Shadow DOM usage, event handling, and performance optimization. Focuses on generic, reusable patterns that apply to any Web Component implementation.
---

# Web Component Best Practices

This skill provides comprehensive best practices for building modern, production-ready Web Components. These practices are based on proven patterns from real-world implementations and focus on maintainability, type safety, performance, and developer experience.

## When to Use This Skill

Use this skill when:
- Creating new Web Components from scratch
- Refactoring existing Web Components
- Designing Web Component architecture
- Implementing complex component interactions
- Optimizing Web Component performance
- Establishing Web Component development standards

## Architecture Design

### Base Class Pattern

Always use a base class to provide common functionality across all components:

```typescript
// base-element.ts
export class BaseElement extends HTMLElement {
  static get observedAttributes(): string[] {
    return [];
  }

  attributeChangedCallback(
    attrName: string,
    oldValue: string | null,
    newValue: string | null
  ): void {}

  connectedCallback(): void {}

  disconnectedCallback(): void {}
}
```

**Benefits:**
- Provides consistent inheritance pattern
- Centralizes common lifecycle hooks
- Enables shared utility methods
- Ensures uniform API across components

### Component Hierarchy

Organize components in logical hierarchies:

```
BaseElement (abstract base)
  ├── InteractiveElement (click, keyboard, focus)
  │     ├── ButtonElement
  │     └── InputElement
  └── ContainerElement (layout, sizing)
        ├── CardElement
        └── PanelElement
```

### Dependency Injection

Use dependency injection for shared services:

```typescript
class MyComponent extends BaseElement {
  #service: DataService;

  set service(value: DataService) {
    this.#service = value;
  }

  connectedCallback(): void {
    if (!this.#service) {
      this.#service = globalThis.dataService;
    }
  }
}
```

## TypeScript Integration

### Type-Safe Constants

Define constants with TypeScript types:

```typescript
// constants.ts
export const ComponentProps = {
  VALUE: 'value',
  LABEL: 'label',
  DISABLED: 'disabled',
} as const;

export const ComponentAttributes = Object.entries(ComponentProps).reduce(
  (dictObj, [key, propName]) => {
    dictObj[key] = propName.toLowerCase();
    return dictObj;
  },
  {} as Record<keyof typeof ComponentProps, string>
);

export type ComponentAttributes = typeof ComponentAttributes;
export type ComponentProps = typeof ComponentProps;
```

### Type-Safe Event System

Define events with strong types:

```typescript
export const ComponentEvents = {
  VALUE_CHANGE: 'valuechange',
  CLICK: 'click',
} as const;

export type ComponentEventType = {
  [K in keyof typeof ComponentEvents]: CustomEvent<{
    [key: string]: any;
  }>;
};
```

### Attribute Type Mapping

Create type-safe attribute getters/setters:

```typescript
// attribute-utils.ts
export function getBooleanAttr(el: HTMLElement, attrName: string): boolean {
  return el.hasAttribute(attrName);
}

export function setBooleanAttr(
  el: HTMLElement,
  attrName: string,
  value: boolean | null | undefined
): void {
  if (value == null) {
    if (el.hasAttribute(attrName)) {
      el.removeAttribute(attrName);
    }
    return;
  }
  if (getBooleanAttr(el, attrName) === value) return;
  el.toggleAttribute(attrName, value);
}

export function getNumericAttr(
  el: HTMLElement,
  attrName: string,
  defaultValue?: number
): number | undefined {
  const attrVal = el.getAttribute(attrName);
  return attrVal != null ? +attrVal : defaultValue;
}

export function setNumericAttr(
  el: HTMLElement,
  attrName: string,
  value: number | null | undefined
): void {
  if (value == null) {
    el.removeAttribute(attrName);
    return;
  }
  const current = getNumericAttr(el, attrName);
  if (current === value) return;
  el.setAttribute(attrName, String(value));
}

export function getStringAttr(
  el: HTMLElement,
  attrName: string,
  defaultValue?: string
): string | undefined {
  return el.getAttribute(attrName) ?? defaultValue;
}

export function setStringAttr(
  el: HTMLElement,
  attrName: string,
  value: string | null | undefined
): void {
  if (value == null) {
    el.removeAttribute(attrName);
    return;
  }
  const current = getStringAttr(el, attrName);
  if (current === value) return;
  el.setAttribute(attrName, value);
}
```

## Attribute Management

### Observed Attributes

Declare observed attributes statically:

```typescript
export class MyComponent extends BaseElement {
  static get observedAttributes(): string[] {
    return [
      ComponentAttributes.VALUE,
      ComponentAttributes.LABEL,
      ComponentAttributes.DISABLED,
    ];
  }

  attributeChangedCallback(
    attrName: string,
    oldValue: string | null,
    newValue: string | null
  ): void {
    if (oldValue === newValue) return;

    switch (attrName) {
      case ComponentAttributes.VALUE:
        this.handleValueChange(oldValue, newValue);
        break;
      case ComponentAttributes.LABEL:
        this.handleLabelChange(oldValue, newValue);
        break;
      case ComponentAttributes.DISABLED:
        this.handleDisabledChange(oldValue, newValue);
        break;
    }
  }

  private handleValueChange(oldValue: string | null, newValue: string | null): void {
    // Update internal state and DOM
  }

  private handleLabelChange(oldValue: string | null, newValue: string | null): void {
    // Update label display
  }

  private handleDisabledChange(oldValue: string | null, newValue: string | null): void {
    // Update disabled state
  }
}
```

### Property-Attribute Synchronization

Use getters/setters to sync properties with attributes:

```typescript
export class MyComponent extends BaseElement {
  // Boolean property
  get disabled(): boolean {
    return getBooleanAttr(this, ComponentAttributes.DISABLED);
  }

  set disabled(value: boolean) {
    setBooleanAttr(this, ComponentAttributes.DISABLED, value);
  }

  // String property
  get label(): string {
    return getStringAttr(this, ComponentAttributes.LABEL, '');
  }

  set label(value: string) {
    setStringAttr(this, ComponentAttributes.LABEL, value);
  }

  // Number property
  get value(): number {
    return getNumericAttr(this, ComponentAttributes.VALUE, 0) ?? 0;
  }

  set value(value: number) {
    setNumericAttr(this, ComponentAttributes.VALUE, value);
  }
}
```

### Custom Attribute Serializers

Create serializers for complex types:

```typescript
// serializers.ts
const AttributeSerializers: Record<string, (value: any) => string> = {
  'array': (arr: any[]) => JSON.stringify(arr),
  'object': (obj: object) => JSON.stringify(obj),
  'date': (date: Date) => date.toISOString(),
  'set': (set: Set<any>) => Array.from(set).join(' '),
};

const AttributeDeserializers: Record<string, (value: string) => any> = {
  'array': (str: string) => JSON.parse(str),
  'object': (str: string) => JSON.parse(str),
  'date': (str: string) => new Date(str),
  'set': (str: string) => new Set(str.split(/\s+/)),
};

export function serializeAttr(attrName: string, value: any): string {
  const serializer = AttributeSerializers[attrName];
  return serializer ? serializer(value) : String(value);
}

export function deserializeAttr(attrName: string, value: string): any {
  const deserializer = AttributeDeserializers[attrName];
  return deserializer ? deserializer(value) : value;
}
```

## Lifecycle Management

### Connection Hook

Use `connectedCallback` for initialization:

```typescript
export class MyComponent extends BaseElement {
  #cleanupFns: Array<() => void> = [];

  connectedCallback(): void {
    this.initShadowDOM();
    this.initEventListeners();
    this.initObservers();
    this.updateFromAttributes();
  }

  private initShadowDOM(): void {
    if (!this.shadowRoot) {
      this.attachShadow({ mode: 'open' });
      this.render();
    }
  }

  private initEventListeners(): void {
    const clickHandler = this.handleClick.bind(this);
    this.addEventListener('click', clickHandler);

    this.#cleanupFns.push(() => {
      this.removeEventListener('click', clickHandler);
    });
  }

  private initObservers(): void {
    const observer = new ResizeObserver(this.handleResize.bind(this));
    observer.observe(this);

    this.#cleanupFns.push(() => {
      observer.disconnect();
    });
  }

  disconnectedCallback(): void {
    // Clean up all resources
    this.#cleanupFns.forEach(cleanup => cleanup());
    this.#cleanupFns = [];
  }
}
```

### Private Field Pattern

Use private fields with arrow functions:

```typescript
export class MyComponent extends BaseElement {
  #value = 0;
  #listeners: Map<string, EventListener> = new Map();

  // Private event handler with bound `this`
  #handleClick = (event: MouseEvent): void => {
    this.value = this.value + 1;
    this.dispatchEvent(new CustomEvent('change', {
      detail: { value: this.value },
      bubbles: true,
      composed: true,
    }));
  };

  private handleClick = (event: MouseEvent): void => { // Private method
    // Implementation
  };
}
```

## Shadow DOM Usage

### Shadow DOM Encapsulation

Always use Shadow DOM for style encapsulation:

```typescript
export class MyComponent extends BaseElement {
  static shadowRootOptions: ShadowRootInit = {
    mode: 'open',
    delegatesFocus: false,
  };

  connectedCallback(): void {
    if (!this.shadowRoot) {
      this.attachShadow((this.constructor as typeof MyComponent).shadowRootOptions);
      this.render();
    }
  }

  private render(): void {
    this.shadowRoot.innerHTML = `
      <style>
        :host {
          display: inline-block;
        }

        .content {
          color: var(--text-color, #333);
        }
      </style>
      <div class="content">
        <slot></slot>
      </div>
    `;
  }
}
```

### CSS Custom Properties

Define and use CSS custom properties for theming:

```typescript
export class MyComponent extends BaseElement {
  private render(): void {
    this.shadowRoot.innerHTML = `
      <style>
        :host {
          /* Default theme values */
          --primary-color: #007bff;
          --secondary-color: #6c757d;
          --text-color: #333;
          --background-color: #fff;
          --border-radius: 4px;
          --padding: 8px 16px;
          --font-size: 14px;
        }

        .button {
          background-color: var(--primary-color);
          color: white;
          padding: var(--padding);
          border-radius: var(--border-radius);
          font-size: var(--font-size);
        }

        .button:hover {
          background-color: var(--secondary-color);
        }
      </style>
      <button class="button">
        <slot>Default Label</slot>
      </button>
    `;
  }
}
```

### Dynamic Style Updates

Use CSS rules for dynamic updates:

```typescript
// utils/css.ts
export function getOrInsertCSSRule(
  styleParent: Element | ShadowRoot,
  selectorText: string
): CSSStyleRule {
  const stylesheets = Array.from(styleParent.styleSheets) as CSSStyleSheet[];
  for (const stylesheet of stylesheets) {
    for (let i = 0; i < stylesheet.cssRules.length; i++) {
      const rule = stylesheet.cssRules[i];
      if (rule.type === CSSRule.STYLE_RULE) {
        const styleRule = rule as CSSStyleRule;
        if (styleRule.selectorText === selectorText) {
          return styleRule;
        }
      }
    }
  }

  // Insert new rule
  const stylesheet = stylesheets[0];
  const index = stylesheet.insertRule(`${selectorText} {}`, stylesheet.cssRules.length);
  return stylesheet.cssRules[index] as CSSStyleRule;
}

// Usage in component
export class MyComponent extends BaseElement {
  #hostRule: CSSStyleRule | null = null;

  connectedCallback(): void {
    super.connectedCallback();
    this.#hostRule = getOrInsertCSSRule(this.shadowRoot!, ':host');
    this.updateStyles();
  }

  private updateStyles(): void {
    if (this.#hostRule) {
      this.#hostRule.style.setProperty('--progress', `${this.progress}%`);
    }
  }
}
```

## Event Handling

### Arrow Function Pattern

Use arrow functions for event handlers to maintain `this` binding:

```typescript
export class MyComponent extends BaseElement {
  #clickHandler = (event: MouseEvent): void => {
    // `this` is correctly bound
    this.handleClick(event);
  };

  connectedCallback(): void {
    super.connectedCallback();
    this.addEventListener('click', this.#clickHandler);
  }

  disconnectedCallback(): void {
    super.disconnectedCallback();
    this.removeEventListener('click', this.#clickHandler);
  }

  private handleClick(event: MouseEvent): void {
    // Implementation
  }
}
```

### Event Composition

Use `composed: true` for events to cross Shadow DOM boundaries:

```typescript
export class MyComponent extends BaseElement {
  private dispatchValueChange(newValue: number): void {
    const event = new CustomEvent('valuechange', {
      detail: { value: newValue },
      bubbles: true,
      composed: true, // Allows event to escape Shadow DOM
    });
    this.dispatchEvent(event);
  }
}
```

### Event Delegation

Use event delegation for dynamic content:

```typescript
export class MyComponent extends BaseElement {
  connectedCallback(): void {
    super.connectedCallback();
    this.shadowRoot!.addEventListener('click', this.handleDelegatedClick);
  }

  #handleDelegatedClick = (event: Event): void => {
    const target = event.target as HTMLElement;
    const button = target.closest('.action-button');

    if (button) {
      const action = button.dataset.action;
      this.handleAction(action);
    }
  };

  private handleAction(action: string | undefined): void {
    // Handle action
  }
}
```

## Performance Optimization

### DOM Updates

Minimize DOM operations:

```typescript
export class MyComponent extends BaseElement {
  #renderPending = false;

  requestRender(): void {
    if (this.#renderPending) return;

    this.#renderPending = true;
    requestAnimationFrame(() => {
      this.render();
      this.#renderPending = false;
    });
  }

  private render(): void {
    // Batch DOM updates here
  }
}
```

### Attribute Change Optimization

Avoid redundant updates:

```typescript
export class MyComponent extends BaseElement {
  get value(): number {
    return getNumericAttr(this, 'value', 0) ?? 0;
  }

  set value(newValue: number) {
    const current = this.value;
    if (current === newValue) return; // Skip if unchanged
    setNumericAttr(this, 'value', newValue);
  }
}
```

### Virtual Scrolling (Advanced)

For lists with many items, implement virtual scrolling:

```typescript
export class VirtualList extends BaseElement {
  #visibleStart = 0;
  #visibleEnd = 0;
  #itemHeight = 40;

  updateVisibleRange(): void {
    const containerHeight = this.clientHeight;
    const scrollTop = this.scrollTop;

    this.#visibleStart = Math.floor(scrollTop / this.#itemHeight);
    this.#visibleEnd = Math.ceil((scrollTop + containerHeight) / this.#itemHeight);

    this.renderVisibleItems();
  }

  private renderVisibleItems(): void {
    // Render only visible items
  }
}
```

## Accessibility

### ARIA Attributes

Always include appropriate ARIA attributes:

```typescript
export class ButtonComponent extends BaseElement {
  static get observedAttributes(): string[] {
    return ['label', 'disabled'];
  }

  private render(): void {
    this.shadowRoot.innerHTML = `
      <button
        part="button"
        aria-label="${this.label}"
        aria-disabled="${this.disabled}"
        ?disabled="${this.disabled}"
      >
        <slot></slot>
      </button>
    `;
  }
}
```

### Keyboard Navigation

Implement keyboard navigation for interactive components:

```typescript
export class InteractiveComponent extends BaseElement {
  #handleKeyDown = (event: KeyboardEvent): void => {
    switch (event.key) {
      case 'Enter':
      case ' ':
        event.preventDefault();
        this.activate();
        break;
      case 'ArrowLeft':
        event.preventDefault();
        this.navigate(-1);
        break;
      case 'ArrowRight':
        event.preventDefault();
        this.navigate(1);
        break;
    }
  };

  connectedCallback(): void {
    super.connectedCallback();
    this.setAttribute('tabindex', '0');
    this.addEventListener('keydown', this.#handleKeyDown);
  }

  private activate(): void {
    // Activate component
  }

  private navigate(direction: number): void {
    // Navigate to next/previous item
  }
}
```

### Focus Management

Manage focus properly:

```typescript
export class ModalComponent extends BaseElement {
  #previousFocus: HTMLElement | null = null;

  open(): void {
    this.#previousFocus = document.activeElement as HTMLElement;
    const focusable = this.shadowRoot!.querySelector('[autofocus]') as HTMLElement;
    focusable?.focus();
  }

  close(): void {
    this.#previousFocus?.focus();
  }
}
```

## Testing

### Unit Testing Pattern

Test components in isolation:

```typescript
// my-component.test.ts
import { describe, it, expect, vi } from 'vitest';
import { MyComponent } from './my-component';

describe('MyComponent', () => {
  it('should render with default values', () => {
    const element = document.createElement('my-component') as MyComponent;
    document.body.appendChild(element);

    expect(element.value).toBe(0);
    expect(element.disabled).toBe(false);
  });

  it('should update value attribute', () => {
    const element = document.createElement('my-component') as MyComponent;
    document.body.appendChild(element);

    element.value = 42;
    expect(element.getAttribute('value')).toBe('42');
  });

  it('should dispatch valuechange event', () => {
    const element = document.createElement('my-component') as MyComponent;
    document.body.appendChild(element);

    const handler = vi.fn();
    element.addEventListener('valuechange', handler);

    element.value = 100;

    expect(handler).toHaveBeenCalledTimes(1);
    expect(handler).toHaveBeenCalledWith(
      expect.objectContaining({ detail: { value: 100 } })
    );
  });
});
```

## Component Registration

### Safe Registration Pattern

Register components only once:

```typescript
export function registerCustomElement(
  tagName: string,
  componentClass: CustomElementConstructor
): void {
  if (!globalThis.customElements.get(tagName)) {
    globalThis.customElements.define(tagName, componentClass);
  }
}

// Usage
registerCustomElement('my-component', MyComponent);
```

### Lazy Registration

Register components on-demand:

```typescript
export function lazyRegister(
  tagName: string,
  importer: () => Promise<{ default: CustomElementConstructor }>
): void {
  if (customElements.get(tagName)) return;

  const observer = new MutationObserver((mutations) => {
    mutations.forEach((mutation) => {
      mutation.addedNodes.forEach((node) => {
        if (
          node instanceof HTMLElement &&
          node.tagName.toLowerCase() === tagName
        ) {
          observer.disconnect();
          importer().then(({ default: componentClass }) => {
            customElements.define(tagName, componentClass);
          });
        }
      });
    });
  });

  observer.observe(document.documentElement, {
    childList: true,
    subtree: true,
  });
}

// Usage
lazyRegister('my-component', () => import('./my-component'));
```

## Error Handling

### Graceful Degradation

Handle errors gracefully:

```typescript
export class MyComponent extends BaseElement {
  connectedCallback(): void {
    try {
      this.init();
    } catch (error) {
      console.error('Failed to initialize MyComponent:', error);
      this.renderFallback();
    }
  }

  private renderFallback(): void {
    this.shadowRoot!.innerHTML = `
      <style>
        .error {
          color: #dc3545;
          padding: 1rem;
          border: 1px solid #dc3545;
          border-radius: 4px;
        }
      </style>
      <div class="error">
        Failed to load component
      </div>
    `;
  }
}
```

### Validation

Validate inputs and attributes:

```typescript
export class InputComponent extends BaseElement {
  set value(newValue: number) {
    if (!Number.isFinite(newValue)) {
      throw new Error('Value must be a finite number');
    }
    if (newValue < this.min || newValue > this.max) {
      throw new Error(`Value must be between ${this.min} and ${this.max}`);
    }
    setNumericAttr(this, 'value', newValue);
  }
}
```

## Documentation

### Web Component Manifest

Include a manifest for documentation tools:

```typescript
// my-component.manifest.ts
export default {
  tagName: 'my-component',
  description: 'A description of what this component does',
  attributes: [
    {
      name: 'value',
      type: 'number',
      description: 'The current value',
      default: 0,
    },
    {
      name: 'disabled',
      type: 'boolean',
      description: 'Whether the component is disabled',
      default: false,
    },
  ],
  events: [
    {
      name: 'valuechange',
      description: 'Fired when the value changes',
      detail: { value: 'number' },
    },
  ],
  slots: [
    {
      name: '',
      description: 'Default slot for content',
    },
  ],
  cssParts: [
    {
      name: 'button',
      description: 'Internal button element',
    },
  ],
};
```

## Migration Guide

### Upgrading Components

When upgrading components, maintain backward compatibility:

```typescript
export class MyComponent extends BaseElement {
  static get observedAttributes(): string[] {
    return [
      'value', // New attribute
      'old-value', // Deprecated attribute
    ];
  }

  attributeChangedCallback(
    attrName: string,
    oldValue: string | null,
    newValue: string | null
  ): void {
    super.attributeChangedCallback(attrName, oldValue, newValue);

    if (attrName === 'old-value') {
      // Migrate to new attribute
      console.warn(
        'old-value is deprecated. Use value instead.',
        this
      );
      this.setAttribute('value', newValue || '0');
    }
  }
}
```

## Summary Checklist

When creating Web Components, ensure:

- ✅ Use a base class for common functionality
- ✅ Implement TypeScript with type-safe constants
- ✅ Use utility functions for attribute operations
- ✅ Properly manage lifecycle hooks
- ✅ Encapsulate styles with Shadow DOM
- ✅ Use CSS custom properties for theming
- ✅ Handle events with arrow functions
- ✅ Optimize DOM operations
- ✅ Include ARIA attributes for accessibility
- ✅ Implement keyboard navigation
- ✅ Write comprehensive tests
- ✅ Register components safely
- ✅ Handle errors gracefully
- ✅ Document component API
- ✅ Maintain backward compatibility
