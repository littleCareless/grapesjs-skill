# GrapesJS Skill for Claude Code

A [Claude Code](https://claude.ai/code) skill that turns Claude into a GrapesJS expert — covering all 20 module APIs, React integration, and best practices for building web page builders and visual editors.

## What It Covers

- **All 20 GrapesJS modules**: Editor, Components (addType, Symbols), Blocks, Commands, Panels, Style Manager, Traits, Canvas, Storage, Pages, Asset Manager, Selector Manager, Device Manager, Undo Manager, Parser, Keymaps, Modal, Rich Text Editor, I18n, and Emitter/Events
- **React integration**: `@grapesjs/react` provider pattern (GjsEditor, WithEditor, Canvas, useEditor)
- **Custom development**: Component types, trait types, block registration, command lifecycle, plugin architecture
- **CSS theming**: Style Manager sectors, custom property types, Catppuccin Mocha example

## Install

```bash
claude skill add --url https://github.com/littleCareless/grapesjs-skill
```

Or manually copy the skill to your local skills directory:

```bash
git clone https://github.com/littleCareless/grapesjs-skill ~/.claude/skills/grapesjs
```

## Usage

Once installed, the skill activates automatically when you work with GrapesJS in Claude Code. It triggers on keywords like `GrapesJS`, `web builder`, `page builder`, `drag-and-drop editor`, `WYSIWYG`, `visual HTML editor`, etc.

Example prompts:

```
"帮我创建一个自定义 pricing-card 组件，带 traits 配置"
"How do I set up GrapesJS with React and localStorage storage?"
"Add a custom color-palette trait type"
```

## Repository Structure

```
grapesjs-skill/
├── SKILL.md                  # Main skill definition (978 lines)
├── evals/
│   └── evals.json            # Evaluation test cases
├── references/
│   └── api-complete.md       # Complete API reference (757 lines)
└── README.md
```

## License

MIT
