---
name: codex
description: Executes OpenAI Codex CLI for code analysis, refactoring, and automated editing. Activates when users mention codex commands, code review requests, or automated code transformations requiring advanced reasoning models.
---

# Codex Execution Skill

## Prerequisites
- Codex CLI installed and configured (`~/.codex/config.toml`)
- Verify availability: `codex --version` on first use per session

## Workflow Checklist

**For every Codex task, follow this sequence**:

1. ☐ **Ask user for execution parameters** via `AskUserQuestion` (single prompt):
   - Model: `gpt-5`, `gpt-5-codex`, or default
   - Reasoning effort: `minimal`, `low`, `medium`, `high`

2. ☐ **Determine sandbox mode** based on task:
   - `read-only`: Code review, analysis, documentation
   - `workspace-write`: Code modifications, file creation
   - `danger-full-access`: System operations, network access

3. ☐ **Build command** with required flags:
   ```bash
   codex exec [OPTIONS] "PROMPT"
   ```
   Essential flags:
   - `-m <MODEL>` (if overriding default)
   - `-c model_reasoning_effort="<LEVEL>"`
   - `-s <SANDBOX_MODE>`
   - `--skip-git-repo-check` (if outside git repo)
   - `-C <DIRECTORY>` (if changing workspace)
   - `--full-auto` (for non-interactive execution)

4. ☐ **Execute with stderr suppression**:
   - Append `2>/dev/null` to hide thinking tokens
   - Remove only if user requests verbose output or debugging

5. ☐ **Validate execution**:
   - Check exit code (0 = success)
   - Summarize output for user
   - Report errors with actionable solutions

6. ☐ **Inform about resume capability**:
   - "Resume this session anytime: `codex resume`"

## Command Patterns

### Read-Only Analysis
```bash
codex exec -m gpt-5 -c model_reasoning_effort="medium" -s read-only \
  --skip-git-repo-check --full-auto "review @file.py for security issues" 2>/dev/null
```

### Stdin Input (bypasses sandbox file restrictions)
```bash
cat file.py | codex exec -m gpt-5 -c model_reasoning_effort="low" \
  --skip-git-repo-check --full-auto - 2>/dev/null
```

### Code Modification
```bash
codex exec -m gpt-5 -c model_reasoning_effort="high" -s workspace-write \
  --skip-git-repo-check --full-auto "refactor @module.py to async/await" 2>/dev/null
```

### Resume Session
```bash
echo "fix the remaining issues" | codex exec --skip-git-repo-check resume --last 2>/dev/null
```

### Cross-Directory Execution
```bash
codex exec -C /path/to/project -m gpt-5 -c model_reasoning_effort="medium" \
  -s read-only --skip-git-repo-check --full-auto "analyze architecture" 2>/dev/null
```

### Using Profiles
```bash
codex exec --profile production -c model_reasoning_effort="high" \
  --full-auto "optimize performance in @app.py" 2>/dev/null
```

## CLI Reference

### Core Flags

| Flag | Values | When to Use |
|------|--------|-------------|
| `-m, --model` | `gpt-5`, `gpt-5-codex` | Override default model |
| `-c, --config` | `key=value` | Runtime config override (repeatable) |
| `-s, --sandbox` | `read-only`, `workspace-write`, `danger-full-access` | Set execution permissions |
| `-C, --cd` | `path` | Change workspace directory |
| `--skip-git-repo-check` | flag | Allow execution outside git repos |
| `--full-auto` | flag | Non-interactive mode (workspace-write + approvals on failure) |
| `-p, --profile` | `string` | Load configuration profile from config.toml |
| `--json` | flag | JSON event output (CI/CD pipelines) |
| `-o, --output-last-message` | `path` | Write final message to file |
| `-i, --image` | `path[,path...]` | Attach images (repeatable or comma-separated) |
| `--oss` | flag | Use local open-source model (requires Ollama) |

### Configuration Options

**Model Reasoning Effort** (`-c model_reasoning_effort="<LEVEL>"`):
- `minimal`: Quick tasks, simple queries
- `low`: Standard operations, routine refactoring
- `medium`: Complex analysis, architectural decisions (default)
- `high`: Critical code, security audits, complex algorithms

**Model Verbosity** (`-c model_verbosity="<LEVEL>"`):
- `low`: Minimal output
- `medium`: Balanced detail (default)
- `high`: Verbose explanations

**Approval Prompts** (`-c approvals="<WHEN>"`):
- `on-request`: Before any tool use
- `on-failure`: Only on errors (default for `--full-auto`)
- `untrusted`: Minimal prompts
- `never`: No interruptions (use with caution)

## Configuration Management

### Config File Location
`~/.codex/config.toml`

### Runtime Overrides
```bash
# Override single setting
codex exec -c model="gpt-5" "task"

# Override multiple settings
codex exec -c model="gpt-5" -c model_reasoning_effort="high" "task"
```

### Using Profiles
Define in `config.toml`:
```toml
[profiles.research]
model = "gpt-5"
model_reasoning_effort = "high"
sandbox = "read-only"

[profiles.development]
model = "gpt-5-codex"
sandbox = "workspace-write"
```

Use with:
```bash
codex exec --profile research "analyze codebase"
```

## Resume Behavior

**Automatic inheritance**:
- Model selection
- Reasoning effort
- Sandbox mode
- Configuration overrides

**Resume syntax**:
```bash
# Resume last session
codex exec resume --last

# Resume with new prompt
codex exec resume --last "continue with next steps"

# Resume via stdin
echo "new instructions" | codex exec resume --last 2>/dev/null

# Resume specific session
codex exec resume <SESSION_ID> "follow-up task"
```

**Flag injection** (between `exec` and `resume`):
```bash
# Change reasoning effort for resumed session
codex exec -c model_reasoning_effort="high" resume --last
```

## Error Handling

### Validation Loop
1. Execute command
2. Check exit code (non-zero = failure)
3. Report error with context
4. Ask user for direction via `AskUserQuestion`
5. Retry with adjustments or escalate

### Permission Requests
Before using high-impact flags, request user approval via `AskUserQuestion`:
- `--full-auto`: Automated execution
- `-s danger-full-access`: System-wide access
- `--dangerously-bypass-approvals-and-sandbox`: All safeguards disabled (isolated environments only)

### Partial Success Handling
When output contains warnings:
1. Summarize successful operations
2. Detail failures with context
3. Use `AskUserQuestion` to determine next steps
4. Propose specific adjustments

## Troubleshooting

### File Access Blocked

**Symptom**: "shell is blocked by the sandbox" or permission errors

**Root cause**: Sandbox `read-only` mode restricts file system

**Solutions** (priority order):

1. **Stdin piping** (recommended):
   ```bash
   cat target.py | codex exec -m gpt-5 -c model_reasoning_effort="medium" \
     --skip-git-repo-check --full-auto - 2>/dev/null
   ```

2. **Explicit permissions**:
   ```bash
   codex exec -m gpt-5 -s read-only \
     -c 'sandbox_permissions=["disk-full-read-access"]' \
     --skip-git-repo-check --full-auto "@file.py" 2>/dev/null
   ```

3. **Upgrade sandbox**:
   ```bash
   codex exec -m gpt-5 -s workspace-write \
     --skip-git-repo-check --full-auto "review @file.py" 2>/dev/null
   ```

### Invalid Flag Errors

**Symptom**: "unexpected argument '--add-dir' found"

**Cause**: Flag does not exist in Codex CLI

**Solution**: Use `-C <DIR>` to change directory:
```bash
codex exec -C /target/dir -m gpt-5 --skip-git-repo-check \
  --full-auto "task" 2>/dev/null
```

### Exit Code Failures

**Symptom**: Non-zero exit without clear message

**Diagnostic steps**:
1. Remove `2>/dev/null` to see full stderr
2. Verify installation: `codex --version`
3. Check configuration: `cat ~/.codex/config.toml`
4. Test minimal command: `codex exec -m gpt-5 "hello world"`
5. Verify model access: `codex exec --model gpt-5 "test"`

### Model Unavailable

**Symptom**: "model not found" or authentication errors

**Solutions**:
1. Check configured model: `grep model ~/.codex/config.toml`
2. Verify API access: Ensure valid credentials
3. Try alternative model: `-m gpt-5-codex`
4. Use OSS fallback: `--oss` (requires Ollama)

### Session Resume Fails

**Symptom**: Cannot resume previous session

**Diagnostic steps**:
1. List recent sessions: `codex history`
2. Verify session ID format
3. Try `--last` flag instead of specific ID
4. Check if session expired or was cleaned up

## Best Practices

### Reasoning Effort Selection
- **minimal**: Syntax fixes, simple renaming
- **low**: Standard refactoring, basic analysis
- **medium**: Complex refactoring, architecture review
- **high**: Security audits, algorithm optimization, critical bugs

### Sandbox Mode Selection
- **read-only**: Default for any analysis or review
- **workspace-write**: File modifications only
- **danger-full-access**: Network operations, system commands (rare)

### Stderr Suppression
- **Always use `2>/dev/null`** unless:
  - User explicitly requests thinking tokens
  - Debugging failed commands
  - Troubleshooting configuration issues

### Profile Usage
Create profiles for common workflows:
- `review`: High reasoning, read-only
- `refactor`: Medium reasoning, workspace-write
- `quick`: Low reasoning, read-only
- `security`: High reasoning, workspace-write

### Stdin vs File Reference
- **Stdin**: Single file analysis, avoids permissions
- **File reference**: Multi-file context, codebase-wide changes

## Safety Guidelines

**Never combine**:
- `--full-auto` + `--dangerously-bypass-approvals-and-sandbox` (unless isolated VM)

**Always verify before**:
- Using `danger-full-access` sandbox
- Disabling approval prompts
- Running with `--full-auto` on production code

**Ask user approval for**:
- First-time `workspace-write` usage
- System-wide access requests
- Destructive operations (deletions, migrations)

## Advanced Usage

### CI/CD Integration
```bash
codex exec --json -o result.txt -m gpt-5 \
  -c model_reasoning_effort="medium" \
  --skip-git-repo-check --full-auto \
  "run security audit on changed files" 2>/dev/null
```

### Batch Processing
```bash
for file in *.py; do
  cat "$file" | codex exec -m gpt-5 -c model_reasoning_effort="low" \
    --skip-git-repo-check --full-auto "lint and format" - 2>/dev/null
done
```

### Multi-Step Workflows
```bash
# Step 1: Analysis
codex exec -m gpt-5 -c model_reasoning_effort="high" -s read-only \
  --full-auto "analyze @codebase for architectural issues" 2>/dev/null

# Step 2: Resume with changes
echo "implement suggested refactoring" | \
  codex exec -s workspace-write resume --last 2>/dev/null
```

## When to Escalate

If errors persist after troubleshooting:

1. **Check documentation**:
   ```bash
   WebFetch https://developers.openai.com/codex/cli/reference
   ```
   ```bash
   WebFetch https://developers.openai.com/codex/local-config#cli
   ```

2. **Report to user**:
   - Error message verbatim
   - Attempted solutions
   - Configuration details
   - Exit codes and stderr output

3. **Request guidance**:
   - Alternative approaches
   - Configuration adjustments
   - Manual intervention points

## Model Selection Guide

| Task Type | Recommended Model | Reasoning Effort |
|-----------|------------------|------------------|
| Quick syntax fixes | `gpt-5` | minimal |
| Code review | `gpt-5` | medium |
| Refactoring | `gpt-5-codex` | medium |
| Architecture analysis | `gpt-5` | high |
| Security audit | `gpt-5` | high |
| Algorithm optimization | `gpt-5-codex` | high |
| Documentation generation | `gpt-5` | low |

## Common Workflows

### Code Review Workflow
1. Ask user: model + reasoning effort
2. Run read-only analysis
3. Present findings
4. If changes needed: resume with workspace-write
5. Validate changes
6. Inform about resume capability

### Refactoring Workflow
1. Ask user: model + reasoning effort
2. Analyze current code (read-only)
3. Propose changes
4. Get user approval
5. Apply changes (workspace-write)
6. Run validation/tests
7. Report results

### Security Audit Workflow
1. Use high reasoning effort
2. Run comprehensive analysis (read-only)
3. Document findings
4. Propose fixes
5. Apply fixes if approved (workspace-write)
6. Re-audit to verify
7. Generate report
