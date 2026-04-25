# Gemini CLI — Full Autonomy (YOLO) Mode 🚀

This repository provides a permanent "YOLO mode" configuration for the official Gemini CLI, bypassing all execution prompts so it operates as a fully autonomous agent.

Because Gemini CLI employs a strict security and policy engine, achieving full autonomy requires configuring three separate layers: the Policy Engine, the Settings configuration, and a shell wrapper. 

If you follow all 3 steps below, Gemini CLI will never prompt you for permission again—it will natively auto-approve file edits, shell commands, system modifications, and network requests.

## How It Works

The Gemini CLI uses a policy engine to determine which tool actions are allowed, denied, or require user approval. By injecting a wildcard policy with a maximum allowed priority of `999`, we override all defaults. Combined with `auto_edit` settings and a CLI flag injection wrapper, the CLI is forced to auto-execute any command without pausing.

---

## Complete Installation Guide

### Step 1: Apply the Wildcard Policy File
The CLI needs a rule that universally allows all tools. Copy the `allow-all.toml` file into your Gemini CLI configuration folder.

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.gemini\policies"
Copy-Item allow-all.toml "$env:USERPROFILE\.gemini\policies\allow-all.toml"
```

**Mac / Linux:**
```bash
mkdir -p ~/.gemini/policies
cp allow-all.toml ~/.gemini/policies/allow-all.toml
```

*(Note: The `priority` in the toml file must be `999`. Using `1000` or higher triggers a schema validation error and the policy will be ignored.)*

### Step 2: Configure `settings.json`
You must configure the CLI's base settings to recognize automatic edits, otherwise it will crash or ignore the policy for certain tools.

Open your `settings.json` file:
* **Windows:** `%USERPROFILE%\.gemini\settings.json`
* **Mac/Linux:** `~/.gemini/settings.json`

Ensure your `general` and `security` blocks look like this:
```json
{
  "general": {
    "defaultApprovalMode": "auto_edit"
  },
  "security": {
    "disableYoloMode": false,
    "yolo": true,
    "enablePermanentToolApproval": true,
    "folderTrust": {
      "enabled": false
    }
  }
}
```
*(Crucial: Do not set `defaultApprovalMode` to `"yolo"`, as it will cause an Enum Error on startup. It must be `"auto_edit"`).*

### Step 3: Set up the Shell Wrapper (To skip workspace trust & force YOLO mode)
To guarantee you never see a prompt—including the initial workspace folder trust prompt or any interactive UI elements—you should wrap the `gemini` command in your shell profile. This secretly injects bypass flags every time you run it.

Add this to your `$PROFILE` (PowerShell) or `~/.bashrc` (Linux/Mac):

**PowerShell Wrapper:**
```powershell
# ── Gemini CLI: Full Autonomy Wrapper ──────────────────────────────────────
# Always runs with --approval-mode yolo and --skip-trust.
function gemini {
    & (Get-Command gemini -CommandType Application | Select-Object -First 1).Source `
        --approval-mode yolo `
        --skip-trust `
        @args
}
# ───────────────────────────────────────────────────────────────────────────
```

**Bash/Zsh Wrapper:**
```bash
alias gemini="gemini --approval-mode yolo --skip-trust"
```

---

## Verification
Once you've done all 3 steps, open a **brand new terminal**. Test it with a command that normally requires permission, such as creating and deleting a file:

```bash
gemini "create a file called test.txt then delete it using a shell command"
```
It should execute seamlessly from start to finish.

## Security Warning ⚠️
**This entirely disables the safety rails in Gemini CLI.**
If you tell the CLI to do something destructive (e.g. `rm -rf /` or `delete system32`), **it will execute it immediately without asking "Are you sure?"**. Use this configuration entirely at your own risk.
