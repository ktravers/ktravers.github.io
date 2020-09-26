---
layout: post
title: Sublime Text Key Bindings
tags: [sublime text, recommendations]
---

Keyboard shortcuts are a programmer's best friend, and Sublime Text lets you write your own. I use these five below on a daily basis, and you're going to want to, too. Here's the setup:

1. Open Sublime Text
2. Type `command + shift + p` to open your Command Pallette (aka `super+shift+p` in Sublime lingo)
3. Select Preferences: Key Bindings - User
4. Insert the following into this file:

```json
[
  { "keys": ["super+shift+w"], "command": "close_all"},
  { "keys": ["super+shift+r"], "command": "reveal_in_side_bar" },
  { "keys": ["ctrl+alt+shift+down"], "command": "goto_definition" },
  { "keys": ["super+k"], "command": "insert_snippet", "args": { "contents": "[$SELECTION](${0})" }, "context":
    [
      { "key": "selector", "operator": "equal", "operand": "text.html.markdown" },
      { "key": "selection_empty", "operator": "equal", "operand": false, "match_all": true }
    ]
  },
  { "keys": ["alt+c"], "command": "insert_snippet", "args": { "contents": "console.log(${1:}$SELECTION);${0}" } },
  { "keys": ["alt+d"], "command": "insert_snippet", "args": { "contents": "debugger;" } },
  { "keys": ["alt+b"], "command": "insert_snippet", "args": { "contents": "binding.pry" } }
]
```

Quick summary of each key binding:

##### `super+shift+w` => close_all

Closes all open tabs.

##### `super+shift+r` => reveal_in_side_bar

Opens sidebar with currently open file selected.

##### `ctrl+alt+shift+down` => goto_definition

Navigates to selected method or class definition (based on your cursor's position).

##### `super+k`

Inserts Markdown link around selected text and places cursor inside the `()` (same behavior as [GitHub's `command+k` keyboard shortcut](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/keyboard-shortcuts#comments)).

##### `alt+c` => insert_snippet 'console.log();'

Inserts `console.log();` at your current cursor position.

##### `alt+d` => insert_snippet 'debugger;'

Inserts `debugger;` at your current cursor position.

##### `alt+b` => insert_snippet 'binding.pry'

Inserts `binding.pry` at your current cursor position.

