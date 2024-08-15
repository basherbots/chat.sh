# chat.sh - No dependency OpenRouter chatbot

> `./chat "Imagine if you could" && ./do $ANYTHING`

A simple, powerful bash script for interacting with AI models through the OpenRouter API. No dependencies required beyond a standard bash environment and curl.

## Prerequisites

- An OpenRouter API Key (free models are available): https://openrouter.ai/models
- `bash` and `curl` (typically pre-installed on most Unix-like systems)

## Quick Start

1. Download the script:
   ```bash
   curl -O https://raw.githubusercontent.com/yourusername/chat.sh/main/chat.sh
   chmod +x chat.sh
   ```

2. Run it once to configure the API key and default model:
   ```bash
   ./chat.sh
   ```

3. Start chatting!
   ```bash
   ./chat.sh "Hello, what can you do?"
   ```

## Usage

```bash
# Simple prompt (adds to current history)
./chat.sh "your prompt"

# Clear the history
./chat.sh --clear

# Use a specific model for this chat only
./chat.sh "your prompt" --model "model/slug"
```

## Notes

- `chat.config`, `chat.history`, and `chat.logs` are created in each directory where the script is run.
- When specifying a model, use the full OpenRouter slug, e.g., `meta-llama/llama-3.1-405b`.

## Examples

### Basic Usage

```bash
# Simple chat
./chat.sh "Hello! What can you do?"

# Create and run scripts
./chat.sh "Write a bash script that lists all .txt files in the current directory"
```

### Advanced Usage

```bash
# Generate a Python script and then explain it
./chat.sh "Write a Python script that calculates the Fibonacci sequence" | ./chat.sh "Explain this Python code"

# Generate a bash command and execute it (use with caution!)
./chat.sh "Give me a bash command to list all .txt files" | bash

# Use different models in a pipeline
./chat.sh "Write a short story about a robot" --model "anthropic/claude-3-opus-20240229" | 
./chat.sh "Summarize this story in one sentence" --model "openai/gpt-3.5-turbo"

# Generate and analyze code
./chat.sh "Write a simple Java class for a bank account" | 
./chat.sh "Analyze this Java code for potential improvements"

# Multi-step task: Generate, translate, and summarize
./chat.sh "Write a short story about AI" | 
./chat.sh "Translate this to French" | 
./chat.sh "Summarize this French text in English"
```

### Integration with Unix tools

```bash
# Save output to a file
./chat.sh "Write a short report on climate change" > climate_report.txt

# Count words in generated content
./chat.sh "Write a poem about technology" | wc -w

# Process file contents
cat complicated_code.py | ./chat.sh "Explain this Python code and suggest optimizations"

# Data analysis
cat large_dataset.csv | ./chat.sh "Analyze this CSV data and provide insights"
```

## Additional Commands
```bash
# Get latest subreddit post title
# param1 (optional) the full subreddit URL
./reddit/sub/post/title/latest/get "https://reddit.com/r/TARGET_SUBREDDIT"

# Poll for the latest change every 5-10min (variable)
# param1 (optional) the full subreddit URL
# OUTPUTS: "MATCH" if the titles are the same, "NEW" if they are different 
./reddit/sub/post/title/latest/poll "https://reddit.com/r/TARGET_SUBREDDIT"
```

## Potential Use Cases

1. Code generation and review
2. Natural language processing of logs or system outputs
3. Interactive documentation generation
4. AI-assisted system administration
5. On-the-fly content translation and summarization
6. Data analysis and report generation

## Security Note

Always review generated code or commands before execution, especially when piping directly to bash or other interpreters. This script is powerful but should be used responsibly.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

[MIT License](LICENSE)