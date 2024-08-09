#!/bin/bash

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
CONFIG_FILE="$SCRIPT_DIR/chat.config"
HISTORY_FILE="$SCRIPT_DIR/chat.history"
LOG_FILE="$SCRIPT_DIR/chat.logs"

# System prompt as a variable for easy editing
SYSTEM_PROMPT="You are in a Linux Terminal. ALWAYS respond using <bot>response goes here</bot> markup. You MUST ALWAYS include executable bash scripts within <bash>bash script here</bash> tags. NEVER use backticks for code, ALWAYS use <bash> tags as these will be parsed out and executed. When asked to create files etc, assume the current directory. Always escape quotes since you're in a terminal. REMEMBER: YOU MUST ALWAYS OUTPUT THE ACTUAL CODE IN <bash> TAGS, NOT JUST DESCRIBE IT. The content may be | piped...DO NOT output any extra content like 'Done!' or 'Ok' if there is code, just output the tags with the code so it can run."

# Function to load or prompt for configuration
load_or_prompt_config() {
    if [ -f "$CONFIG_FILE" ]; then
        source "$CONFIG_FILE"
    fi

    if [ -z "$OPENROUTER_API_KEY" ]; then
        read -p "Enter your OpenRouter API key: " OPENROUTER_API_KEY >&2
        echo "OPENROUTER_API_KEY='$OPENROUTER_API_KEY'" >> "$CONFIG_FILE"
    fi

    if [ -z "$OPENROUTER_MODEL" ]; then
        read -p "Enter the OpenRouter model (e.g., openai/gpt-3.5-turbo): " OPENROUTER_MODEL >&2
        echo "OPENROUTER_MODEL='$OPENROUTER_MODEL'" >> "$CONFIG_FILE"
    fi
}

# Function to send message to OpenRouter API
send_message() {
    local messages="$1"
    local model="$2"
    local response=$(curl -s "https://openrouter.ai/api/v1/chat/completions" \
        -H "Content-Type: application/json" \
        -H "Authorization: Bearer $OPENROUTER_API_KEY" \
        -d '{
            "model": "'"$model"'",
            "messages": '"$messages"'
        }')
    
    echo "$response"
}

# Function to update history
update_history() {
    local role="$1"
    local content="$2"
    echo "<$role>$content</$role>" >> "$HISTORY_FILE"
}

# Function to update logs
update_logs() {
    local content="$1"
    echo "$content" >> "$LOG_FILE"
}

# Function to clear history and logs
clear_history_and_logs() {
    if [ -f "$HISTORY_FILE" ]; then
        rm "$HISTORY_FILE"
        echo "Chat history cleared." >&2
    else
        echo "No chat history found." >&2
    fi
    if [ -f "$LOG_FILE" ]; then
        rm "$LOG_FILE"
        echo "Chat logs cleared." >&2
    else
        echo "No chat logs found." >&2
    fi
}

# Function to execute bash scripts found in the response
execute_bash_scripts() {
    local response="$1"
    echo "$response"  # Output the entire response
    
    # Extract and execute all bash scripts
    while read -r line; do
        if [[ $line == *"<bash>"* ]]; then
            local script=$(echo "$line" | sed -n 's/.*<bash>\(.*\)<\/bash>.*/\1/p')
            echo "Executing: $script"
            eval "$script"
        elif [[ $line != *"<bot>"* && $line != *"</bot>"* ]]; then
            echo "$line"
        fi
    done <<< "$response"
}

# Main chat function
chat() {
    local user_input="$1"
    local model="$2"
    
    # Check if input is coming from a pipe
    if [ -p /dev/stdin ]; then
        local piped_input=$(cat)
        user_input="Context: $piped_input\n\nTask: $user_input"
    fi
    
    # Load history if exists
    if [ -f "$HISTORY_FILE" ]; then
        history=$(cat "$HISTORY_FILE")
    else
        history=""
    fi

    # Prepare the messages for the API
    local messages="["
    messages+="{\"role\":\"system\",\"content\":\"$SYSTEM_PROMPT\"},"
    while IFS= read -r line || [ -n "$line" ]; do
        if [[ $line =~ \<user\>(.*)\</user\> ]]; then
            messages+="{\"role\":\"user\",\"content\":\"${BASH_REMATCH[1]}\"},"
        elif [[ $line =~ \<bot\>(.*)\</bot\> ]]; then
            messages+="{\"role\":\"assistant\",\"content\":\"${BASH_REMATCH[1]}\"},"
        fi
    done <<< "$history"
    messages+="{\"role\":\"user\",\"content\":\"$user_input\"}]"

    # Send message and get response
    local full_response=$(send_message "$messages" "$model")
    
    # Log the raw server response
    update_logs "$full_response"

    # Extract the bot's response from the full server response
    local bot_response=$(echo "$full_response" | sed -n 's/.*<bot>\(.*\)<\/bot>.*/\1/p' | sed 's/\\n/\n/g')
    
    # If bot_response is empty, extract content without <bot> tags
    if [ -z "$bot_response" ]; then
        bot_response=$(echo "$full_response" | jq -r '.choices[0].message.content' | sed 's/\\n/\n/g')
    fi
    
    # Execute any bash scripts found in the response and output other content
    execute_bash_scripts "$bot_response"

    # Update history with user input and assistant's response
    update_history "user" "$user_input"
    update_history "bot" "$bot_response"
}

# Load or prompt for configuration
load_or_prompt_config

# Initialize variables
clear_flag=false
model="$OPENROUTER_MODEL"
user_input=""

# Parse command line arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        --clear)
            clear_flag=true
            shift
            ;;
        --model)
            model="$2"
            shift 2
            ;;
        *)
            user_input+=" $1"
            shift
            ;;
    esac
done

# Check for --clear flag
if $clear_flag; then
    clear_history_and_logs
    exit 0
fi

# Run chat function with input from command line or pipe
if [ -p /dev/stdin ]; then
    # Input is coming from a pipe
    user_input=$(cat)
else
    # Trim leading and trailing spaces from user input
    user_input="${user_input#"${user_input%%[![:space:]]*}"}"
    user_input="${user_input%"${user_input##*[![:space:]]}"}"
fi

if [ -z "$user_input" ]; then
    echo "Usage: $0 [--clear] [--model MODEL] \"Your message here\"" >&2
    echo "       echo \"Your message here\" | $0 [--clear] [--model MODEL]" >&2
    exit 1
fi

chat "$user_input" "$model"