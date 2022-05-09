---
description: https://marketplace.visualstudio.com/items?itemName=vscodevim.vim
---

# VS Code Vim 速记

VS Code 的 [Vim 扩展](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim)，除了复现标准的 Vim 键位之外，默认启用了几个插件，在此简单罗列平时常用的几个。

### vim-surround <a href="#vim-surround" id="vim-surround"></a>

Based on [surround.vim](https://github.com/tpope/vim-surround), the plugin is used to work with surrounding characters like parentheses, brackets, quotes, and XML tags.

`t` or `<` as `<desired>` or `<existing>` will enter tag entry mode. Using `<CR>` instead of `>` to finish changing a tag will preserve any existing attributes.

| Surround Command           | Description                                              |
| -------------------------- | -------------------------------------------------------- |
| `y s <motion> <desired>`   | Add `desired` surround around text defined by `<motion>` |
| `d s <existing>`           | Delete `existing` surround                               |
| `c s <existing> <desired>` | Change `existing` surround to `desired`                  |
| `S <desired>`              | Surround when in visual modes (surrounds full selection) |

Some examples:

* `"test"` with cursor inside quotes type `cs"'` to end up with `'test'`
* `"test"` with cursor inside quotes type `ds"` to end up with `test`
* `"test"` with cursor inside quotes type `cs"t` and enter `123>` to end up with `<123>test</123>`

### vim-commentary <a href="#vim-commentary" id="vim-commentary"></a>

Similar to [vim-commentary](https://github.com/tpope/vim-commentary), but uses the VS Code native _Toggle Line Comment_ and _Toggle Block Comment_ features.

Usage examples:

* `gc` - toggles line comment. For example `gcc` to toggle line comment for current line and `gc2j` to toggle line comments for the current line and the next two lines.
* `gC` - toggles block comment. For example `gCi)` to comment out everything within parentheses.

### vim-indent-object <a href="#vim-indent-object" id="vim-indent-object"></a>

Based on [vim-indent-object](https://github.com/michaeljsmith/vim-indent-object), it allows for treating blocks of code at the current indentation level as text objects. Useful in languages that don't use braces around statements (e.g. Python).

Provided there is a new line between the opening and closing braces / tag, it can be considered an agnostic `cib`/`ci{`/`ci[`/`cit`.

| Command        | Description                                                                                          |
| -------------- | ---------------------------------------------------------------------------------------------------- |
| `<operator>ii` | This indentation level                                                                               |
| `<operator>ai` | This indentation level and the line above (think `if` statements in Python)                          |
| `<operator>aI` | This indentation level, the line above, and the line after (think `if` statements in C/C++/Java/etc) |

### VSCodeVim tricks! <a href="#vscodevim-tricks" id="vscodevim-tricks"></a>

* `gd` - jump to definition.
* `gq` - on a visual selection reflow and wordwrap blocks of text, preserving commenting style. Great for formatting documentation comments.
* `gb` - adds another cursor on the next word it finds which is the same as the word under the cursor.
* `af` - visual mode command which selects increasingly large blocks of text. For example, if you had "blah (foo \[bar 'ba|z'])" then it would select 'baz' first. If you pressed `af` again, it'd then select \[bar 'baz'], and if you did it a third time it would select "(foo \[bar 'baz'])".
* `gh` - equivalent to hovering your mouse over wherever the cursor is. Handy for seeing types and error messages without reaching for the mouse!
