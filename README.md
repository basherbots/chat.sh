## OpenRouter Bash Script

This script provides an interactive chatbot experience powered by the OpenRouter API. It allows you to have conversations with a large language model from OpenRouter.

### Features

* Interactive chat session with an OpenRouter language model
* Ability to set a system prompt for the assistant
* Option to clear or archive chat history
* Support for providing an initial message to start the conversation

### Requirements

* Bash shell
* OpenRouter API key (obtainable from [https://openrouter.ai/](https://openrouter.ai/))
* OpenRouter model name (e.g., openai/gpt-3.5-turbo)

### Installation

1. **Save the script:** Save the script content as `modelprompter.sh` (or any desired name) in a preferred location. 
2. **Make the script executable:** Open a terminal and navigate to the directory where you saved the script. Then, run the following command:

```
chmod +x modelprompter.sh
```

3. **Configure OpenRouter details (optional):**
  * You can configure your OpenRouter API key and model name directly in the script by editing the `load_or_prompt_config` function.
  * Alternatively, the script will prompt you for these details during the first run.

### Usage

1. **Run the script:** Open a terminal and navigate to the directory where the script is saved. Run the script with the following command:

```
./modelprompter.sh
```

2. **(Optional) Provide arguments:**
  * `--help`: Displays help information and exits.
  * `--clear`: Clears the chat history.
  * `--archive`: Archives the current chat history.
  * `--system-prompt FILE`: Sets the system prompt from a file. (Provide the file path after the argument)
  * `--system-prompt-string "PROMPT"`: Sets the system prompt as a string. (Replace "PROMPT" with your desired message)
  * You can also provide an initial message in quotes after the script name to start the conversation with that message.

3. **Interact with the chatbot:** Type your messages and press Enter to chat with the OpenRouter model.
4. **Exit:** Type `q` and press Enter to exit the chat session.

### Additional Notes

* The script stores chat history in the `~/.mp_history` file by default.
* The script uses a temporary file (`/tmp/mp_temp_script.sh`) to execute any code embedded in the response from the OpenRouter API.

This README provides a basic understanding of the script.  Feel free to customize the script further based on your needs.
