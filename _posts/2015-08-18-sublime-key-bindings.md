
Keyboard shortcuts are a programmer's best friend, and Sublime Text lets you write your own. I use these five below on a daily basis, and you're going to want to, too. Here's the setup:

1. Open Sublime Text
2. Type `command + shift + p` to open your Command Pallette (aka `super+shift+p` in Sublime lingo)
3. Select Preferences: Key Bindings - User
4. Insert the following into this file:

```
[
  { "keys": ["super+shift+w"], "command": "close_all"},
  { "keys": ["ctrl+alt+shift+down"], "command": "goto_definition" },
  { "keys": ["alt+c"], "command": "insert_snippet", "args": { "contents": "console.log(${1:}$SELECTION);${0}" } },
  { "keys": ["alt+d"], "command": "insert_snippet", "args": { "contents": "debugger;" } },
  { "keys": ["alt+b"], "command": "insert_snippet", "args": { "contents": "binding.pry" } }
]
```

Quick summary of each key binding:

##### `super+shift+w` => close_all
Closes all open tabs.
<hr>
##### `ctrl+alt+shift+down` => goto_definition
Navigates to selected method or class definition (based on your cursor's position).
<hr>
##### `alt+c` => insert_snippet 'console.log();'
Inserts `console.log();` at your current cursor position.
<hr>
##### `alt+d` => insert_snippet 'debugger;'
Inserts `debugger;` at your current cursor position.
<hr>
##### `alt+b` => insert_snippet 'binding.pry'
Inserts `binding.pry` at your current cursor position.
<hr>
