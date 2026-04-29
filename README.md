# Local Coding LLM

Configs, scripts and docs 

## Basic Setup on Windows

1. Download llama.cpp latest version from https://github.com/ggml-org/llama.cpp/releases/ (latest CUDA version and related CUDA DLLs). You might need to copy the CUDA DLLs into the same folder as llama.cpp before running it. Extract all files in a folder on your SSD.

2. Pick a model, for general coding stuff qwen3.5 is a good starter model. Check your memory requirements (RAM + GPU memory): https://unsloth.ai/docs/models/qwen3.5#usage-guide

3. Download the .GGUF file model, which is several GBs. There's several versions of each model. Pick one that is (extra large XL) and quantized to 4bits (Q4_K_XL). For example for qwen3.5-27b pick: https://huggingface.co/unsloth/Qwen3.5-27B-GGUF/blob/main/Qwen3.5-27B-UD-Q4_K_XL.gguf

Place it inside the same folder as llama-server.exe on the SSD.

4. Look up the startup parameters. For example, for https://unsloth.ai/docs/models/qwen3.5#qwen3.5-27b the parameters are:

export LLAMA_CACHE="unsloth/Qwen3.5-27B-GGUF"
./llama.cpp/llama-cli \
    -hf unsloth/Qwen3.5-27B-GGUF:UD-Q4_K_XL \
    --ctx-size 16384 \
    --temp 0.6 \
    --top-p 0.95 \
    --top-k 20 \
    --min-p 0.00 

Create a new batch (.bat) file in the llama.cpp directory with:

llama-server.exe -m .\Qwen3.5-27B-UD-Q4_K_XL.gguf -ngl 999 --host 0.0.0.0 --port 11434 --ctx-size 16384 --temp 0.6 --top-p 0.95 --top-k 20 --min-p 0.00

The --ctx-size is important. It tells the size of your maximum context. The larger the context size, the more GPU memory you need. You can do some tricks by lowering --ngl to something like 5, which will start swapping layers between RAM and GPU memory, but everything will be pretty slow. If you run out of memory, lower ctx-size. If it runs fine, increase it. The more, the smarter it gets.

5. Launch the batch file to start the model. You can open the local chat at http://0.0.0.0:11434 to test it for simple prompts.

6. To make it useful for work, get opencode https://opencode.ai/. Don't bother to use the user interface, which is still in development. Instead, use the command line interface. First download the program and configure it to use your local server. You'll need to create/edit your ~/.config/opencode/opencode.json file with:

{
  "$schema": "https://opencode.ai/config.json",
  "compaction": {
    "auto": true,
    "prune": true,
    "reserved": 10000
  },
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (Local)",
      "options": {
        "baseURL": "http://0.0.0.0:11434/v1"
      },
      "models": {
        "qwen3.5:27b": {
          "name": "qwen3.5:27b",
          "limit": {
            "context": 65536,
            "input": 47014,
            "output": 18432
          }
        }
      }
    }
  }
}

https://opencode.ai/docs/config/

Note the context limit keys, which help avoid out-of-context errors.

7. Launch opencode-cli from the command line on a directory where you have some files/code and start asking questions or assign it tasks. Careful that this software can do destructive actions autonomously, by default. It can remove files by mistake! Either use it on stuff that you have backed up, or configure it to ask for permission to do dangerous stuff: https://opencode.ai/docs/config/#permissions

> By default, opencode allows all operations without requiring explicit approval. You can change this using the permission option.

## Best Models (TODO)

