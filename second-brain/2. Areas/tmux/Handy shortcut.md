### 🚀 **Session Management**

| Command                       | Description                  |
| ----------------------------- | ---------------------------- |
| `tmux`                        | Start a new session          |
| `tmux new -s <session-name>`  | Start a new named session    |
| `tmux ls`                     | List all sessions            |
| `tmux a` or `tmux attach`     | Attach to last session       |
| `tmux a -t <session-name>`    | Attach to a specific session |
| `tmux kill-session -t <name>` | Kill a specific session      |
| `tmux kill-server`            | Kill all tmux sessions       |
### 🔀 **Window Management**

| Command                 | Description               |
| ----------------------- | ------------------------- |
| `Ctrl-b c`              | Create a new window       |
| `Ctrl-b n` / `Ctrl-b p` | Next / previous window    |
| `Ctrl-b &`              | Kill the current window   |
| `Ctrl-b ,`              | Rename current window     |
| `Ctrl-b w`              | Choose window from a list |
### 🧱 **Pane Management**

| Command              | Description                |
| -------------------- | -------------------------- |
| `Ctrl-b %`           | Split vertically           |
| `Ctrl-b "`           | Split horizontally         |
| `Ctrl-b o`           | Move to next pane          |
| `Ctrl-b ;`           | Toggle to last active pane |
| `Ctrl-b x`           | Close the current pane     |
| `Ctrl-b z`           | Zoom/unzoom pane           |
| `Ctrl-b <arrow key>` | Move between panes         |
| `Ctrl-b q`           | Show pane numbers briefly  |
### 🧠 **Useful Tips**

| Tip                             | Description                              |
| ------------------------------- | ---------------------------------------- |
| `Ctrl-b [`                      | Enter copy mode (scroll with arrow keys) |
| `Ctrl-b ]`                      | Paste copied text                        |
| `Ctrl-b d`                      | Detach from session                      |
| `tmux source-file ~/.tmux.conf` | Reload config file                       |
