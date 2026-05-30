# Neovim Plugin Engineering Notes
# Phase 1 — Lua Foundations
# Phase 2 — Neovim API Foundations (Beginning)

---

# Lesson 1 — Variables, Scope, Functions

## Core Idea

Lua plugins are just Lua modules.  
Bad scope discipline creates globals that leak into Neovim’s runtime and can conflict with other plugins.

Rule:

```lua
Always use local unless you explicitly need global scope.
```

---

# Variables

## Correct

```lua
local greeting = "hello"
```

## Wrong

```lua
greeting = "hello"
```

Problem:
- Creates global variable
- Pollutes runtime namespace
- Other plugins can overwrite it
- Hard debugging later

---

# Block Scope

```lua
local x = 10

do
  local x = 20
  print(x) -- 20
end

print(x) -- 10
```

Mental model:
- Inner `local` shadows outer variable
- Exists only inside block

---

# Functions

## Standard Function

```lua
local function greet(name)
  return "Hello, " .. name
end
```

## Anonymous Function

```lua
local greet = function(name)
  return "Hello, " .. name
end
```

Both are equivalent.

---

# Multiple Return Values

Very important for Neovim APIs.

```lua
local function get_cursor()
  return 5, 12
end

local row, col = get_cursor()
```

Neovim APIs often return:
- row, col
- ok, result
- width, height

Must be comfortable destructuring immediately.

---

# Nil and Default Config Pattern

Most important plugin config pattern.

```lua
opts = opts or {}

local width = opts.width or 80
```

Mental model:
- `nil` means “missing”
- `or` provides fallback/default

---

# Universal Plugin Setup Pattern

```lua
local function setup(opts)
  opts = opts or {}

  local config = {
    width = opts.width or 80,
    title = opts.title or "My Plugin",
  }
end
```

You will write this constantly in plugins.

---

# Mistakes You Made

## Mistake 1 — Re-declaring Parameters

Wrong:

```lua
function make_window_config(opts)
  local opts = opts or {}
end
```

Problem:
- Shadowing warning
- Redundant
- Bad style

Correct:

```lua
local function make_window_config(opts)
  opts = opts or {}
end
```

---

## Mistake 2 — Global Function

Wrong:

```lua
function make_window_config(opts)
```

Problem:
- Creates global function
- Pollutes namespace

Correct:

```lua
local function make_window_config(opts)
```

---

## Mistake 3 — Incomplete Config Return

Wrong:

```lua
return {
  width = width,
  title = title
}
```

Problem:
- Missing required fields
- Causes silent plugin bugs

Correct:

```lua
return {
  width = width,
  height = height,
  title = title,
  relative = relative,
}
```

---

# pairs() vs ipairs()

## pairs()

Dictionary iteration.
Order NOT guaranteed.

```lua
for k, v in pairs(tbl) do
```

Use for:
- config tables
- dictionaries
- objects

---

## ipairs()

Ordered numeric iteration.

```lua
for i, v in ipairs(tbl) do
```

Use for:
- lists
- UI entries
- arrays

---

# Quick Recall

- always use `local`
- `opts = opts or {}`
- config defaults use `or`
- multiple returns are common
- `pairs()` unordered
- `ipairs()` ordered
- never leak globals

---

# Lesson 2 — Tables

## Core Idea

Tables are EVERYTHING in Lua.

They are:
- arrays
- dictionaries
- objects
- modules
- classes
- plugin state

Master tables = master Lua plugin development.

---

# Dictionary Style Table

```lua
local config = {
  width = 80,
  title = "Plugin",
}
```

Access:

```lua
config.width
config["width"]
```

---

# Array Style Table

```lua
local items = { "a", "b", "c" }
```

Important:
- Lua arrays start at 1
- NOT 0

---

# Table Functions

## Insert

```lua
table.insert(items, "x")
```

## Remove

```lua
table.remove(items, 1)
```

---

# Tables Are References

Critical concept.

```lua
local a = { width = 80 }
local b = a

b.width = 999

print(a.width) -- 999
```

Both variables point to SAME table.

---

# Copying Tables

```lua
local function copy_table(original)
  local copy = {}

  for k, v in pairs(original) do
    copy[k] = v
  end

  return copy
end
```

Needed when:
- storing config safely
- avoiding accidental mutation

---

# Nested Tables

Real plugins use nested state heavily.

```lua
local state = {
  config = {},
  windows = {},
  items = {},
}
```

---

# Functions Inside Tables

Lua “OOP”.

```lua
local Window = {}

function Window.set_title(title)
end
```

Table acts as namespace/module.

---

# Module Pattern

Most important plugin pattern.

```lua
local M = {}

return M
```

---

# Private vs Public

## Private

```lua
local state = {}
local function helper()
```

## Public

```lua
function M.setup()
```

---

# Mistakes You Made

## Mistake 1 — Shadowing Built-in `table`

Wrong:

```lua
function printArrayTable(table)
```

Problem:
- Overrides Lua built-in library
- `table.insert()` breaks inside scope

Correct:

```lua
function printArrayTable(items)
```

Never use:
- table
- string
- math
- io
- os

as variable names.

---

## Mistake 2 — Printing Instead of Returning

Wrong:

```lua
function M.count()
  print(#tasks)
end
```

Problem:
- Caller cannot use value programmatically

Correct:

```lua
function M.count()
  return #tasks
end
```

Rule:
- Functions return values
- Callers decide display

---

## Mistake 3 — Returning Table Instead of Printing

Wrong:

```lua
function M.list()
  return tasks
end
```

Spec required printing.

Correct:

```lua
function M.list()
  printArrayTable(tasks)
end
```

---

# Mental Model

```text
Private state → local
Public API → attached to M
```

---

# Quick Recall

- tables are references
- arrays start at 1
- pairs() unordered
- ipairs() ordered
- local M = {}
- return M
- private state stays local
- return values instead of printing

---

# Lesson 3 — Modules & require()

## Core Idea

`require()` loads Lua modules once and caches them.

---

# Runtimepath

Neovim searches:

```text
~/.config/nvim/lua/
```

Example:

```text
lua/tasklist/init.lua
```

Loaded via:

```lua
require("tasklist")
```

---

# Dot Mapping

```lua
require("tasklist.utils")
```

maps to:

```text
lua/tasklist/utils.lua
```

---

# Standard Plugin Structure

```text
tasklist/
├── init.lua
├── config.lua
├── state.lua
└── utils.lua
```

---

# File Responsibilities

## init.lua

Public API.

Only user-facing file.

---

## config.lua

Stores:
- defaults
- setup logic
- active config

---

## state.lua

Stores mutable runtime state.

---

## utils.lua

Pure helper functions.

---

# Why Split Files?

Benefits:
- debugging easier
- isolated responsibilities
- scalable architecture
- maintainability

---

# require() Caching

Important.

```lua
require("tasklist")
```

runs ONCE.

Future requires:
- return cached module
- do not re-execute file

---

# Reload During Development

```lua
package.loaded["tasklist"] = nil
```

Then require again.

---

# Mistakes You Made

## Mistake 1 — No Input Validation

Wrong:

```lua
table.remove(tasks, index)
```

without checking index.

Correct:

```lua
if type(index) ~= "number" then
```

---

## Mistake 2 — Config Stored Publicly

Wrong:

```lua
M.config = {}
```

Problem:
- callers can overwrite internal config

Correct:

```lua
local config = {}
```

---

## Mistake 3 — Returning Raw Table in state.lua

Wrong:

```lua
return tasks
```

Problem:
- hard to extend later

Correct:

```lua
local M = {}

M.tasks = {}

return M
```

---

# next(config) == nil

Checks if table is empty.

Used to detect:
- setup() never called

---

# Mental Model

```text
init.lua = public API
config.lua = configuration
state.lua = runtime state
utils.lua = helpers
```

---

# Quick Recall

- require() caches modules
- modules return M
- split responsibilities
- config should stay private
- validate all input
- runtime state isolated

---

# Lesson 4 — Error Handling with pcall

## Core Idea

Plugins must NOT crash Neovim.

Use:
- error()
- pcall()
- vim.notify()

correctly.

---

# pcall()

Protected call.

```lua
local ok, result = pcall(fn)
```

Returns:
- true + result
- false + error_message

---

# error()

Throws exception.

```lua
error("something failed")
```

Should be used in:
- internal logic
- lower layers

---

# vim.notify()

User-facing messaging.

```lua
vim.notify("message", vim.log.levels.WARN)
```

Levels:
- INFO
- WARN
- ERROR

---

# Correct Architecture Pattern

## Internal Function

```lua
local function safe_divide(a, b)
  error(...)
end
```

No UI concerns.

---

## Boundary Wrapper

```lua
local function divide(a, b)
  local ok, result = pcall(safe_divide, a, b)

  if not ok then
    vim.notify(result)
  end
end
```

Handles:
- user messaging
- recovery
- graceful failure

---

# Mistakes You Made

## Mistake — One Function Doing Everything

Problem:
- tightly coupled logic + UI
- impossible to reuse internally

Correct:
- internal function throws
- wrapper handles pcall + notify

---

# Plugin Architecture Rule

```text
Internal layers:
- error()
- no UI

Boundary/public API:
- pcall()
- vim.notify()
```

---

# Safe require()

Very common.

```lua
local ok, telescope = pcall(require, "telescope")
```

Prevents crash if dependency missing.

---

# Quick Recall

- error() throws
- pcall() catches
- vim.notify() informs user
- internal logic should not know UI
- boundary functions handle failures

---

# Lesson 5 — OOP Patterns

## Core Idea

Lua has no classes.

Plugins use:
- tables
- metatables
- __index

for OOP.

---

# Singleton Pattern

One shared instance.

```lua
local M = {}
```

Use when:
- one plugin state
- one UI
- one picker

---

# Class Pattern

Multiple independent instances.

---

# __index

```lua
Window.__index = Window
```

If key missing on instance:
- Lua checks Window table

This shares methods.

---

# Constructor Pattern

```lua
function Window.new(opts)
  local self = setmetatable({}, Window)
  return self
end
```

---

# Colon vs Dot

## Colon

```lua
function Window:open()
```

self passed automatically.

---

## Dot

```lua
function Window.open(self)
```

Must pass self manually.

---

# Mistakes You Made

## Mistake 1 — Type Check Order

Wrong:

```lua
if #name == 0 or type(name) ~= "string"
```

Problem:
- crashes on non-string

Correct:

```lua
if type(name) ~= "string" or #name == 0
```

Always validate type FIRST.

---

## Mistake 2 — Printing Instead of Returning

Wrong:

```lua
function Buffer:info()
  print(...)
end
```

Correct:

```lua
function Buffer:info()
  return ...
end
```

---

## Mistake 3 — Missing notify level

Wrong:

```lua
vim.notify("updated")
```

Correct:

```lua
vim.notify("updated", vim.log.levels.INFO)
```

---

# Ternary Pattern

Lua idiom:

```lua
local msg = val and "locked" or "unlocked"
```

Used constantly in plugins.

---

# Quick Recall

- __index shares methods
- new() creates instances
- colon passes self
- singleton = one shared state
- class pattern = many instances
- type check BEFORE operations

---

# Lesson 6 — Neovim API Foundations

## Core Idea

Three ways to interact with Neovim:

```text
vim.api  → preferred
vim.fn   → Vimscript fallback
vim.cmd  → last resort
```

---

# vim.api

Raw Neovim API.

Preferred for plugins.

---

# vim.fn

Calls Vimscript functions.

Use only if API equivalent missing.

---

# vim.cmd

Runs Vimscript commands.

Avoid when possible.

---

# Buffers

Everything in Neovim revolves around buffers.

---

# Creating Scratch Buffer

```lua
local buf = vim.api.nvim_create_buf(false, true)
```

Arguments:

```text
listed  = false
scratch = true
```

Correct for plugin UI buffers.

---

# Writing Lines

```lua
vim.api.nvim_buf_set_lines(buf, 0, -1, false, {
  "line 1"
})
```

---

# Reading Lines

```lua
vim.api.nvim_buf_get_lines(buf, 0, -1, false)
```

---

# Buffer Validation

Critical.

```lua
vim.api.nvim_buf_is_valid(buf)
```

Buffers can disappear anytime.

---

# Buffer Options

```lua
vim.api.nvim_buf_set_option(buf, "modifiable", false)
```

Modern:

```lua
vim.bo[buf].modifiable = false
```

---

# Current Buffer

```lua
vim.api.nvim_get_current_buf()
```

---

# Cursor Position

```lua
vim.api.nvim_win_get_cursor(0)
```

Returns:
- row = 1-based
- col = 0-based

Important inconsistency.

---

# Mistakes You Made

## Mistake 1 — Wrong Line Count

Wrong:

```lua
nvim_buf_get_lines(..., 0, 13)
```

Problem:
- only loads first 13 lines
- not real line count

Correct:

```lua
vim.api.nvim_buf_line_count(buf)
```

---

## Mistake 2 — Manual Slicing Helper

You wrote extra helper.

Unnecessary because:
- API already supports ranges

Correct:

```lua
nvim_buf_get_lines(buf, 0, 5, false)
```

---

## Mistake 3 — Huge String Concatenation

Wrong:

```lua
print("A" .. "B" .. "C")
```

Hard to maintain.

Correct:

```lua
string.format("ID: %d", id)
```

---

# Plugin Engineering Rules

## Rule 1

Know API capabilities before writing helpers.

---

## Rule 2

Use `string.format` for templated output.

---

## Rule 3

Always validate buffer existence.

---

# Quick Recall

- vim.api preferred
- buffers are core primitive
- scratch buffers = false, true
- validate buffers
- use API ranges
- string.format > concatenation

---

# Overall Mental Models

## Plugin Architecture

```text
Public API
    ↓
Boundary layer (pcall + notify)
    ↓
Internal logic
    ↓
State/config/utils
```

---

## Lua Architecture

```text
Tables are everything
Modules return tables
State should stay private
```

---

## Neovim Architecture

```text
Buffers hold text
Windows display buffers
Plugins manipulate buffers/windows
```

---

# Most Important Rules So Far

```text
1. Always use local
2. Validate inputs early
3. Return values instead of printing
4. Separate logic from UI
5. Use pcall at boundaries
6. Keep state private
7. require() caches modules
8. Buffers can become invalid
9. Use vim.api first
10. Learn API before writing helpers
```
