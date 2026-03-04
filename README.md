# DayTrace

Automatically collects daily activities from Linear, GitHub, Slack, and Claude Code, generates a work-focused summary using Claude Agent SDK (Haiku), and saves it as an Obsidian markdown file.

## How It Works

```
Collectors (Linear, GitHub, Slack, Claude Code)
    ↓ Gather raw activity data (read-only APIs)
Claude Agent SDK (Haiku)
    ↓ Filter work-related activities + generate narrative summary
Obsidian Vault
    ↓ Save as YYYY-MM-DD.md (with YAML frontmatter, Dataview compatible)
```

Runs automatically via macOS launchd on Tue–Sat at 7:00 AM, recording the previous day's activities.

## Output Example

```markdown
---
date: '2026-03-03'
tracker: true
linear_completed: 6
slack_messages: 10
claude_sessions: 32
---
# 2026-03-03 (Tue)

## Linear

### Completed

- **AI-2300: HWP/HWPX File Processing Skill**
  - Implemented HWP/HWPX file parsing and text extraction
  - Completed full investigation of file processing mechanisms

## Slack

### onprem-tepsco

- **Gemma3 12B Model Verification**
  - Verified checksum in Runpod environment
  - Compared against version 0.10.2

## Claude Code

### mally (8 sessions)

- **Dev Workflow Pipeline Design**
  - Worktree creation → planning → Linear ticket → branch → implementation
```

## Installation

```bash
git clone https://github.com/jeongsangmin/daytrace.git
cd daytrace
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Configuration

```bash
cp .env.example .env
```

Edit `.env` with your tokens and paths:

| Variable | Description | Required |
|----------|-------------|----------|
| `LINEAR_API_KEY` | Linear → Profile icon → Account → API → Personal API key | Optional |
| `GITHUB_TOKEN` | GitHub → Settings → Developer settings → Personal access tokens | Optional |
| `GITHUB_USERNAME` | GitHub username | Optional |
| `SLACK_USER_TOKEN` | Slack App User OAuth Token (`search:read` scope) | Optional |
| `OBSIDIAN_VAULT_PATH` | Absolute path to your Obsidian Vault | **Required** |
| `OBSIDIAN_DAYTRACE_DIR` | Folder name inside Vault (default: `DayTrace`) | Optional |
| `CLAUDE_DIR` | Claude config directory (default: `~/.claude`) | Optional |

Collectors with missing tokens are automatically skipped.

### Slack App Setup

1. Go to https://api.slack.com/apps → Create New App
2. OAuth & Permissions → Add `search:read` to User Token Scopes
3. Install to Workspace → Copy User OAuth Token (`xoxp-...`)

### Claude Code Filtering

Add personal project names to `PERSONAL_KEYWORDS` in `collectors/claude.py` to exclude them from the summary.

### Customizing the Summary Prompt

Edit `prompts/daily_summary.md` to change the summary tone, format, or filtering rules.

## Usage

```bash
source .venv/bin/activate

# Record yesterday's activities
python main.py

# Record a specific date
python main.py --date 2026-03-03

# Test individual collectors
python -m collectors.linear 2026-03-04
python -m collectors.slack 2026-03-04
python -m collectors.claude 2026-03-04
```

## Automation (macOS)

```bash
chmod +x setup_launchd.sh
./setup_launchd.sh
```

### Auto-wake Mac

```bash
sudo pmset repeat wake MTWRF 07:00:00
```

Uses dark wake — the display stays off, only the system wakes.

### Full Flow

| Time | Event |
|------|-------|
| 07:00 | pmset dark-wakes the Mac |
| 07:00 | launchd runs `caffeinate -i python main.py` |
| ~07:01 | Collect → Summarize → Save complete |
| ~07:02 | caffeinate releases → system sleeps automatically |

## Backfill

`.last_run.json` tracks the last successful run date. Missed days are automatically backfilled on the next run.

## Project Structure

```
daytrace/
├── main.py                 # Entry point (backfill + collect + summarize + write)
├── config.py               # .env loading + settings
├── summarizer.py           # Claude Agent SDK summary generation
├── writer.py               # Obsidian markdown generation & file writing
├── collectors/
│   ├── linear.py           # Linear GraphQL API (read-only)
│   ├── github.py           # GitHub Events API (read-only)
│   ├── slack.py            # Slack search.messages (read-only, public channels only)
│   └── claude.py           # ~/.claude JSONL parsing (work projects only)
├── prompts/
│   └── daily_summary.md    # Summary prompt guide
├── setup_launchd.sh        # launchd plist generator & loader
├── .env.example            # Token template
├── .gitignore
└── requirements.txt
```

## License

MIT
