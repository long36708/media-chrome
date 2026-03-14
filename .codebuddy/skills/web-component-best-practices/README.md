# Web Component Best Practices

This skill provides comprehensive best practices for building modern, production-ready Web Components.

## Contents

### SKILL.md
The main skill documentation containing:
- Architecture design patterns
- TypeScript integration guidelines
- Attribute management strategies
- Lifecycle management best practices
- Shadow DOM usage patterns
- Event handling techniques
- Performance optimization tips
- Accessibility guidelines
- Testing strategies
- Component registration patterns

### References

#### `references/complete-example.ts`
A complete, production-ready counter component demonstrating all best practices:
- Base class inheritance
- Attribute-property synchronization
- Event handling with delegation
- Lifecycle management
- Accessibility features
- Keyboard navigation
- Shadow DOM encapsulation
- CSS custom properties for theming

#### `references/attribute-utils.ts`
Utility functions for type-safe attribute operations:
- Boolean, numeric, string, JSON attribute getters/setters
- Custom attribute serializers/deserializers
- Token list and date attribute helpers
- Attribute change watching utilities
- Batch attribute operations

#### `references/css-utils.ts`
CSS utilities for Shadow DOM:
- CSS rule management (get, insert, remove)
- CSS custom property management
- Scoped stylesheets
- Animation helpers
- Media query listeners
- Focus trap implementation
- Resize/Intersection/Mutation observers

#### `references/testing-guide.ts`
Comprehensive testing patterns with Vitest:
- Basic rendering tests
- Attribute-property synchronization tests
- Value update tests
- Event handling tests
- Event dispatching tests
- Boundary condition tests
- Accessibility tests
- Lifecycle tests
- Edge case handling
- Integration tests

### Assets

#### `assets/examples.html`
Live HTML examples demonstrating:
- Basic usage
- Attribute configuration
- JavaScript API
- Event handling
- Styling with CSS custom properties
- Slot usage
- Disabled state
- Multiple instances
- Keyboard navigation

## Key Features

### 1. Type Safety
- Strongly typed constants and events
- Type-safe attribute utilities
- Comprehensive TypeScript support

### 2. Performance
- Efficient DOM updates with requestAnimationFrame
- CSS rule caching
- Minimal attribute changes
- Event delegation for dynamic content

### 3. Accessibility
- ARIA attributes
- Keyboard navigation
- Focus management
- Screen reader support

### 4. Developer Experience
- Clear naming conventions
- Comprehensive documentation
- Reusable utility functions
- Testing patterns

### 5. Best Practices
- Base class pattern
- Shadow DOM encapsulation
- CSS custom properties for theming
- Event composition
- Lifecycle management

## Usage

When creating or refactoring Web Components, this skill provides:

1. **Architectural guidance** - How to structure components
2. **TypeScript patterns** - Type-safe implementations
3. **Attribute management** - Best practices for attributes/properties
4. **Event handling** - Reliable event patterns
5. **Styling** - Shadow DOM and CSS custom properties
6. **Testing** - Comprehensive testing strategies
7. **Accessibility** - A11y best practices

## Example

```typescript
import {
  getBooleanAttr,
  setBooleanAttr,
  getNumericAttr,
  setNumericAttr,
} from './attribute-utils';

export class MyComponent extends HTMLElement {
  static get observedAttributes(): string[] {
    return ['value', 'disabled'];
  }

  // Properties with type-safe getters/setters
  get value(): number {
    return getNumericAttr(this, 'value', 0) ?? 0;
  }

  set value(newValue: number) {
    setNumericAttr(this, 'value', newValue);
  }

  get disabled(): boolean {
    return getBooleanAttr(this, 'disabled');
  }

  set disabled(newValue: boolean) {
    setBooleanAttr(this, 'disabled', newValue);
  }

  connectedCallback(): void {
    this.attachShadow({ mode: 'open' });
    this.render();
  }

  private render(): void {
    this.shadowRoot.innerHTML = `
      <style>
        :host {
          --primary-color: #007bff;
        }
        .button {
          background: var(--primary-color);
        }
      </style>
      <button class="button">
        <slot></slot>
      </button>
    `;
  }
}
```

## Testing

See `references/testing-guide.ts` for comprehensive testing examples covering:
- Rendering
- Attribute synchronization
- Event handling
- Accessibility
- Edge cases
- Integration scenarios

## Contributing

This skill is based on real-world best practices from production Web Component implementations. It provides generic, reusable patterns applicable to any Web Component project.

## License

This skill is part of the .codebuddy/skills collection.
