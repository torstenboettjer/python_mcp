# Prototyping a MCP Server on NixOS (using Gradio)

This repo provides tools and explanations to build a MCP server with Gradio, a NodeJS/Python framework, on NixOS. Python virtual environments rely on the [automatic shell activation](https://devenv.sh/automatic-shell-activation/) with devenv.sh and direnv.

## Prepare a Repository

Create a repository on github, clone the it into a local folder. Within the local folder prepare the devevlopment environment

```sh
devenv init
```

Update the greet variable and add the following packages in  devenv.nix

```nix
# https://devenv.sh/basics/
env.GREET = "MCP Prototype";

# https://devenv.sh/packages/
packages = with pkgs; [
  nodejs
  python313Packages.gradio
  python313Packages.mcp
];
```

Add the python environment parameter to `devenv.nix`

```nix
# https://devenv.sh/languages/
languages.python = {
enable = true;
package = pkgs.python313;
venv.enable = true;
venv.requirements = ''
    requests
    pip
'';
uv.enable = true;
};
```

After saving the devenv configuration python and uv should be available.

```sh
python --version && uv --version
pip show gradio
```

At time of testing the gradio and mcp packages for NixOS where out of date, if necessary update gradio and mcp to the latest version

```sh
pip install --upgrade gradio && pip install --upgrade mcp
```

Add the dotfiles to `.gitignore`

```sh
echo -e ".envrc" >> .gitignore
```

## Initial test with FastMCP

FastMCP is a Python library that significantly simplifies the creation and interaction with Model Context Protocol (MCP) servers and clients. `demo_fastmcp.py` is the demo example from the huggingface tutorial. Starting the example with the MCP inspector

```sh
mcp dev demo_fastmcp.py
```

## Gradio

Gradio is a UI layer for Python functions, it provides an interface to third-party API (e.g. OpenAI), LLM frameworks, custom-trained models or Hugging Face models. Hugging Face is a platform to publish and consume pre-trained LLMs. Gradio offers several ways to integrate with LLMs:

### Hugging Face Inference Endpoints

Using `gr.load()` is the **easiest and often recommended way** if the model is available on the Hugging Face Hub and supports inference endpoints, not need to download the model locally.

```python
import gradio as gr

# Load a translation model from Hugging Face Inference Endpoints
demo = gr.load("Helsinki-NLP/opus-mt-en-es", src="models")
demo.launch()
```

  * `"Helsinki-NLP/opus-mt-en-es"` is the model ID on Hugging Face.
  * `src="models"` tells Gradio to look for this model on the Hugging Face Model Hub and use its inference endpoint. Gradio automatically handles the API calls.

### Local Hugging Face Models

If the model isn't supported by inference endpoints or local inference is required, Using `transformers.pipeline`, Gradio can load a model using Hugging Face's `transformers` library and then wrap it in an interface.

```python
import gradio as gr
from transformers import pipeline

# Load a local text generation pipeline
# Make sure you have the model downloaded or it will download on first run
pipe = pipeline("text-generation", model="distilgpt2")

def generate_text(prompt):
    return pipe(prompt, max_new_tokens=50)[0]["generated_text"]

demo = gr.Interface(
    fn=generate_text,
    inputs="text",
    outputs="text",
    title="DistilGPT2 Text Generator"
)
demo.launch()
```

  * Create a `pipeline` object from `transformers`.
  * Define a Python function (`generate_text` in this case) that calls your `pipe` object.
  * Pass this function to `gr.Interface`.

### Transformer Pipeline

Using `gr.Interface.from_pipeline`, Gradio provides a convenient shorthand for `transformers.pipeline` objects.

```python
import gradio as gr
from transformers import pipeline

# Load a local text generation pipeline
pipe = pipeline("text-generation", model="distilgpt2")

# Directly create an Interface from the pipeline
demo = gr.Interface.from_pipeline(pipe)
demo.launch()
```

### External APIs (e.g., OpenAI, Google Gemini, Anthropic)

Using their respective Python SDKs, commercial or hosted LLMs are called via API.

**OpenAI API**

```python
import gradio as gr
from openai import OpenAI
import os

# Set your OpenAI API key (replace with your actual key or load from env)
# It's best practice to use environment variables for API keys
os.environ["OPENAI_API_KEY"] = "YOUR_OPENAI_API_KEY"
client = OpenAI() # Initialize OpenAI client

def chat_with_gpt(message, history):
    # Convert Gradio chat history format to OpenAI format
    messages = []
    for human, assistant in history:
        messages.append({"role": "user", "content": human})
        messages.append({"role": "assistant", "content": assistant})
    messages.append({"role": "user", "content": message})

    response = client.chat.completions.create(
        model="gpt-3.5-turbo", # Or "gpt-4", "gpt-4o", etc.
        messages=messages,
        stream=True # For streaming responses in Gradio
    )

    partial_message = ""
    for chunk in response:
        if chunk.choices[0].delta.content is not None:
            partial_message += chunk.choices[0].delta.content
            yield partial_message # Yield partial responses for streaming UI

demo = gr.ChatInterface(
    fn=chat_with_gpt,
    title="ChatGPT Chatbot"
)
demo.launch()
```

What this code does:
  * Import the `OpenAI` library.
  * Initialize the client with your API key (stored securely).
  * The `chat_with_gpt` function constructs the API call, sends the request, and processes the response.
  * For chatbots, `gr.ChatInterface` is very convenient as it handles the chat history and UI elements automatically.
  * The `yield` keyword is used for streaming responses, which provides a better user experience for LLMs.

**Google Gemini API**

The structure would be similar, just using the `google.generativeai` library instead of `openai`.

```python
import gradio as gr
import google.generativeai as genai
import os

# Set your Google API key (replace with your actual key or load from env)
os.environ["GEMINI_API_KEY"] = "YOUR_GEMINI_API_KEY"
genai.configure(api_key=os.environ["GEMINI_API_KEY"])

model = genai.GenerativeModel('gemini-pro')

def chat_with_gemini(message, history):
    # Convert Gradio chat history format to Gemini format
    # Gemini's history format is slightly different
    # You might need to adjust this based on the specific model and its requirements
    convo = model.start_chat(history=[]) # Start a new conversation for each call for simplicity, or manage history

    # You would typically convert the full history and pass it to the model
    # For a simple example, let's just send the current message
    response = convo.send_message(message)
    return response.text

demo = gr.ChatInterface(
    fn=chat_with_gemini,
    title="Gemini Chatbot"
)
demo.launch()
```

### Custom Python Functions

Locally Trained LLMs or Custom Python Functions perform LLM-like tasks, but through a text-processing function.

```python
import gradio as gr

def simple_echo_llm(text_input):
    """A very basic 'LLM' that just echoes the input with a prefix."""
    return "You said: " + text_input.upper() + "!"

demo = gr.Interface(
    fn=simple_echo_llm,
    inputs="text",
    outputs="text",
    title="Simple Echo LLM Demo"
)
demo.launch()
```

What this code does:

  * Define a Python function (`simple_echo_llm`).
  * Pass this function directly to `gr.Interface` (or `gr.ChatInterface` if it's a conversational model).

### Integrating with Frameworks like LangChain or LlamaIndex

Frameworks like LangChain and LlamaIndex provide abstractions for building complex LLM applications (agents, RAG, etc.). You can integrate these within your Gradio function.

**LangChain**

```python
import gradio as gr
from langchain_openai import ChatOpenAI
from langchain.schema import HumanMessage, AIMessage
import os

# Set your OpenAI API key
os.environ["OPENAI_API_KEY"] = "YOUR_OPENAI_API_KEY"

model = ChatOpenAI(model="gpt-3.5-turbo") # Initialize LangChain LLM

def langchain_chat(message, history):
    # Convert Gradio history to LangChain's message format
    langchain_history = []
    for user_msg, ai_msg in history:
        langchain_history.append(HumanMessage(content=user_msg))
        langchain_history.append(AIMessage(content=ai_msg))
    langchain_history.append(HumanMessage(content=message))

    # Invoke the LangChain model
    response = model.invoke(langchain_history)
    return response.content

demo = gr.ChatInterface(
    fn=langchain_chat,
    title="LangChain Chatbot"
)
demo.launch()
```

What this code does:

  * Import necessary components from LangChain.
  * Initialize your LangChain `model` or `chain`.
  * The Gradio function wraps the LangChain call, handling the conversion of input/output formats.

### Considerations when Choosing a Backend

#### 1. Ease of Use/Quick Demo

* `gr.load()` for Hugging Face Inference Endpoints is the fastest way to get a demo running for many common models.
* `gr.Interface.from_pipeline()` for `transformers` pipelines is also very quick for local models.

#### 2. Specific Model Requirements

* For a specific LLM (e.g., a commercial one like GPT-4o, Claude 3.5 Sonnet, or Gemini 1.5 Pro), use the **API/SDK**.
* To fine-tuning or experimenting with a Hugging Face model locally, using `transformers.pipeline` directly gives the most control.

#### 3. Local vs. Cloud/API

* **Local Models:** Run on a local machine, require setup (dependencies, model weights), and leverage the local hardware (CPU/GPU). Good for privacy, cost control (after initial setup), and custom models.
* **Cloud/API Models:** Managed by a provider, require an API key and internet access. Easier to get started, scalable, but incurs usage costs and sends data to a third party.

#### 4. Complex LLM Applications

Building sophisticated applications involving retrieval-augmented generation (RAG), agents, tool use, etc., **LangChain or LlamaIndex** are potential choices to manage the complexity. These frameworks can be integrated into Gradio functions.


## Links

### Sources

* [MCP Specification](https://modelcontextprotocol.io/)
* [Antrophic MCP Quickstart](https://modelcontextprotocol.io/quickstart/server)
* [Huggingface MCP Tutorial](https://huggingface.co/learn/mcp-course/unit1/sdk)
* [FastMCP on GitHub](https://github.com/jlowin/fastmcp)
* [Gradio on GitHub](https://github.com/gradio-app/gradio)
* [Gradio Homepage](https://www.gradio.app/)


### Tools

* [NixOS](https://nixos.org/)
* [Home Manager](https://nix-community.github.io/home-manager/)
* [Devenv.sh](https://devenv.sh/)
* [Direnv](https://direnv.net/)
