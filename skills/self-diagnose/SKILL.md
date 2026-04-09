---
name: self-diagnose
description: Self-diagnose and auto-repair workspace issues — missing scripts, broken configs, stale data, and environment problems. Activate when a tool fails, a script is not found, or the user reports something not working.
---

# Self-Diagnose & Auto-Repair

When something goes wrong — a command fails, a file is missing, or the user says "X isn't working" — follow this skill to diagnose the root cause and fix it automatically.

## When to Activate

- A bash command exits with a non-zero code
- "file not found" / "command not found" / "module not found" errors
- User says something like "this is broken", "why doesn't X work", "fix this"
- A skill or tool produces unexpected results
- Cron job fails repeatedly

## Application Logs

The sandbox writes application logs to `/tmp/kicker-logs/sandbox.log` (human-readable format, one line per entry). Use this to investigate runtime errors, failed LLM calls, model fallbacks, and tool execution issues:

```bash
# Recent errors
grep -i 'ERROR' /tmp/kicker-logs/sandbox.log | tail -20

# Recent warnings
grep -i 'WARN' /tmp/kicker-logs/sandbox.log | tail -20

# Last 50 lines
tail -50 /tmp/kicker-logs/sandbox.log

# Search for a specific runId
grep '<runId>' /tmp/kicker-logs/sandbox.log
```

## Diagnosis Flow

### 1. Identify the Problem Category

Read the error message or user description and classify:

| Category                | Signals                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------ |
| **Missing script/file** | `No such file or directory`, `ENOENT`, user asks to run something that doesn't exist |
| **Missing dependency**  | `MODULE_NOT_FOUND`, `command not found`, `pip: not found`                            |
| **Permission error**    | `EACCES`, `Permission denied`                                                        |
| **Config/env issue**    | `undefined`, `null`, connection refused, auth failures                               |
| **Stale data**          | Outdated cache, wrong timezone, old session data                                     |
| **Disk/resource issue** | `ENOSPC`, `ENOMEM`, disk full                                                        |

### 2. Diagnose

Run targeted checks based on category:

**Missing script/file:**

```bash
# Check if the file exists anywhere in workspace
find ~/ -name "<filename>" -type f 2>/dev/null | head -5
# Check recent bash history for clues about what the script should do
cat ~/.bash_history 2>/dev/null | grep "<keyword>" | tail -10
```

**Missing dependency:**

```bash
# Check what's available
which <command> 2>/dev/null || echo "not installed"
pip3 list 2>/dev/null | grep <package>
npm list -g 2>/dev/null | grep <package>
```

**Permission error:**

```bash
ls -la <path>
whoami
id
```

**Config/env issue:**

```bash
# Check environment (only safe vars)
env | grep -i "<keyword>"
# Check connectivity
curl -s --max-time 5 <url> -o /dev/null -w "%{http_code}"
```

**Disk/resource:**

```bash
df -h /
du -sh ~/  2>/dev/null
free -h 2>/dev/null || vm_stat 2>/dev/null
```

### 3. Auto-Repair

#### Missing Script → Create It

If the user asks to run a script that doesn't exist, **don't just report the error**. Instead:

1. Understand what the script should do from context (user's message, filename, surrounding files)
2. Create the script in `code/scripts/`
3. Make it executable: `chmod +x code/scripts/<name>`
4. Run it

Example: User says "run the cleanup script" but `cleanup.sh` doesn't exist → infer from context what needs cleaning (temp files? old logs? cache?), create the script, run it.

#### Missing Dependency → Install It

```bash
# Python packages
pip3 install --user <package>

# Node packages (global tools)
npm install -g <package>

# System packages (if available)
apt-get install -y <package> 2>/dev/null
```

Always try installing before telling the user it's unavailable.

#### Permission Error → Fix Permissions

```bash
chmod +x <script>           # Make script executable
chmod 644 <config-file>     # Fix config permissions
```

#### Stale Data → Refresh

- Clear cache: `rm -rf data/cache/*`
- Refresh memory index: re-read and re-index workspace files
- Update timezone if it looks wrong

#### Broken Config → Repair

1. Read the config file
2. Identify the malformed section
3. Fix it with the edit tool
4. Validate the fix

### 4. Verify the Fix

After any repair, **always verify**:

1. Re-run the original command that failed
2. Confirm the output is correct
3. Tell the user what was wrong and what you did

## Auto-Generated Scripts

When creating scripts that didn't exist before, follow these conventions:

- Location: `code/scripts/`
- Always add a shebang: `#!/bin/bash` or `#!/usr/bin/env python3`
- Add a brief comment header explaining what the script does
- Make executable with `chmod +x`
- Use error handling (`set -e` for bash, try/except for Python)

## Boundaries

- **Don't** modify system files outside the workspace
- **Don't** install packages that could conflict with the sandbox runtime
- **Don't** delete user data without confirmation
- **Don't** expose environment variables or secrets in output
- If you can't fix the problem, clearly explain the root cause and what the user can do
