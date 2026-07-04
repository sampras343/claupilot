# ClauDeck

[![CI](https://github.com/sampras343/claudeck/actions/workflows/ci.yml/badge.svg)](https://github.com/sampras343/claudeck/actions/workflows/ci.yml)
[![npm version](https://img.shields.io/npm/v/claudeck)](https://www.npmjs.com/package/claudeck)
[![Node.js](https://img.shields.io/node/v/claudeck)](https://nodejs.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Interactive web dashboard for monitoring and managing multiple [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI instances from a single screen.

## The Problem

When running multiple Claude Code terminals simultaneously, it's hard to track which one is doing what, which one needs your input, and switching between them is tedious.

## The Solution

ClauDeck gives you a real-time dashboard with scorecards for every active Claude instance. See status at a glance, respond to prompts without switching terminals, and auto-approve safe operations.

### Features

- **Real-time scorecards** — status, working directory, model, version, uptime, token usage, linked PRs
- **Input relay** — respond to permission prompts and questions directly from the dashboard
- **Dynamic prompt mirroring** — modal shows the exact same options Claude presents in the terminal (AskUserQuestion choices, tool permissions, free-text questions)
- **Smart auto-yes** — per-instance toggle that auto-approves safe operations (read-only commands, type checks) and defers risky ones (deletions, force pushes) to you
- **Grouping** — organize scorecards into named groups with drag-and-drop, collapsible sections
- **Toast notifications** — alerts when any instance needs attention

## Quick Start

```bash
npx claudeck
```

Or install globally:

```bash
npm install -g claudeck
claudeck
```

The dashboard opens at **http://localhost:3200**.

## Requirements

- **Node.js 20+**
- **Linux** (uses `pidfd_getfd` syscall for terminal input relay; ptrace_scope must be 0)
- **Claude Code CLI** installed and running in one or more terminals

## How It Works

ClauDeck reads Claude Code's internal state files to build a live view of all running instances:

| Source | Data |
|--------|------|
| `~/.claude/sessions/*.json` | PID, status (idle/busy/waiting), name, working directory |
| `~/.claude/jobs/*/state.json` | Background job state, pending prompts, token count, linked PRs |
| `~/.claude/daemon/roster.json` | Worker socket paths and auth tokens for input relay |
| `~/.claude/projects/*/*.jsonl` | Conversation transcripts for extracting actual prompt options |

### Input Relay

- **Background workers** — replies via the daemon's rendezvous Unix socket with authenticated messages
- **Interactive terminals** — injects keystrokes into the PTY master using `pidfd_getfd` to duplicate the terminal emulator's file descriptor

### Safety Assessment

The auto-yes feature classifies each permission prompt before acting:

| Level | Action | Examples |
|-------|--------|---------|
| **Safe** | Auto-approve | `ls`, `grep`, `git status`, `tsc --noEmit`, `Read` |
| **Moderate** | Auto-approve with logging | `Edit`, `npm install`, `git commit`, `mkdir` |
| **Risky** | Always ask user | `rm`, `git push`, `curl`, `docker`, `ssh` |
| **Dangerous** | Always ask user + warning | `rm -rf`, `git push --force`, `sudo`, credential access |

## Development

```bash
git clone https://github.com/sampras343/claudeck.git
cd claudeck
npm install
cd client && npm install && cd ..
npm run dev
```

This starts the backend (port 3200) and Vite dev server (port 5173 with proxy) concurrently.

## Configuration

| Environment Variable | Default | Description |
|---------------------|---------|-------------|
| `PORT` | `3200` | Server port |

## Architecture

```
Browser (:3200) <-- WebSocket --> Node.js Server
                                       |
                                  File Watchers (chokidar)
                                  ~/.claude/sessions/
                                  ~/.claude/jobs/
                                  ~/.claude/daemon/roster.json
                                       |
                                  Input Relay
                                  |-- Rendezvous socket (background workers)
                                  |-- PTY master injection (interactive terminals)
```

**Tech stack:** Node.js, Express, WebSocket, chokidar (backend) / React, Vite, Tailwind CSS v4 (frontend)

## License

MIT
