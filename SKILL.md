---
name: grapesjs
description: GrapesJS web builder framework expert. Use this skill whenever working with GrapesJS — the JavaScript framework for building drag-and-drop web page editors and website builders. Covers ALL 20 module APIs: Editor, Components (addType, Symbols, isComponent), Blocks, Commands, Panels, Style Manager (sectors, custom property types), Traits (custom trait types), Canvas (Spots, zoom, iframe access), Storage (local/remote/custom), Pages (multi-page), Asset Manager, Selector Manager, Device Manager, Undo Manager, Parser, Keymaps, Modal, Rich Text Editor, I18n. Includes React integration (@grapesjs/react), external framework custom events (*:custom pattern), plugin development, and CSS theming. Triggers on: GrapesJS, grapejs, web builder, page builder, drag-and-drop editor, visual HTML editor, WYSIWYG builder, CMS builder, landing page builder, email template editor, or any visual HTML/CSS composition tool. Also use when adding custom component types, traits, blocks, or commands to a GrapesJS editor instance.
---

# GrapesJS Expert Skill

You are an expert in GrapesJS, the open-source JavaScript framework for building web page builders and visual editors. This skill covers the complete API surface and best practices.

## Architecture Overview

GrapesJS follows a module-based architecture where the Editor is the central hub:

```
Editor (grapesjs.init)
├── DomComponents   → Component tree (addType, getWrapper, etc.)
├── BlockManager    → Draggable blocks palette
├── Panels          → UI panels with command buttons
├── Commands        → Named actions (run/stop lifecycle)
├── StyleManager    → CSS property sectors & custom property types
├── Traits          → Component attribute editors (Text, Select, Color, etc.)
├── SelectorManager → CSS class/state management
├── AssetManager    → Image/media library
├── Canvas          → The editing iframe + spots
├── Pages           → Multi-page support
├── Storage         → Local/remote persistence
├── Devices         → Responsive viewport presets
├── UndoManager     → Undo/redo stack
├── Keymaps         → Keyboard shortcuts
├── Modal           → Modal dialogs
├── RichTextEditor  → Inline text formatting toolbar
├── Parser          → HTML/CSS parser
├── I18n            → Internationalization
└── (Emitter)       → Event system built into editor
```

All modules are accessed as `editor.ModuleName` (e.g., `editor.DomComponents`, `editor.Blocks`).

## Initialization

```js
import grapesjs from 'grapesjs';

const editor = grapesjs.init({
  container: '#gjs',
  fromElement: true,           // use existing HTML in container
  height: '100%',
  width: 'auto',

  // Storage
  storageManager: {
    type: 'local',
    autosave: true,
    stepsBeforeSave: 1,
    options: {
      local: { key: 'gjs-project' },
    },
  },

  // Disable defaults you don't need
  panels: { defaults: [] },
});
```

## Component System

The component system is the heart of GrapesJS. Every element on the canvas is a Component model with a corresponding view.

### Custom Component Types

Define reusable component types with `addType`:

```js
editor.DomComponents.addType('my-component', {
  // Detection: how to recognize this type from HTML
  isComponent: (el) => el.tagName === 'MY-TAG' || el.classList?.contains('my-class'),

  model: {
    defaults: {
      tagName: 'div',
      draggable: 'div, section, main',  // CSS selector for valid drop targets
      droppable: true,                    // can accept children
      removable: true,
      copyable: true,
      stylable: true,                     // can be styled in Style Manager
      attributes: { class: 'my-component' },
      traits: [
        'id',
        'title',
        { type: 'select', name: 'variant', options: [
          { value: 'primary', name: 'Primary' },
          { value: 'secondary', name: 'Secondary' },
        ]},
      ],
      styles: `.my-component { padding: 16px; }`,  // component-scoped CSS
      components: '<span>Default content</span>',     // default children
    },

    init() {
      // Called once when component is created
      this.on('change:attributes:variant', this.onVariantChange);
    },

    updated(property, value, prevValue) {
      // Called when any property changes
    },

    removed() {
      // Cleanup when component is removed
    },
  },

  view: {
    events: {
      'click': 'onClick',
    },

    init({ model }) {
      this.listenTo(model, 'change:someprop', this.render);
    },

    onRender({ el, model }) {
      // Custom canvas rendering (React mount, third-party widget init, etc.)
      // Do NOT use this for DOM manipulation that should survive export
    },

    removed() {
      // Cleanup (unmount React, destroy widgets, etc.)
    },
  },
});
```

### Extending Existing Types

```js
editor.DomComponents.addType('my-input', {
  extend: 'input',
  model: {
    defaults: {
      tagName: 'input',
      attributes: { type: 'text' },
    },
  },
});
```

### Component Instance API

```js
const comp = editor.getSelected();

// Read
comp.get('tagName');
comp.getAttributes();
comp.props();                          // all properties
comp.components();                     // children collection
comp.find('.some-class');              // query descendants
comp.toHTML();                         // export HTML
comp.getEl();                          // live DOM element (canvas only)
comp.getView();                        // view instance

// Write
comp.set('tagName', 'span');
comp.setAttributes({ id: 'my-id' });
comp.addAttributes({ 'data-value': 'x' });
comp.removeAttributes('data-value');
comp.components('<p>New children</p>');  // replace children
comp.append('<p>Appended</p>');
comp.setStyle({ color: 'red' });

// Lifecycle
comp.remove();                          // remove from canvas
comp.replaceWith(otherComp);            // swap
```

### Components & Symbols (Reusable Synced Components)

```js
// Create a symbol (reusable component)
const symbol = editor.DomComponents.addSymbol(component);
// All instances of this symbol stay in sync
// Detach an instance:
editor.DomComponents.detachSymbol(instance);
```

## Blocks (Draggable Palette)

Blocks are the user-facing drag items that create components on the canvas.

```js
editor.BlockManager.add('hero-section', {
  label: 'Hero',
  category: 'Sections',
  select: true,      // auto-select after placement
  activate: true,    // trigger active event after placement
  // BEST PRACTICE: use component definition object, NOT raw HTML
  content: { type: 'hero-section' },
});
```

**Key principles:**
- Use component definition objects (`{ type: 'my-type' }`) instead of raw HTML strings — this ensures custom types with their traits, styles, and behavior are properly instantiated
- Do NOT put functions, callbacks, or non-serializable values in block content
- Use component `styles` property for CSS, not inline styles in blocks
- Customize block UI rendering via the `block:custom` event

## Traits (Property Panel)

Traits let users edit component attributes through form controls.

### Built-in Trait Types

- `text` — single-line text input
- `number` — numeric input
- `checkbox` — boolean toggle
- `select` — dropdown
- `color` — color picker
- `button` — action button

### Define Traits on Components

```js
defaults: {
  traits: [
    'name',                              // shorthand: attribute name
    'placeholder',                       // maps to attribute
    { type: 'checkbox', name: 'required' },
    {
      type: 'select',
      name: 'data-size',
      label: 'Size',
      options: [
        { value: 'sm', name: 'Small' },
        { value: 'md', name: 'Medium' },
        { value: 'lg', name: 'Large' },
      ],
    },
    {
      type: 'number',
      name: 'data-columns',
      label: 'Columns',
      min: 1,
      max: 12,
      step: 1,
    },
  ],
}
```

### Custom Trait Types

```js
editor.Traits.addType('slider', {
  createInput({ trait }) {
    const el = document.createElement('input');
    el.type = 'range';
    el.min = trait.get('min') || 0;
    el.max = trait.get('max') || 100;
    return el;
  },

  onEvent({ elInput, component, event }) {
    const value = elInput.value;
    component.addAttributes({ 'data-value': value });
    // Or update a CSS property:
    // component.setStyle({ opacity: value / 100 });
  },

  onUpdate({ elInput, component }) {
    elInput.value = component.getAttributes()['data-value'] || 50;
  },
});
```

## Style Manager

Provides a visual CSS editing panel organized into sectors.

```js
grapesjs.init({
  styleManager: {
    sectors: [
      {
        name: 'Dimension',
        open: true,
        buildProps: ['width', 'height', 'max-width', 'min-height', 'margin', 'padding'],
        properties: [
          {
            type: 'integer',
            name: 'The width',
            property: 'width',
            units: ['px', '%', 'vw'],
            defaults: 'auto',
            min: 0,
          },
        ],
      },
      {
        name: 'Typography',
        open: false,
        buildProps: ['font-family', 'font-size', 'font-weight', 'color', 'text-align'],
      },
    ],
  },
});
```

### Custom Style Property Types

```js
editor.StyleManager.addType('color-gradient', {
  create({ props, component }) {
    const el = document.createElement('input');
    el.type = 'text';
    return el;
  },
  emit({ updateStyle, value, property }) {
    updateStyle(value);
  },
  update({ value, updateStyle }) {
    // sync UI from current value
  },
  destroy() {
    // cleanup
  },
});
```

## Commands & Panels

Commands are named actions that can be triggered by buttons, keyboard shortcuts, or code.

```js
// Simple command
editor.Commands.add('show-alert', (editor, sender, options = {}) => {
  alert(options.message || 'Hello!');
});

// Stateful command (with run/stop lifecycle)
editor.Commands.add('my-tool', {
  run(editor) { /* activate */ },
  stop(editor) { /* deactivate */ },
});

// Execute
editor.runCommand('show-alert', { message: 'Hi' });
editor.stopCommand('my-tool');
```

### Panels with Buttons

```js
editor.Panels.addPanel({
  id: 'toolbar',
  el: '.toolbar-container',
  buttons: [
    {
      id: 'undo',
      className: 'fa fa-undo',
      command: 'core:undo',
    },
    {
      id: 'preview',
      className: 'fa fa-eye',
      command: 'core:preview',
      active: false,
      togglable: true,
    },
  ],
});
```

### Built-in Commands

All prefixed with `core:`: `canvas-clear`, `component-delete`, `copy`, `paste`, `undo`, `redo`, `preview`, `fullscreen`, `open-code`, `open-layers`, `open-styles`, `open-traits`, `open-blocks`, `open-assets`.

### Intercepting Commands

```js
editor.on('command:run:before:component-delete', (options) => {
  if (shouldPreventDelete) {
    options.abort = true;  // cancel the command
  }
});
```

## Canvas API

The canvas is an iframe where the page is rendered.

```js
const canvas = editor.Canvas;

// Access iframe elements
canvas.getDocument();      // iframe document
canvas.getWindow();        // iframe window
canvas.getBody();          // iframe body
canvas.getFrameEl();       // iframe element

// Zoom & scroll
canvas.setZoom(75);        // 0-100
canvas.getZoom();
canvas.scrollTo(component, { behavior: 'smooth' });

// Custom Spots (visual markers on canvas)
canvas.addSpot({ type: 'select', component: selectedComp });
canvas.removeSpots({ type: 'select' });
```

## Storage

### Local Storage (default)

```js
storageManager: {
  type: 'local',
  options: { local: { key: 'my-project' } },
}
```

### Remote Storage

```js
storageManager: {
  type: 'remote',
  options: {
    remote: {
      urlStore: '/api/save',
      urlLoad: '/api/load',
      headers: { Authorization: 'Bearer token' },
    },
  },
}
```

### Custom Storage Backend

```js
editor.Storage.add('my-backend', {
  async load(options) {
    const res = await fetch('/api/project');
    return res.json();
  },
  async store(data, options) {
    await fetch('/api/project', {
      method: 'POST',
      body: JSON.stringify(data),
    });
  },
});

// Then set it as current
editor.Storage.setCurrent('my-backend');
```

### Manual Save/Load

```js
editor.store();              // save
editor.load();               // load
editor.getProjectData();     // get full project JSON
editor.loadProjectData(data); // restore from JSON
```

## Pages (Multi-page Projects)

```js
const pages = editor.Pages;

pages.add({ id: 'about', name: 'About Us', component: '<div>About</div>' });
pages.select('about');
pages.remove('about');

// Each page has its own component tree and styles
const page = pages.get('about');
page.get('component');  // root component
page.get('styles');     // page-specific CSS rules
```

## Events System

The editor is an event emitter. All module events follow `module:action` naming.

```js
// Listen
editor.on('component:selected', (component) => { });
editor.on('component:update:tagName', (component) => { });
editor.on('block:drag:stop', (component) => { });
editor.on('storage:store', (data) => { });
editor.on('canvas:drop', (data) => { });

// Once
editor.once('load', () => { });

// Remove listener
editor.off('component:selected', handler);

// Namespace: listen to all component events
editor.on('component', (event, ...args) => { });
```

### Key Event Namespaces

| Module | Events |
|--------|--------|
| Editor | `load`, `update`, `undo`, `redo`, `destroy` |
| Components | `component:create`, `component:add`, `component:remove`, `component:update`, `component:selected`, `component:drag:start/end` |
| Blocks | `block:add`, `block:remove`, `block:drag:start`, `block:drag:stop`, `block:custom` |
| Style | `style:target`, `style:property:update`, `style:update` |
| Storage | `storage:start`, `storage:end`, `storage:error`, `storage:store`, `storage:load` |
| Canvas | `canvas:drop`, `canvas:zoom`, `canvas:pointer`, `canvas:frame:load` |
| Page | `page:select`, `page:add`, `page:remove`, `page:update` |
| Command | `command:run:before:{id}`, `command:run:{id}`, `command:stop:{id}` |

## Plugins

Plugins encapsulate reusable functionality:

```js
function myPlugin(editor, options) {
  const opts = { defaultKey: 'value', ...options };

  editor.DomComponents.addType('my-widget', { /* ... */ });
  editor.BlockManager.add('my-widget', {
    content: { type: 'my-widget' },
    label: opts.label || 'My Widget',
  });
}

// Usage
grapesjs.init({
  plugins: [myPlugin],
  pluginsOpts: {
    [myPlugin]: { defaultKey: 'custom' },
  },
});
```

TypeScript:
```ts
import type { Plugin } from 'grapesjs';

interface Opts { theme?: string; }
const plugin: Plugin<Opts> = (editor, options) => { /* ... */ };
```

## React Integration

GrapesJS provides `@grapesjs/react` for React projects:

```tsx
import { GjsEditor, Canvas, WithEditor } from '@grapesjs/react';
import grapesjs from 'grapesjs';

function MyEditor() {
  return (
    <GjsEditor
      grapesjs={grapesjs}
      options={{
        container: '#gjs',
        fromElement: true,
        storageManager: { type: 'local' },
      }}
      onEditor={(editor) => {
        // Register types, blocks, etc.
      }}
    >
      <WithEditor>
        <div style={{ display: 'flex' }}>
          <Sidebar />    {/* blocks, layers, pages */}
          <Canvas />     {/* editing canvas */}
          <StylePanel /> {/* style manager, traits */}
        </div>
      </WithEditor>
    </GjsEditor>
  );
}
```

Key components: `GjsEditor` (provider), `Canvas`, `WithEditor` (HOC ensures editor ready), `useEditor` (hook).

## Common Patterns

### Detect and Prevent Unwanted Drops

```js
editor.on('block:drag:stop', (component, block, { cancel }) => {
  if (isInvalidPlacement(component)) {
    cancel();  // prevent the drop
  }
});
```

### Custom Export

```js
function exportProject() {
  const html = editor.getHtml({ cleanId: true });
  const css = editor.getCss({ onlyMatched: true });
  const json = editor.getProjectData();
  // Download or send to server
}
```

### Theming the Editor UI

```css
:root {
  --gjs-primary-color: #78366a;
  --gjs-secondary-color: rgba(255,255,255,0.7);
  --gjs-tertiary-color: #ec5896;
  --gjs-quaternary-color: #ec5896;
}

/* Or override CSS classes */
.gjs-one-bg { background: #1a1a2e; }
.gjs-two-color { color: #e0e0e0; }
.gjs-three-bg { background: #16213e; }
.gjs-four-color { color: #0f3460; }
```

### Responsive Design with Devices

```js
editor.Devices.add({ name: 'Tablet', width: '768px', widthMedia: '768px' });

editor.on('device:select', (device) => {
  console.log('Switched to:', device.get('name'));
});
```

### Asset Manager with Custom Upload

```js
grapesjs.init({
  assetManager: {
    upload: '/api/upload',
    uploadName: 'files',
    headers: { Authorization: 'Bearer token' },
    // Server must return: { data: [{ src: 'https://...' }] }
  },
});

// Listen for uploads
editor.on('asset:upload', (response) => {
  // response.body contains server response
});
```

### Custom Canvas Rendering (Third-party Widgets)

For widgets that need live JS in the canvas (charts, sliders, maps):

```js
editor.DomComponents.addType('swiper', {
  model: { defaults: { tagName: 'div', draggable: true } },
  view: {
    onRender({ el, model }) {
      // Initialize Swiper on the canvas element
      const swiper = new Swiper(el, { /* config */ });
      model.set('swiper-instance', swiper);  // store for cleanup
    },
    removed() {
      const swiper = this.model.get('swiper-instance');
      swiper?.destroy();
    },
  },
});
```

**Important:** `onRender` runs in the canvas iframe context. Third-party widget instances won't appear in exported HTML — the exported HTML should include the widget's initialization script separately.

## External Framework Integration (React/Vue)

GrapesJS provides `*:custom` events for replacing default UI with React/Vue components:

### trait:custom — Custom Trait UI

```js
editor.on('trait:custom', ({ trait, component, el }) => {
  const root = createRoot(el);
  root.render(<MyCustomTrait trait={trait} component={component} />);
});
```

Trait options support `changeProp: 1` to trigger component prop changes, and `eventCapture: ['input', 'change']` for custom event delegation.

### style:custom — Custom Style Property UI

```js
editor.on('style:custom', ({ property, component, el }) => {
  // Render custom style editor UI
});
```

### block:custom — Custom Block Preview

```js
editor.on('block:custom', ({ block, el }) => {
  createApp(BlockPreview, { block }).mount(el);
});
```

### asset:custom — Custom Asset Manager

```js
editor.on('asset:custom', ({ assets, options, open, close }) => {
  // Completely replace asset manager UI
});
```

For standalone asset picker outside the editor, use `assetManager.custom.open/close`:
```js
grapesjs.init({
  assetManager: {
    custom: {
      open(props) { props.container.appendChild(myElement); },
      close() { /* cleanup */ },
    },
  },
});
```

## Custom Modal (for Bootstrap, Ant Design, etc.)

```js
grapesjs.init({
  modal: { custom: true },
});

editor.on('modal', ({ open, title, content, close }) => {
  if (open) {
    Modal.info({ title, content });  // your framework's modal
  } else {
    Modal.destroyAll();
  }
});
```

**Trap:** When using modals with commands, always stop the command on modal close, or the UI state becomes inconsistent:
```js
editor.Modal.onceClose(() => {
  editor.stopCommand('my-modal-command');
});
```

## Undo Manager

```js
const undoManager = editor.UndoManager;

undoManager.undo();
undoManager.redo();
undoManager.hasUndo();     // boolean
undoManager.hasRedo();     // boolean

// Temporarily stop tracking during batch operations
undoManager.skip(() => {
  component.set('tagName', 'span');
  component.setStyle({ color: 'red' });
});
// Changes inside skip() won't create undo entries

undoManager.clear();       // clear entire undo stack
```

Components and CSS rules are automatically tracked — no need to register them manually.

## Selector Manager

```js
const selectors = editor.Selectors;

// Add CSS classes
selectors.add({ name: 'highlight', label: 'Highlight' });
selectors.add('.my-class');  // shorthand string form

// Configure pseudo-states for the Style Manager
selectors.setStates([
  { name: 'hover', label: 'Hover' },
  { name: 'focus', label: 'Focus' },
  { name: 'nth-of-type(2n)', label: 'Even/Odd' },
]);

// Component-first styling strategy
selectors.setComponentFirst(true);
// When enabled, applying styles prefers component-scoped selectors
```

## Parser

```js
const parser = editor.Parser;

// Parse HTML string into component definitions
const components = parser.parseHtml('<div class="test"><span>Hello</span></div>', {
  allowScripts: false,
  htmlType: 'text/html',
  keepEmptyTextNodes: false,
  asDocument: false,
});

// Parse CSS into rule definitions
const rules = parser.parseCss('.test { color: red; }');

// Intercept parsing
editor.on('parse:html:before', (input, options) => {
  // Modify input before parsing
});
editor.on('parse:css', (rules) => {
  // Post-process parsed CSS rules
});
```

## Keymaps

```js
editor.Keymaps.add('my-shortcut', 'ctrl+shift+k', (editor) => {
  editor.runCommand('my-command');
}, { prevent: true });  // prevent default browser behavior

// Multi-key alternatives (comma-separated)
editor.Keymaps.add('undo-alt', '⌘+z, ctrl+z', (editor) => {
  editor.runCommand('core:undo');
});
```

## Rich Text Editor

```js
const rte = editor.RichTextEditor;

// Add custom formatting action
rte.add('strikethrough', {
  icon: '<s>S</s>',
  attributes: { title: 'Strikethrough' },
  result: (rte) => rte.exec('strikethrough'),
});

// Access toolbar element
rte.getToolbarEl();

// Events
editor.on('rte:enable', (rte) => { /* text editing started */ });
editor.on('rte:disable', () => { /* text editing ended */ });
```

## I18n

```js
// Import built-in locale
import zh from 'grapesjs/locale/zh';

grapesjs.init({
  i18n: {
    locale: 'zh',
    detectLocale: true,   // auto-detect browser language (default: true)
    messages: { zh },
  },
});

// Dynamic updates
editor.I18n.addMessages({
  zh: {
    styleManager: {
      sectors: { dimension: '尺寸' },
      properties: { 'font-size': '字体大小' },
    },
  },
});

// Plugin i18n pattern
const myPlugin = (editor) => {
  editor.I18n.addMessages({
    en: { myPlugin: { label1: 'English' } },
    zh: { myPlugin: { label1: '中文' } },
  });
};
```

## Canvas Spots

Spots are visual markers on the canvas with 5 built-in types:

| Type | Purpose |
|------|---------|
| `select` | Selection border around selected component |
| `resize` | Resize handles on selected component |
| `target` | Drop target highlight during drag |
| `hover` | Hover highlight |
| `spacing` | Spacing/margin guides |

```js
// Disable specific built-in spots
grapesjs.init({
  canvas: {
    spots: { hover: false, target: false },
  },
});

// Custom spot
editor.Canvas.addSpot({
  type: 'custom',
  component: selectedComp,
  // Spot auto-updates position when component moves
});
```

**Note:** Spot containers use `pointer-events: none` by default. If your custom spot needs mouse interaction, set `pointer-events: auto` explicitly.

## Important Gotchas

### Pages key naming
Page config uses **singular** keys: `component` and `styles` (NOT `components`/`style`). This differs from the editor-level config.

### Component type stack
Types form an inheritance chain: `default` → `text` → `image` → custom. Use `extendFn: ['init']` to inherit specific model methods, and `extendFnView: ['onRender']` for view methods.

### data-gjs-* attributes in blocks
When using HTML string blocks, add `data-gjs-*` attributes to control behavior:
```html
<div data-gjs-type="my-type" data-gjs-draggable=".container">
  ...
</div>
```

### Block content forms
Blocks support three content forms:
```js
// 1. Component object (recommended)
content: { type: 'my-type' }

// 2. HTML string
content: '<div>Hello</div>'

// 3. Mixed array
content: [{ type: 'my-type' }, '<span>Extra</span>']
```

### Style Manager built-in property types
`integer`, `number`, `slider`, `color`, `select`, `composite` (shorthand for multi-value props like margin), `stack` (text-shadow, box-shadow), `file`.

### Storage data enrichment
Use events to add metadata before saving:
```js
editor.on('storage:store', (data) => {
  data.savedAt = new Date().toISOString();
  data.userId = getCurrentUserId();
});
```

### Component additional properties
Besides the commonly used ones, components also support:
- `selectable: true` — can be selected on canvas
- `hoverable: true` — shows hover highlight
- `badgable: true` — shows component type badge
- `layerable: true` — appears in Layers panel
- `locked: false` — prevents move/delete

## Key Principles

1. **Component types > raw HTML.** Always register custom types with `addType` and reference them in blocks via `{ type: 'my-type' }`. This gives you traits, styles, lifecycle hooks, and proper serialization.

2. **`onRender` is for canvas-only rendering.** It does NOT affect exported output. For export-friendly content, set `components` and `styles` in the model defaults.

3. **Events over polling.** Use the event system (`editor.on`) to react to changes rather than checking state in intervals.

4. **Storage is pluggable.** Use `editor.Storage.add()` for any backend. The editor handles serialization; your storage only needs `load()` and `store()`.

5. **Traits for user-facing props, Style Manager for CSS.** Traits map to HTML attributes and data; Style Manager properties map to CSS rules.

6. **Use `*:custom` events for framework integration.** Replace any default GrapesJS UI (traits, style props, blocks, assets, modals) with your framework's components via the `*:custom` event pattern.

## References

For deep-dives into specific modules, see the reference files:
- `references/api-complete.md` — Complete API reference with all methods, events, and configuration options for all 20 modules
