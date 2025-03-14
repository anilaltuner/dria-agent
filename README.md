# Dria Agent

`tiny-agent-α` is a tiny model for building tool calling agents on edge devices.

It's fast and veeeery good compared to it's size.

Demo:

https://github.com/user-attachments/assets/909656ad-4924-435a-8b4d-ac1b6d664d9c


### Features

Tiny-Agent-α is an extension of [Dria-Agent-a](https://huggingface.co/collections/driaforall/dria-agent-a-67a61f4b7d3d544fe5d3cd8a=), trained on top of the Qwen2.5-Coder series to be used in edge devices. 
These models are carefully fine-tuned with quantization aware training to minimize performance degradation after quantization. 
The smallest model is 0.5B with 4bit quantization (398MB on disk), and the largest model is 3B with 4bit quantization.

It's good at:

- **One-shot Parallel Multiple Function Calls**

- **Free-form Reasoning and Actions**

- **On-the-fly Complex Solution Generation**

#### Demo:

https://github.com/user-attachments/assets/5f7cbd26-7ba3-46aa-926f-4ac68de5ccb0

#### Edge Device Optimized:
- Supports mlx, ollama, and transformers (Hugging Face).
- Includes built-in support for macOS, Gmail, search, and more.
- Uses similarity search to efficiently select relevant tools.
- Optimized for Edge

`tiny-agent-a-0.5b` gets a whopping 72 on the _DPAB_ benchmark and run with `183.49 tokens/s` on a M1 macbook pro. Yet it's only **530MB**!

### Installation

To install the package run:
```bash
pip install dria_agent # Best for CPU inference, uses ollama
pip install 'dria_agent[mcp]' # To use MCP tools
pip install 'dria_agent[mlx]' # To use MLX as backend for macOS. 
pip install 'dria_agent[huggingface]' # HuggingFace/transformers backend for GPU.
pip install 'dria_agent[mlx, tools]' # In order to use factory tools in package, run with backend of your choice
```

### Quick Start

#### CLI Mode

You can run the agent with pre-defined [tools](#tool-library) using the CLI. Agent will use all of the tools in the library.
For CLI, you should install tools with backend of your choice

```bash
pip install 'dria_agent[ollama, tools]'
```

For using MCP tools on cli, you need to run

```bash
pip install 'dria_agent[ollama, tools, mcp]'
```

And then, run:

```bash
dria_agent --chat  # for chat mode
dria_agent Please solve 5x^2 + 8x + 9 = 0 and 4x^2 + 11x - 3 = 0 # for single query
```

For running MPC from cli mode;

```bash
dria_agent --mcp_path mcp.json query search term synthetic data
dria_agent --chat --mcp_path mcp.json 
```

For help, `dria_agent --help`
```
dria_agent [-h] [--chat] [--mcp_path ...] [--backend {mlx,ollama,huggingface}]
                  [--agent_mode {ultra_light,fast,balanced,performant}]
                  [query ...]
```

#### Using your own tools

Write your functions in pure python, decorate them with @tool to expose them to the agent.

````python
from dria_agent import tool

@tool
def check_availability(day: str, start_time: str, end_time: str) -> bool:
    """
    Checks if a given time slot is available.

    :param day: The date in "YYYY-MM-DD" format.
    :param start_time: The start time of the desired slot (HH:MM format, 24-hour).
    :param end_time: The end time of the desired slot (HH:MM format, 24-hour).
    :return: True if the slot is available, otherwise False.
    """
    # Mock implementation
    if start_time == "12:00" and end_time == "13:00":
        return False
    return True

````

Create an agent:

```python
from dria_agent import ToolCallingAgent

agent = ToolCallingAgent(
    tools=[check_availability]
)

```

Use agent.run(query) to execute tasks with synchronous tools.

```python
execution = agent.run("Check my calendar for tomorrow noon", print_results=True)
```

For using asynchronous tools or MCP, use agent.async_run(query).

```python
execution = await agent.async_run("Check my calendar for tomorrow noon", print_results=True)
```

#### Model Context Protocol (MCP) Support

The agent supports MCP, which allows you to use tools from any MCP-compatible server.

To use MCP, write JSON file and pass it to the agent class.

Fetch Server Example:
```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```

Run Agent with MCP:

```python
from dria_agent import ToolCallingAgent
from dria_agent.agent.mcp.tool_adapter import MCPToolAdapter


async def run_agent():
    """Run agent with MCP"""
    
    agent = ToolCallingAgent(mcp_file="mcp.json", backend="ollama")

    query = "fetch google.com"
    
    try:
      # server initialization is needed on MCP
      await agent.initialize_servers()
      execution = await agent.async_run(query, print_results=True)
    finally:
      await agent.close_servers()

if __name__ == "__main__":
    asyncio.run(run_agent())
```

#### Run Modes

Agent has 4 modes to choose from, depending on your needs:

- **Ultra Light**: Fastest inference, uses the least amount of memory.
- **Fast**: Faster inference, uses more memory.
- **Balanced**: Balanced between speed and memory.
- **Performant**: Best performance, uses the most memory.

To initialize the agent with a specific mode:

```python
agent = ToolCallingAgent(tools=[my_tool], backend="ollama", mode="ultra_light")
```
---

`agent.run()`

- **query (str)**: The user query to process.
- **dry_run (bool, default=False)**: If True, only performs inference—no tool execution.
- **show_completion (bool, default=True)**: Displays the model’s raw output before tool execution.
- **num_tools (int, default=2)**: Selects the best K tools for inference (using similarity search).
  - *Allows handling thousands of tools efficiently*.
  - * perform best with 4-5 tools max*.
- **print_results (bool, default=True)**: Prints execution results.

---

`agent.run_feedback()`

Same as run, but if there are errors in the execution, it will feed the errors back until execution is successful.

#### Tool Library

See [tool's library](dria_agent/tools/library/__init__.py) for implemented tools.


## Models

A fast and powerful tool calling model designed to run on edge devices.

| Model                  | Description                                | HF Download Link                                                                                                         | Ollama Tag                         | Size   |
|------------------------|--------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|-------------------------------------|--------|
| Tiny-Agent-a-3B (8bit) | High performance and reasoning             | [Download](https://huggingface.co/driaforall/Tiny-Agent-a-3B/resolve/main/dria-agent-a-3b.Q8_0.gguf?download=true)       | driaforall/tiny-agent-a:3B-q8_0  | 3.3 GB |
| Tiny-Agent-a-3B (4bit) | Tradeoff 3B quality for memory             | [Download](https://huggingface.co/driaforall/Tiny-Agent-a-3B/resolve/main/dria-agent-a-3b.Q4_K_M.gguf?download=true)     | driaforall/tiny-agent-a:3B-q4_K_M | 1.9 GB |
| Tiny-Agent-a-1.5B (8bit) | Balanced performance and speed             | [Download](https://huggingface.co/driaforall/Tiny-Agent-a-1.5B/resolve/main/dria-agent-a-1.5b.Q8_0.gguf?download=true)   | driaforall/tiny-agent-a:1.5B-q8_0 | 1.6 GB |
| Tiny-Agent-a-1.5B (4bit) | Faster CPU inference, performance tradeoff | [Download](https://huggingface.co/driaforall/Tiny-Agent-a-1.5B/resolve/main/dria-agent-a-1.5b.Q8_0.gguf?download=true)   | driaforall/tiny-agent-a:1.5B-q4_K_M | 986 MB |
| Tiny-Agent-a-0.5B (8bit) | Ultra-light                                | [Download](https://huggingface.co/driaforall/Tiny-Agent-a-1.5B/resolve/main/dria-agent-a-1.5b.Q4_K_M.gguf?download=true) | driaforall/tiny-agent-a:0.5B-q8_0 | 531 MB |


## Evaluation & Performance

We evaluate the model on the **Dria-Pythonic-Agent-Benchmark ([DPAB](https://github.com/firstbatchxyz/function-calling-eval)):** The benchmark we curated with a synthetic data generation +model-based validation + filtering and manual selection to evaluate LLMs on their Pythonic function calling ability, spanning multiple scenarios and tasks. See [blog](https://huggingface.co/blog/andthattoo/dpab-a) for more information.

Below are the DPAB results: 

Current benchmark results for various models **(strict)**:

| Model Name                      | Pythonic | JSON |
|---------------------------------|----------|------|
| **Closed Models**               |          |      |
| Claude 3.5 Sonnet              | 87       | 45   |
| gpt-4o-2024-11-20              | 60       | 30   |
| **Open Models**                 |          |      |
| **> 100B Parameters**           |          |      |
| DeepSeek V3 (685B)             | 63       | 33   |
| MiniMax-01                     | 62       | 40   |
| Llama-3.1-405B-Instruct        | 60       | 38   |
| **> 30B Parameters**            |          |      |
| Qwen-2.5-Coder-32b-Instruct    | 68       | 32   |
| Qwen-2.5-72b-instruct          | 65       | 39   |
| Llama-3.3-70b-Instruct         | 59       | 40   |
| QwQ-32b-Preview                | 47       | 21   |
| **< 20B Parameters**           |          |      |
| Phi-4 (14B)                    | 55       | 35   |
| Qwen2.5-Coder-7B-Instruct      | 44       | 39   |
| Qwen-2.5-7B-Instruct           | 47       | 34   |
| **Tiny-Agent-a-3B**               | **72**       | 34   |
| Qwen2.5-Coder-3B-Instruct      | 26       | 37   |
| **Tiny-Agent-a-1.5B**               | **73**       | 30   |


#### Citation

```
@misc{Dria-Agent-a,
      url={https://huggingface.co/blog/andthattoo/dria-agent-a},
      title={Dria-Agent-a},
      author={"andthattoo", "Atakan Tekparmak"}
}
```

