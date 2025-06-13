# Termlog - Terminal Session Logger

A bash script that monitors and logs terminal input/output to markdown files, perfect for AI coding agents to review console history, errors, and test results.

## Installation

The script has been installed to `~/.local/bin/termlog`. To use it immediately in your current session:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

This PATH addition has been added to your `~/.bashrc` for future sessions.

## Usage Modes

### 1. Session Mode (NEW) - Log Everything

Start logging all terminal output:
```bash
# Start logging
termlog start
termlog start --thread 1

# Run your commands normally - everything is logged
npm run dev
python app.py
uvicorn main:app --reload

# Stop logging
termlog stop
termlog stop --thread 1

# Check status
termlog status
```

### 2. Command Mode - Log Single Commands

```bash
# Log a command's output
termlog npm run dev

# Pipe output through termlog
npm run dev 2>&1 | termlog
```

## Thread/Channel Support

Use the `--thread` or `-t` option to organize logs:

```bash
# Session mode with threads
termlog start --thread 1      # Terminal 1
termlog start --thread 2      # Terminal 2

# Multiple terminals, same project (logs to same file with prefixes)
termlog start --thread 1       # Logs as "1> output"
termlog start --thread 2       # Logs as "2> output"

# Multiple terminals, different projects (logs to different files)
termlog start --thread server1  # Logs to console-server.md
termlog start --thread client1  # Logs to console-client.md
```

## Log Files

- Default location: `~/.termlog/`
- Default file: `console.md`
- Named threads: `console-{name}.md`
- Session info: `~/.termlog/.sessions/`

## Options

- `start`: Start logging session
- `stop`: Stop logging session
- `status`: Check logging status
- `-t, --thread ID`: Specify thread/channel ID
- `-d, --dir PATH`: Custom log directory (default: ~/.termlog)
- `-h, --help`: Show help message

## Examples

### Long-Running Development Server

```bash
# Terminal 1: Start logging and run server
termlog start --thread 1
npm run dev
# ... server runs, all output is logged ...
# Press Ctrl+C to stop server
termlog stop --thread 1

# Terminal 2: Run tests while server is running
termlog start --thread 2
npm test
termlog stop --thread 2
```

### Multiple Projects

```bash
# Backend project
termlog start --thread backend1
uvicorn main:app --reload
# ... all output logged to console-backend.md ...

# Frontend project (different terminal)
termlog start --thread frontend1
npm run dev
# ... all output logged to console-frontend.md ...
```

### Review Logs

```bash
# View current logs
cat ~/.termlog/console.md

# Watch logs in real-time
tail -f ~/.termlog/console.md

# Check if logging is active
termlog status
termlog status --thread 1
```

## Log Format

Logs are formatted as markdown with:
- Session start/stop markers
- Timestamps for each output block
- Command executed (in command mode)
- STDOUT/STDERR differentiation
- Exit codes (in command mode)
- Thread prefixes for multi-terminal sessions

### Session Mode Example:
```markdown
## 🟢 Session Started
**Thread:** 1
**Time:** 2025-06-13 10:15:30

---

```
[2025-06-13 10:15:35] SESSION
1> npm run dev
1> 
1> > my-app@1.0.0 dev
1> > next dev
1> 
1> ready - started server on http://localhost:3000
```

```
[2025-06-13 10:15:40] SESSION
1> event - compiled client and server successfully
```

## 🔴 Session Ended
**Time:** 2025-06-13 10:20:45
```

### Command Mode Example:
```markdown
## Command: npm run dev

```
[2025-06-13 09:47:50] STDOUT
1> Starting development server...
```

```error
[2025-06-13 09:47:51] STDERR
1> Error: Port 3000 already in use
```

### Exit Code: 1
```

## Notes

- **Session Mode**: Started with `termlog start`, captures ALL terminal output
- **Command Mode**: Only captures output from specific commands
- Logs persist between sessions
- Each new command/session adds to existing log files
- STDERR is formatted with error syntax highlighting
- Thread IDs can be numeric (1, 2, 3) or named (server1, client1)
- Named threads automatically create separate log files
- Session mode captures everything including prompts, interactive commands, etc.