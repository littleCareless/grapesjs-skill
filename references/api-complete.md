# GrapesJS Complete API Reference

## Table of Contents

1. [Editor API](#editor-api)
2. [Components API](#components-api)
3. [Blocks API](#blocks-api)
4. [Commands API](#commands-api)
5. [Panels API](#panels-api)
6. [Style Manager API](#style-manager-api)
7. [Traits API](#traits-api)
8. [Canvas API](#canvas-api)
9. [Storage Manager API](#storage-manager-api)
10. [Pages API](#pages-api)
11. [Asset Manager API](#asset-manager-api)
12. [Selector Manager API](#selector-manager-api)
13. [Device Manager API](#device-manager-api)
14. [Undo Manager API](#undo-manager-api)
15. [Parser API](#parser-api)
16. [Keymaps API](#keymaps-api)
17. [Modal API](#modal-api)
18. [Rich Text Editor API](#rich-text-editor-api)
19. [I18n API](#i18n-api)
20. [Emitter / Events](#emitter--events)

---

## Editor API

**Access:** `const editor = grapesjs.init(config)`

### Initialization Config

```js
grapesjs.init({
  container: '#gjs' | HTMLElement,
  fromElement: boolean,          // parse existing HTML in container
  height: string,                // '100%', '900px'
  width: string,                 // 'auto', '100%'
  mediaCondition: 'max-width' | 'min-width',  // responsive direction (default: 'max-width')

  // Module configs
  domComponents: {},
  blockManager: { appendTo: '#blocks' },
  panels: { defaults: [] },
  commands: { defaults: [] },
  styleManager: { sectors: [] },
  traitManager: { appendTo: '#traits' },
  selectorManager: { appendTo: '#selectors' },
  assetManager: { upload: '/upload' },
  canvas: { customSpots: {} },
  pageManager: { pages: [] },
  storageManager: { type: 'local' },
  deviceManager: { devices: [] },
  undoManager: {},
  keymaps: {},
  parser: {},
  i18n: { locale: 'en', messages: {} },
});
```

### Core Methods

| Method | Description |
|--------|-------------|
| `getConfig(prop?)` | Get config object or specific property |
| `getHtml(opts?)` | Get HTML. `opts: { component, cleanId }` |
| `getCss(opts?)` | Get CSS. `opts: { component, json, avoidProtected, onlyMatched, keepUnusedStyles }` |
| `getJs(opts?)` | Get JS output |
| `getProjectData()` | Full project as JSON |
| `loadProjectData(data, opts?)` | Restore project from JSON |
| `getComponents()` | Get root component collection |
| `setComponents(components)` | Set root components |
| `addComponents(components, opts)` | Add components |
| `getStyle()` | Get all CSS rules |
| `setStyle(style)` | Set CSS rules |
| `addStyle(style)` | Add CSS rules |
| `getSelected()` | Get currently selected component |
| `getSelectedAll()` | Get all selected components |
| `select(el, opts)` | Select a component |
| `selectAdd(el)` | Add to selection |
| `selectRemove(el)` | Remove from selection |
| `selectToggle(el)` | Toggle selection |
| `setDevice(name)` | Switch device |
| `getDevice()` | Current device name |
| `runCommand(id, options)` | Execute a command |
| `stopCommand(id, options)` | Stop a stateful command |
| `store(options?)` | Trigger save |
| `load(options?)` | Trigger load |
| `getDirtyCount()` | Number of unsaved changes |
| `clearDirtyCount()` | Reset dirty counter |
| `refresh(opts)` | Refresh editor |
| `destroy()` | Destroy editor instance |
| `render()` | Re-render editor |
| `onReady(clb)` | Callback when editor is ready |
| `t(key, opts?)` | Translate i18n key |
| `html` | Tagged template for JSX-like syntax |

### Editor Events

`update`, `undo`, `redo`, `load`, `project:load`, `project:loaded`, `project:get`, `log`, `destroy`, `destroyed`

---

## Components API

**Access:** `editor.DomComponents`

### Methods

| Method | Description |
|--------|-------------|
| `getWrapper()` | Root wrapper component |
| `getComponents()` | Root children collection |
| `addComponent(comp, opt)` | Add component to root |
| `clear(opts)` | Clear all components |
| `addType(type, methods)` | Define custom component type |
| `getType(type)` | Get type definition |
| `removeType(id)` | Remove a type |
| `getTypes()` | All registered types |
| `isComponent(obj)` | Check if object is a component |
| `addSymbol(component)` | Create reusable synced component |
| `getSymbols()` | Get all symbols |
| `detachSymbol(component)` | Detach from symbol sync |
| `getSymbolInfo(component, opts)` | Symbol metadata |
| `canMove(target, source, index)` | Check if move is valid. Returns `{ result, source, target, reason }` |

### Component Model Properties

```js
{
  tagName: 'div',
  type: 'default',
  removable: true,
  draggable: 'div, section',   // CSS selector
  droppable: true,
  badgable: true,
  stylable: true,
  copyable: true,
  content: '',
  style: {},
  attributes: {},
  traits: [],
  components: '',              // child components
  styles: '',                  // scoped CSS string
}
```

### Component Instance Methods

```js
comp.get(prop)
comp.set(prop, value)
comp.props()                    // all properties
comp.getAttributes()
comp.setAttributes(attrs)
comp.addAttributes(attrs)
comp.removeAttributes(...names)
comp.components()               // children collection
comp.components(html)           // set children
comp.append(child)
comp.find(selector)             // query descendants
comp.closest(selector)          // traverse up
comp.toHTML()                   // export
comp.getEl()                    // live DOM (canvas only)
comp.getView()
comp.remove()
comp.replaceWith(other)
comp.setStyle(styles)
comp.getStyle()
comp.getParent()
comp.is(instance)
```

### Component Events

`component:create`, `component:mount`, `component:add`, `component:remove`, `component:remove:before`, `component:clone`, `component:update`, `component:update:{prop}`, `component:styleUpdate`, `component:styleUpdate:{prop}`, `component:selected`, `component:deselected`, `component:toggled`, `component:type:add`, `component:type:update`, `component:drag:start`, `component:drag`, `component:drag:end`, `component:resize`

---

## Blocks API

**Access:** `editor.Blocks` or `editor.BlockManager`

### Config

```js
blockManager: {
  appendTo: '#blocks',
  blocks: [
    { id: 'section', label: 'Section', category: 'Layout', content: { type: 'section' } },
  ],
}
```

### Methods

| Method | Description |
|--------|-------------|
| `add(id, opts)` | Register a block |
| `get(id)` | Get block by id |
| `getAll()` | All blocks |
| `remove(id)` | Remove block |
| `getConfig()` | Module config |

### Block Options

```js
{
  id: 'my-block',
  label: 'Block Name',            // supports HTML
  category: 'Category Name',      // or { name, order, open }
  select: true,                    // auto-select after placement
  activate: true,                  // trigger active event
  content: { type: 'my-type' },   // component definition (recommended)
  // content: '<div>HTML</div>',   // HTML string (alternative)
  attributes: { class: 'icon-class' },
}
```

### Block Events

`block:add`, `block:remove`, `block:drag:start`, `block:drag`, `block:drag:stop`, `block:custom`

---

## Commands API

**Access:** `editor.Commands`

### Methods

| Method | Description |
|--------|-------------|
| `add(id, handler)` | Register command |
| `get(id)` | Get command |
| `getAll()` | All commands |
| `remove(id)` | Remove command |
| `extend(id, methods)` | Extend existing command |
| `run(id, options)` | Execute command |
| `stop(id, options)` | Stop stateful command |
| `isActive(id)` | Check if stateful command is running |
| `getConfig()` | Module config |

### Built-in Commands

`core:canvas-clear`, `core:component-delete`, `core:component-enter`, `core:component-exit`, `core:component-next`, `core:component-prev`, `core:component-outline`, `core:component-offset`, `core:component-select`, `core:copy`, `core:paste`, `core:preview`, `core:fullscreen`, `core:open-code`, `core:open-layers`, `core:open-styles`, `core:open-traits`, `core:open-blocks`, `core:open-assets`, `core:undo`, `core:redo`

### Command Events

`command:run`, `command:run:{id}`, `command:run:before:{id}`, `command:stop`, `command:stop:{id}`, `command:stop:before:{id}`, `abort:{id}`

---

## Panels API

**Access:** `editor.Panels`

### Methods

| Method | Description |
|--------|-------------|
| `getPanels()` | All panels |
| `getPanelsEl()` | Panels container element |
| `addPanel(panel)` | Add panel |
| `removePanel(panel)` | Remove panel |
| `getPanel(id)` | Get panel by id |
| `addButton(panelId, button)` | Add button to panel |
| `removeButton(panelId, buttonId)` | Remove button |
| `getButton(panelId, id)` | Get button |

### Panel Config

```js
{
  id: 'panel-id',
  el: '.container',        // optional existing element
  resizable: { maxDim, minDim, tc, cl, cr, bc, keyWidth },
  buttons: [{ id, className, label, command, active, togglable, context }],
}
```

---

## Style Manager API

**Access:** `editor.StyleManager`

### Config

```js
styleManager: {
  appendTo: '.styles-container',
  sectors: [
    {
      name: 'Dimension',
      open: true,
      buildProps: ['width', 'height', 'margin', 'padding'],
      properties: [
        {
          id: 'custom-width',
          type: 'integer',
          name: 'Width',
          property: 'width',
          units: ['px', '%', 'vw'],
          defaults: 'auto',
          min: 0,
        },
      ],
    },
  ],
}
```

### Methods

| Method | Description |
|--------|-------------|
| `addSector(id, sector, opts?)` | Add sector |
| `getSector(id)` | Get sector |
| `getSectors(opts?)` | Get all sectors |
| `removeSector(id)` | Remove sector |
| `addProperty(sectorId, property, opts?)` | Add property to sector |
| `getProperty(sectorId, id)` | Get property |
| `getProperties(sectorId)` | All properties in sector |
| `removeProperty(sectorId, id)` | Remove property |
| `addBuiltIn(prop, definition)` | Add built-in property |
| `getBuiltIn(prop)` | Get built-in property |
| `getBuiltInAll()` | All built-in properties |
| `addType(id, { create, emit, update, destroy })` | Custom property type |

### Style Events

`style:sector:add/remove/update`, `style:property:add/remove/update`, `style:target`, `style:layer:select`, `style:custom`, `style`

---

## Traits API

**Access:** `editor.Traits`

### Built-in Types

`text`, `number`, `checkbox`, `select`, `color`, `button`

### Methods

| Method | Description |
|--------|-------------|
| `addType(id, methods)` | Register custom trait type |
| `getType(id)` | Get trait type |

### Custom Trait Type Methods

```js
{
  createInput({ trait }) → HTMLElement,
  onEvent({ elInput, component, event }),
  onUpdate({ elInput, component }),
  onChange({ trait, component, value }),
  eventCapture: ['change'],    // custom event names to listen
}
```

### Trait Events

`trait:custom`

---

## Canvas API

**Access:** `editor.Canvas`

### Config

```js
canvas: {
  customSpots: { target: true },   // disable built-in spots
}
```

### Methods

| Method | Description |
|--------|-------------|
| `getElement()` | Canvas container element |
| `getFrameEl()` | Main iframe |
| `getWindow()` | iframe window |
| `getDocument()` | iframe document |
| `getBody()` | iframe body |
| `getRect()` | Canvas rect data |
| `setCustomBadgeLabel(fn)` | Custom badge text |
| `setZoom(value)` | Set zoom (0-100) |
| `getZoom()` | Current zoom |
| `setCoords(x, y)` | Scroll position |
| `getCoords()` | Current coords |
| `scrollTo(el, opts)` | Scroll to element |
| `hasFocus()` | Is canvas focused |
| `startDrag(dragSource)` | Start custom drag |
| `endDrag()` | End custom drag |
| `getLastDragResult()` | Last drag result |
| `addSpot(opts)` | Add visual marker |
| `getSpots(filter?)` | Get spots |
| `removeSpots(filter)` | Remove spots |
| `hasCustomSpot(type)` | Check custom spot |
| `refresh(opts)` | Refresh canvas |
| `getWorldRectToScreen(rect)` | World → screen coords |

### Canvas Events

`canvas:dragenter`, `canvas:dragover`, `canvas:dragend`, `canvas:dragdata`, `canvas:drop`, `canvas:spot`, `canvas:spot:add/update/remove`, `canvas:coords`, `canvas:zoom`, `canvas:pointer`, `canvas:refresh`, `canvas:frame:load`, `canvas:frame:load:head`, `canvas:frame:load:body`, `canvas:frame:unload`

---

## Storage Manager API

**Access:** `editor.Storage`

### Config

```js
storageManager: {
  type: 'local' | 'remote',
  autosave: true,
  autoload: true,
  stepsBeforeSave: 1,
  options: {
    local: { key: 'gjs-project' },
    remote: {
      urlStore: '/api/store',
      urlLoad: '/api/load',
      headers: {},
    },
  },
}
```

### Methods

| Method | Description |
|--------|-------------|
| `getConfig()` | Module config |
| `isAutosave()` | Check autosave |
| `setAutosave(val)` | Toggle autosave |
| `getStepsBeforeSave()` | Steps threshold |
| `setStepsBeforeSave(val)` | Set threshold |
| `add(type, storage)` | Add custom storage `{ load, store }` |
| `get(type)` | Get storage |
| `getStorages()` | All storages |
| `getCurrent()` | Current storage type |
| `setCurrent(type)` | Switch storage |
| `getStorageOptions(type)` | Storage options |
| `store(data, opts?)` | Save data |
| `load(opts?)` | Load data |

### Storage Events

`storage:start`, `storage:start:store`, `storage:start:load`, `storage:load`, `storage:store`, `storage:after`, `storage:end`, `storage:end:store`, `storage:end:load`, `storage:error`, `storage:error:store`, `storage:error:load`

---

## Pages API

**Access:** `editor.Pages`

### Config

```js
pageManager: {
  pages: [
    { id: 'home', name: 'Home', component: '<div>Home</div>', styles: '.x{color:red}' },
  ],
}
```

### Methods

| Method | Description |
|--------|-------------|
| `getAll()` | All pages |
| `add(props, opts?)` | Add page |
| `remove(page, opts?)` | Remove page |
| `move(page, opts)` | Move page `{ at: index }` |
| `get(id)` | Get page by id |
| `getMain()` | First page |
| `getAllWrappers()` | All wrapper components |
| `select(page, opts?)` | Select (switch canvas) |
| `getSelected()` | Current page |

### Page Events

`page:add`, `page:remove`, `page:select(page, prevPage)`, `page:update(page, changes)`, `page`

---

## Asset Manager API

**Access:** `editor.AssetManager` or `editor.Assets`

### Config

```js
assetManager: {
  upload: '/api/upload',
  uploadName: 'files',
  headers: {},
  multiUpload: false,
}
```

### Methods

| Method | Description |
|--------|-------------|
| `add(asset, opts?)` | Add asset |
| `get(id)` | Get asset |
| `getAll()` | All assets |
| `getAllVisible()` | Visible assets |
| `remove(asset)` | Remove asset |
| `close()` | Close asset panel |

### Asset Events

`asset:add`, `asset:remove`, `asset:upload`, `asset:upload:end`, `asset:upload:error`, `asset:custom`

---

## Selector Manager API

**Access:** `editor.Selectors`

### Methods

| Method | Description |
|--------|-------------|
| `add(props, opts?)` | Add selector (string or object) |
| `get(name, type?)` | Get selector |
| `remove(selector, opts?)` | Remove |
| `rename(selector, name, opts?)` | Rename |
| `getAll()` | All selectors |
| `setState(val)` | Set active state |
| `getState()` | Get active state |
| `getStates()` | All states |
| `setStates(states)` | Set states `[{name, label}]` |
| `getSelected()` | Selected selectors |
| `getSelectedAll()` | All selected |
| `addSelected(props)` | Add to selection |
| `removeSelected(sel)` | Remove from selection |
| `getSelectedTargets()` | Targets (Component or CssRule) |
| `setComponentFirst(val)` | Component-first mode |
| `getComponentFirst()` | Check mode |

### Selector Events

`selector:add`, `selector:remove`, `selector:remove:before`, `selector:update`, `selector:state`, `selector:custom({states, selected, container})`, `selector`

---

## Device Manager API

**Access:** `editor.Devices`

### Config

```js
deviceManager: {
  devices: [
    { name: 'Desktop', width: '' },
    { name: 'Mobile', width: '320px', widthMedia: '480px' },
    { id: 'tablet', name: 'Tablet', width: '800px', widthMedia: '810px', height: '600px' },
  ],
}
```

### Methods

| Method | Description |
|--------|-------------|
| `add(props, opts?)` | Add device |
| `get(id)` | Get device |
| `remove(device, opts?)` | Remove |
| `getDevices()` | All devices |
| `select(device, opts?)` | Select device |
| `getSelected()` | Current device |

### Device Events

`device:add`, `device:remove`, `device:select`, `device:update`, `device`

---

## Undo Manager API

**Access:** `editor.UndoManager`

### Methods

| Method | Description |
|--------|-------------|
| `add(entity)` | Track a Model/Collection |
| `remove(entity)` | Stop tracking |
| `removeAll()` | Stop all tracking |
| `start()` | Start tracking |
| `stop()` | Stop tracking |
| `undo()` | Undo last |
| `undoAll()` | Undo everything |
| `redo()` | Redo last |
| `redoAll()` | Redo everything |
| `hasUndo()` | Has undo available |
| `hasRedo()` | Has redo available |
| `isRegistered(obj)` | Is tracked |
| `getStack()` | Change stack |
| `skip(callback)` | Temporarily skip tracking |
| `clear()` | Clear stack |

> Components and CSS rules are auto-tracked.

---

## Parser API

**Access:** `editor.Parser`

### Methods

| Method | Description |
|--------|-------------|
| `getConfig()` | Module config |
| `parseHtml(input, options)` | Parse HTML string |
| `parseCss(input)` | Parse CSS string |

### parseHtml Options

`htmlType`, `allowScripts`, `allowUnsafeAttr`, `allowUnsafeAttrValue`, `keepEmptyTextNodes`, `asDocument`, `detectDocument`, `preParser`, `convertDataGjsAttributesHyphens`

### Parser Events

`parse:html`, `parse:html:before`, `parse:css`, `parse:css:before`, `parse`

---

## Keymaps API

**Access:** `editor.Keymaps`

### Methods

| Method | Description |
|--------|-------------|
| `getConfig()` | Module config |
| `add(id, keys, handler, opts?)` | Add keymap |
| `get(id)` | Get keymap |
| `getAll()` | All keymaps |
| `remove(id)` | Remove keymap |
| `removeAll()` | Remove all |

### Key Format

`ctrl+a`, `⌘+z, ctrl+z` (comma-separated alternatives)

### Options

`opts.force`, `opts.prevent`

### Keymap Events

`keymap:add`, `keymap:remove`, `keymap:emit`, `keymap:emit:{id}`

---

## Modal API

**Access:** `editor.Modal`

### Methods

| Method | Description |
|--------|-------------|
| `setTitle(title)` | Set title (chainable) |
| `getContent()` | Get content |
| `setContent(content)` | Set content (chainable) |
| `open(opts?)` | Open modal |
| `close()` | Close modal |
| `isOpen()` | Check if open |
| `onceClose(fn)` | Run once on close |

### Modal Events

`modal` (for custom modal integration)

---

## Rich Text Editor API

**Access:** `editor.RichTextEditor`

### Methods

| Method | Description |
|--------|-------------|
| `add(name, action)` | Add action |
| `get(name)` | Get action |
| `getAll()` | All actions |
| `remove(name)` | Remove action |
| `run(action)` | Execute action |
| `getToolbarEl()` | Toolbar element |
| `getConfig()` | Module config |

### Action Config

```js
{
  icon: '<b>B</b>',
  attributes: { title: 'Bold' },
  result: (rte) => rte.exec('bold'),
}
```

### RTE Events

`rte:enable`, `rte:disable`, `rte:custom`

---

## I18n API

**Access:** `editor.I18n`

### Config

```js
i18n: {
  locale: 'en',
  localeFallback: 'en',
  messages: {
    en: { hello: 'Hello' },
    zh: { hello: '你好' },
  },
}
```

### Methods

| Method | Description |
|--------|-------------|
| `setLocale(locale)` | Set active locale |
| `getLocale()` | Current locale |
| `getMessages(lang?)` | All messages or specific language |
| `setMessages(msg)` | Replace all messages |
| `addMessages(msg)` | Merge messages |
| `t(key, opts?)` | Translate. `opts: { params, debug, l }` |
| `getConfig()` | Module config |

### I18n Events

`i18n:add`, `i18n:update`, `i18n:locale({ value, valuePrev })`
