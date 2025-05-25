# Claude SDK

A flexible Python command-line interface for interacting with Claude CLI that provides structured logging and clean text output.

## Overview

Claude SDK wraps the Claude CLI to provide automatic session management and structured output. For direct access to Claude CLI, use `--simple` as the first argument.

### Key Features
- **Clean session tracking** without cluttering the workspace
- **Smart system prompt hierarchy** (CLI > config > default)
- **Support for both text and JSON outputs** with format-specific defaults
- **Simple mode** for direct Claude CLI access
- **Automatic session logging** for debugging while keeping your working directory clean

The `.claude-sdk/` directory keeps everything organized - conversation logs, outputs, and system prompts - while `prompt.txt` stays visible where you're working. You get easy output handling for scripts and full control over Claude's behavior when needed.

## How It Works

### Default Behavior
- Creates a timestamped directory in `.claude-sdk/`
- Claude streams its conversation as JSON to `conversation.json`
- Claude is instructed to write final output to `output.txt` or `output.json`
- Optionally streams output file content to stdout with `--return-output`
- Always uses `-p --output-format stream-json --verbose`

### System Prompt
System prompts are applied in this order (highest priority first):
1. **CLI flag**: `--system-prompt "Your custom prompt"`
2. **Config file**: `.claude-sdk/system-prompt.txt` (if manually created)
3. **Default**: Based on `--output-format`:
   - Text: Save to `output.txt`
   - JSON: Save valid JSON to `output.json`

The actual system prompt used is saved in each session directory for reference.

### Simple Mode (`--simple`)
- Pure passthrough to Claude CLI with auto-added `-p` flag
- No session management or file creation
- Full control over Claude's options
- Must be the first argument

## Requirements

- Python 3.6+
- Claude CLI installed and accessible in PATH

## Installation

```bash
# Make the script executable
chmod +x claude-sdk

# Add to PATH in ~/.bashrc
export PATH="$PATH:/path/to/claude-sdk"
```

## Usage

### Basic Usage

```bash
# Default - creates .claude-sdk/TIMESTAMP/
claude-sdk --prompt "Write a hello world program"

# Return output to stdout
claude-sdk --prompt "Explain Python decorators" --return-output

# Simple mode - direct passthrough to Claude (auto-adds -p)
claude-sdk --simple --help
claude-sdk --simple --output-format text "What is 2+2?"
claude-sdk --simple --continue
claude-sdk --simple -r  # Resume session
```

### Session Management

```bash
# Default: Reads from prompt.txt, creates .claude-sdk/20250125_143022/
claude-sdk

# Named session: Creates .claude-sdk/20250125_143022_feature_x/
claude-sdk --session-name feature_x

# Save output to file (still creates .claude-sdk session)
claude-sdk --prompt "Analyze this" -o results/analysis.txt
```

### Output Formats

```bash
# Text format (default) - outputs to output.txt
claude-sdk --prompt "Explain quantum computing"

# JSON format - outputs to output.json
claude-sdk --prompt "List programming languages" --output-format json
```

### Custom System Prompt

```bash
# Method 1: CLI flag (one-time use)
claude-sdk --prompt "Describe the sky" --system-prompt "Answer in exactly 5 words"

# Method 2: Config file (persistent)
echo "Always include pros and cons in your analysis" > .claude-sdk/system-prompt.txt
claude-sdk --prompt "Should I learn Rust?"

# Method 3: JSON-specific prompt via CLI
claude-sdk --prompt "Analyze this data" --output-format json \
  --system-prompt "Return a JSON object with 'summary' and 'details' fields"
```

### Output Modes

```bash
# Default: Session info only
claude-sdk --prompt "Analyze this"
# Output: Session: .claude-sdk/20250125_143022

# Return output: Streams output.txt to stdout
claude-sdk --prompt "Analyze this" --return-output
# Output: [Contents of output.txt]

# Pipe to file
claude-sdk --prompt "Generate docs" --return-output > docs.md
```

### Tool Configuration

```bash
# Use default tools
claude-sdk

# Select specific tools
claude-sdk -t Read Write Edit

# Enable all available tools
claude-sdk --all-tools
```

## Command-Line Arguments

| Argument | Short | Description | Default |
|----------|-------|-------------|---------|
| `--prompt-file` | `-p` | Path to prompt file | `prompt.txt` |
| `--prompt` | | Direct prompt string (overrides file) | |
| `--return-output` | | Stream output file to stdout | `False` |
| `--session-name` | | Add name to session directory | |
| `--output` | `-o` | Write output file to specified location | |
| `--output-format` | | Output format: text or json | `text` |
| `--system-prompt` | | Override system prompt for this run | |
| `--model` | `-m` | Claude model to use | |
| `--tools` | `-t` | Tools to allow | Bash, Read, Write, Edit, Task, Grep, Glob |
| `--all-tools` | | Allow all available tools | `False` |
| `--verbose` | `-v` | Enable verbose output | `False` |
| `--dry-run` | | Show command without executing | `False` |
| `--no-stream` | | Don't stream stderr to console | `False` |

## Examples

### Simple Mode
```bash
# Get help
claude-sdk --simple --help

# Quick tasks
claude-sdk --simple --output-format text "What is 15% of 240?"

# Continue conversation
claude-sdk --simple --continue

# Resume specific session
claude-sdk --simple -r session_id
```

### Session Tracking
```bash
# Analyze code with full tracking
claude-sdk -p analyze_code.txt --session-name code_review

# Get analysis output for further processing  
claude-sdk -p analyze_code.txt --return-output | grep "TODO"
```

### Batch Processing
```bash
# Process multiple files with session tracking
for file in *.py; do
    claude-sdk --prompt "Analyze $file for security issues" \
    --session-name "security_${file%.py}" --return-output \
    > "reports/${file%.py}_security.txt"
done
```

### Interactive Development
```bash
# Run analysis and see what files were created
claude-sdk -p refactor_request.txt --session-name refactor_v1

# Check the session
ls -la .claude-sdk/*/
```

### Programmatic Usage
```python
import subprocess

# Get output directly
result = subprocess.check_output([
    "claude-sdk", 
    "--prompt", "Calculate 15+27",
    "--return-output"
]).decode().strip()

# Save to file
subprocess.run([
    "claude-sdk",
    "--prompt", "Analyze this code", 
    "-o", "analysis.txt"
])
with open("analysis.txt") as f:
    result = f.read()
```

## Output Structure

### Default Behavior
```
.claude-sdk/
└── 20250125_143022/
    ├── conversation.json  # Full Claude conversation (stream-json)
    └── output.txt        # Claude's final output
```

With `--return-output`:
- Same files created
- `output.txt` content streamed to stdout
- Session info and paths printed to stderr

With `-o`:
- Same files created in `.claude-sdk/`
- `output.txt` content written to specified file
- Session info and paths printed to stderr

### Simple Mode
- No file creation
- Direct passthrough to Claude CLI
- Output controlled by Claude's flags

## Error Handling

- Returns Claude's exit code
- Handles Ctrl+C interruption gracefully
- Validates prompt file existence
- Warns if output.txt not created

## Tips

1. **Debugging**: Check `.claude-sdk/*/conversation.json` for full conversation
2. **Piping**: Use `--return-output` to integrate with Unix pipelines
3. **Organization**: Use `--session-name` for meaningful directory names
4. **Direct output**: Use `-o` to save output to a specific file

## License

MIT