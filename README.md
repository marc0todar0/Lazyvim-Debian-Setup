# LazyVim Dev setup on Debian (12+) \- Python & Ruby

## NEOVIM

```shell
curl -LO https://github.com/neovim/neovim/releases/latest/download/nvim-linux-x86_64.tar.gz
sudo rm -rf /opt/nvim-linux-x86_64
sudo tar -C /opt -xzf nvim-linux-x86_64.tar.gz
```

## Apt basics
``` 
sudo apt install -y ripgrep fd-find python3-pip build-essential tmux lazygit fzf zoxide
```
## Config files

`~/.bashrc`
 - (Add this at the end of the file)
```
alias ls='ls --color=auto'
export PATH="$PATH:/opt/nvim-linux-x86_64/bin"
export EDITOR='vim'
export VISUAL='vim'
alias n='nvim'
export DISABLE_SPRING=true
export PATH="$HOME/utils:$PATH"
export DOCKER_HOST=unix:///var/run/docker.sock
export AWS_PROFILE=ai
eval "$(zoxide init bash)"
source /usr/share/doc/fzf/examples/key-bindings.bash
```
`~/.bash_profile`
```
# Source .bashrc so login shells use the same settings
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```
`.tmux.conf`

```
setw -g mode-keys vi
set -g base-index 1
set -g default-terminal "tmux-256color"
```
My local alacritty config `~/.config/alacritty/alacritty.toml`
```
[env]
TERM = "xterm-256color"

[font]
size = 9.0

[font.normal]
family = "JetBrainsMono Nerd Font"
style = "Regular"

[font.bold]
family = "JetBrainsMono Nerd Font"
style = "Bold"

[font.italic]
family = "JetBrainsMono Nerd Font"
style = "Italic"

[window]
opacity = 0.90
```

## Clean Neovim State

If this is a new machine, still do this to avoid issues:

`rm -rf ~/.config/nvim`  
`rm -rf ~/.local/share/nvim`  
`rm -rf ~/.cache/nvim`

## Install a Nerd Font (Optional) \[Example: JetBrainsMono\]

`mkdir -p ~/.local/share/fonts`  
`cd ~/.local/share/fonts`  
`curl -LO https://github.com/ryanoasis/nerd-fonts/releases/latest/download/JetBrainsMono.zip`  
`unzip JetBrainsMono.zip`  
`fc-cache -fv`

## Add tree-sitter-cli 

`sudo npm install -g tree-sitter-cli`

## LazyVim

`git clone https://github.com/LazyVim/starter ~/.config/nvim`  
`rm -rf ~/.config/nvim/.git`  
`nvim`  
`→ :checkhealth`

### Mostra file nascosti/ git-ignored

- Shift \+ i \=\> mostra nell’explorer i file ignorati da git  
- Shift \+ h \=\> mostra nell’explorer i file nascosti

## Formatters Setup (Optional) 

pip3 install ruff  
vim \~/.config/nvim/lua/plugins/conform.lua
```
return {
  "stevearc/conform.nvim",
  opts = {
    formatters_by_ft = {
      javascript = { "prettier" },
      javascriptreact = { "prettier" },
      typescript = { "prettier" },
      typescriptreact = { "prettier" },
      json = { "prettier" },
      html = { "prettier" },
      css = { "prettier" },
      python = { "ruff_format" },
      ruby = { "rubocop" },
    },
  },
}
```
spc \+ c \+ f \=\> format python

## Absolute Line-numbers (Optional)
vim \~/.config/nvim/lua/config/options.lua
```
-- Options are automatically loaded before lazy.nvim startup
-- Default options that are always set: https://github.com/LazyVim/LazyVim/blob/main/lua/lazyvim/config/options.lua
-- Add any additional options here
-- -- Disable relative line-numbers
vim.opt.relativenumber = false
-- Enable absolute line-numbers
vim.opt.number = true
```

## LSP (Optional)
vim \~/.config/nvim/lua/plugins/lsp.lua
```
return {
  {
    "neovim/nvim-lspconfig",
    opts = {
      servers = {
        -- Enable ruff server for diagnostics (Errors and warnings)
        ruff = {},
        -- Pyright for type checking 
        pyright = {},
      },
    },
  },
}
```
- gd -> go to definition
- K -> open documentation modal
## LazyVim Replace using ripgrep (Optional)

vim \~/utils/replace

```
#!/bin/bash
#rg -l "$1" | xargs sed -i "s/$1/$2/g"
# . -> project (cwd - current working dir) [DEFAULT]
# src/ -> folder
# file.txt -> single file
set -euo pipefail
if [ "$#" -lt 2 ]; then
	echo "Usage: replace <SEARCH> <REPLACE> [PATH]"
	exit 1
fi
SEARCH="$1"
REPLACE="$2"
SCOPE="${3:-.}"
if [ -f "$SCOPE" ]; then
	#perl -pi -e "s/\Q$SEARCH\E/\Q$REPLACE\E/g" "$SCOPE"
	perl -pi -e "s/\Q$SEARCH\E/$REPLACE/g" "$SCOPE"
	exit 0
fi
rg -l --null -- "$SEARCH" "$SCOPE" |
while IFS= read -r -d '' file; do
	perl -pi -e "s/\Q$SEARCH\E/$REPLACE/g" "$file"
	#perl -pi -e "s/\Q$SEARCH\E/\Q$REPLACE\E/g" "$file"
done

```

chmod \+x \~/utils/replace  
vim \~/.config/nvim/lua/config/keymaps.lua

```
-- Keymaps are automatically loaded on the VeryLazy event
-- Default keymaps that are always set: https://github.com/LazyVim/LazyVim/blob/main/lua/lazyvim/config/keymaps.lua
-- Add any additional keymaps here
vim.keymap.set("n", "<leader>rr", function()
  local search = vim.fn.input("Search: ")
  local replace = vim.fn.input("Replace: ")

  local scope = vim.fn.input("Scope (f/d/p): ")

  local path = "."

  if scope == "f" then
    path = vim.fn.expand("%:p")
  elseif scope == "d" then
    path = vim.fn.expand("%:p:h")
  elseif scope == "p" then
    path = vim.fn.systemlist("git rev-parse --show-toplevel")[1] or "."
  end

  vim.cmd(
    "!"
      .. "~/utils/replace "
      .. vim.fn.shellescape(search)
      .. " "
      .. vim.fn.shellescape(replace)
      .. " "
      .. vim.fn.shellescape(path)
  )
end, { desc = "Replace (rg+perl)" })

```

\-\> then on lazyvim SPC+rr \=\> input SEARCH REPLACE SCOPE   
f → current file  
d → current folder  
p → project root

