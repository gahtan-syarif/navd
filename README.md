# navd

An interactive fuzzy directory navigator for the terminal. Drill down into directories with a live tree preview. Press `CTRL-C` to land in the current directory.

## Demo
<img width="800" height="423" alt="navd" src="https://github.com/user-attachments/assets/a9548f06-ed0f-47fb-868e-6a51404c0b91" />

## Dependencies

| Tool  | Requirement            | Debian/Ubuntu          | Fedora              | Arch          |
|-------|------------------------|------------------------|---------------------|---------------|
| fzf   | Mandatory               | `apt install fzf`        | `dnf install fzf`     | `pacman -S fzf` |
| fd    | Optional                | `apt install fd-find`  | `dnf install fd-find` | `pacman -S fd`  |
| eza   | Optional                | `apt install eza`       | `dnf install eza`     | `pacman -S eza` |
| tree  | Optional                | `apt install tree`      | `dnf install tree`    | `pacman -S tree` |

## Installation

Copy and paste this function to your `~/.bashrc` or `~/.zshrc` file:

```bash
navd() {
  local fd_cmd preview_cmd

  if command -v fdfind &>/dev/null; then
    fd_cmd=fdfind
  elif command -v fd &>/dev/null; then
    fd_cmd=fd
  else
    fd_cmd=""
  fi

  if ! command -v fzf &>/dev/null; then
    echo "fzf not found, install fzf" && return 1
  fi

  if command -v eza &>/dev/null; then
    preview_cmd='eza --tree --color=always {}'
  elif command -v tree &>/dev/null; then
    preview_cmd='tree -C {}'
  else
    preview_cmd=""
  fi

  while true; do
    local dir
    if [ -n "$fd_cmd" ]; then
      dir=$($fd_cmd --exclude '.*/' --type d 2>/dev/null | fzf \
      --header="  CURRENT DIR: $PWD" ${preview_cmd:+--preview "$preview_cmd"} --expect=esc)
    else
      dir=$(find . -mindepth 1 -type d -not -path '*/.*' 2>/dev/null | cut -b3- | fzf \
      --header="  CURRENT DIR: $PWD" ${preview_cmd:+--preview "$preview_cmd"} --expect=esc)
    fi

    [ -z "$dir" ] && break

    local key line
    key=$(echo "$dir" | head -1)
    line=$(echo "$dir" | tail -1)

    if [ "$key" = "esc" ]; then
      local parent
      parent=$(dirname "$PWD")
      [ "$parent" != "$PWD" ] && cd "$parent"
    elif [ -n "$line" ]; then
      cd "$line"
    else
      break
    fi
  done
}
```

Then reload your shell:

```bash
source ~/.bashrc
```

## Usage

```bash
navd
```

- Navigate with arrow keys or type to fuzzy search
- `ENTER`: cd into the selected directory and continue browsing
- `ESC`: go up one directory level (`cd ..`)
- `CTRL-C`: quit and stay in the current directory

## Notes

- Hidden directories (`.git`, `.cache`, etc.) are excluded from results.
- `eza` or `tree` is used for the live tree preview, with `eza` taking priority. if neither is installed, the preview pane is disabled.
- `fd` (or `fdfind` on Debian/Ubuntu) is used by default for faster search, falling back to `find` if unavailable
