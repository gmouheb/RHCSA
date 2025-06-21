# Vi and Vim Editors in Linux

## Introduction to Vi and Vim

Vi is a screen-oriented text editor originally created for the Unix operating system. Vim (Vi IMproved) is an enhanced version of the Vi editor with additional features.

## Basic Vi/Vim Commands

### Starting Vi/Vim
```bash
vim filename    # Open a file with vim
vi filename     # Open a file with vi
```

### Modes in Vi/Vim
- **Normal Mode**: Default mode for navigation and manipulation
- **Insert Mode**: For inserting text
- **Command Mode**: For executing commands
- **Visual Mode**: For selecting text

### Switching Between Modes
- Press `i` to enter insert mode
- Press `Esc` to return to normal mode
- Press `:` to enter command mode

### Basic Navigation (Normal Mode)
- `h` - Move cursor left
- `j` - Move cursor down
- `k` - Move cursor up
- `l` - Move cursor right
- `w` - Move to the beginning of the next word
- `b` - Move to the beginning of the previous word
- `0` - Move to the beginning of the line
- `$` - Move to the end of the line
- `gg` - Move to the beginning of the file
- `G` - Move to the end of the file
- `:n` - Move to line n

### Editing Text
- `i` - Insert text at cursor position
- `a` - Append text after cursor position
- `o` - Open a new line below the current line
- `O` - Open a new line above the current line
- `x` - Delete character under cursor
- `dd` - Delete current line
- `yy` - Copy (yank) current line
- `p` - Paste after cursor
- `P` - Paste before cursor
- `u` - Undo last change
- `Ctrl+r` - Redo change

### Saving and Quitting
- `:w` - Save file
- `:q` - Quit (fails if there are unsaved changes)
- `:q!` - Quit without saving
- `:wq` or `ZZ` - Save and quit

## Advanced Features

### Search and Replace
- `/pattern` - Search forward for pattern
- `?pattern` - Search backward for pattern
- `n` - Repeat search in same direction
- `N` - Repeat search in opposite direction
- `:%s/old/new/g` - Replace all occurrences of 'old' with 'new'
- `:s/old/new/g` - Replace all occurrences of 'old' with 'new' in current line

### Working with Multiple Files
- `:e filename` - Edit a file
- `:split filename` - Split window horizontally
- `:vsplit filename` - Split window vertically
- `Ctrl+w` followed by arrow key - Navigate between windows

## Customizing Vim

You can customize Vim by creating or editing the `.vimrc` file in your home directory:

```bash
vim ~/.vimrc
```

Common settings include:
```
syntax on           " Enable syntax highlighting
set number          " Show line numbers
set autoindent      " Auto-indent new lines
set tabstop=4       " Set tab width to 4 spaces
set expandtab       " Use spaces instead of tabs
```

## Vim Plugins

Vim can be extended with plugins. Popular plugin managers include:
- Vim-Plug
- Vundle
- Pathogen

## Resources for Learning More

- Vimtutor: Type `vimtutor` in your terminal
- `:help` command within Vim
- Online resources: [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)