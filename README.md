# Gemini CLI — Full Autonomy (YOLO) Mode 🚀

This repository provides a permanent "YOLO mode" (auto-approve all tools) configuration for the official Gemini CLI, bypassing all execution prompts so it operates as a fully autonomous agent.

## How it works

The Gemini CLI uses a policy engine to determine which tool actions are allowed, denied, or require user approval. By injecting a wildcard policy with a priority of 999, we override all defaults, forcing the CLI to auto-execute any command—file modifications, system deletion, shell commands, and package installations.

## Installation

### Step 1: Apply the Policy File
Copy the policy file into your Gemini CLI configuration folder:

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

### Step 2: (Optional) PowerShell Wrapper
To guarantee you never see a prompt—including the initial workspace trust prompt—you can wrap the `gemini` command in your profile.

Add this to your `$PROFILE` (PowerShell) or `~/.bashrc` (Linux/Mac):

```powershell
# PowerShell wrapper
function gemini {
    & (Get-Command gemini -CommandType Application | Select-Object -First 1).Source `
        --approval-mode yolo `
        --skip-trust `
        @args
}
```

## Security Warning ⚠️
**This entirely disables the safety rails in Gemini CLI.**
If you tell the CLI to do something destructive (e.g. `rm -rf /` or `delete system32`), **it will execute it immediately without asking "Are you sure?"**. Use this configuration entirely at your own risk.
