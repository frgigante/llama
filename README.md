# LlamaFS

<img src="electron-react-app/assets/llama_fs.png" width="30%" />

## Inspiration

[Watch the explainer video](https://x.com/AlexReibman/status/1789895425828204553)

Open your `~/Downloads` directory. Or your Desktop. It's probably a mess...

> There are only two hard things in Computer Science: cache invalidation and **naming things**.

## What it does

LlamaFS is a self-organizing file manager. It automatically renames and organizes your files based on their content and well-known conventions (e.g., time). It supports many kinds of files, including images (through Moondream) and audio (through Whisper).

LlamaFS runs in two "modes" - as a batch job (batch mode), and an interactive daemon (watch mode).

In batch mode, you can send a directory to LlamaFS, and it will return a suggested file structure and organize your files.

In watch mode, LlamaFS starts a daemon that watches your directory. It intercepts all filesystem operations and uses your most recent edits to proactively learn how you rename file. For example, if you create a folder for your 2023 tax documents, and start moving 1-3 files in it, LlamaFS will automatically create and move the files for you! (watch mode defaults to sending files to groq if you have the environment variable "GROQ_API_KEY" set, otherwise through ollama)

Uh... Sending all my personal files to an API provider?! No thank you!

BREAKING CHANGE: Now by default, llama-fs uses "incognito mode" (if you have not configured an environment key for "GROQ_API_KEY") allowing you route every request through Ollama instead of Groq. Since they use the same Llama 3 model, the perform identically. To use a different model, set the environment variable "MODEL" to a string which litellm can use as a model like "ollama/llama3" or "groq/llama3-70b-8192".

## How we built it

We built LlamaFS on a Python backend, leveraging the Llama3 model through Groq for file content summarization and tree structuring. For local processing, we integrated Ollama running the same model to ensure privacy in incognito mode. The frontend is crafted with Electron, providing a sleek, user-friendly interface that allows users to interact with the suggested file structures before finalizing changes.

- **It's extremely fast!** (by LLM standards)! Most file operations are processed in <500ms in watch mode (benchmarked by [AgentOps](https://agentops.ai/?utm_source=llama-fs)). This is because of our smart caching that selectively rewrites sections of the index based on the minimum necessary filesystem diff. And of course, Groq's super fast inference API. 😉

- **It's immediately useful** - It's very low friction to use and addresses a problem almost everyone has. We started using it ourselves on this project (very Meta).

## What's next for LlamaFS

- Find and remove old/unused files
- We have some really cool ideas for - filesystem diffs are hard...

## Installation

### Prerequisites

Before installing, ensure you have the following requirements:
- Python 3.9 or higher
- pip (Python package installer)

### Installing

To install the project, follow these steps:
1. Clone the repository:
   ```bash
   git clone https://github.com/iyaja/llama-fs.git
   ```

2. Navigate to the project directory:
    ```bash
    cd llama-fs
    ```

3. Install requirements
   ```bash
   pip install -r requirements.txt
   ```

4. Install ollama and pull the model moondream if you want to recognize images
    ```bash
    ollama pull moondream
    ```
   We highly recommend pulling an additional model like llama3 for local ai inference on text files. You can control which ollama model is used by setting the "MODEL" environment variable to a litellm compatible model string.

## Usage

To serve the application locally using FastAPI, run the following command
   ```bash
   fastapi dev server.py
   ```

This will run the server by default on port 8000. The API can be queried using a `curl` command, and passing in the file path as the argument. For example, on the Downloads folder:
   ```bash
   curl -X POST http://127.0.0.1:8000/batch \
    -H "Content-Type: application/json" \
    -d '{"path": "/Users/<username>/Downloads/", "instruction": "string", "incognito": false}'
   ```
