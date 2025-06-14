# Python A2A

<div align="center">

[![PyPI version](https://img.shields.io/pypi/v/python-a2a.svg)](https://pypi.org/project/python-a2a/)
[![Python Versions](https://img.shields.io/pypi/pyversions/python-a2a.svg)](https://pypi.org/project/python-a2a/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PyPI Downloads](https://static.pepy.tech/badge/python-a2a)](https://pepy.tech/project/python-a2a)
[![Documentation Status](https://readthedocs.org/projects/python-a2a/badge/?version=latest)](https://python-a2a.readthedocs.io/en/latest/?badge=latest)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Imports: isort](https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336)](https://pycqa.github.io/isort/)
[![UV Compatible](https://img.shields.io/badge/UV-Compatible-5C63FF.svg)](https://github.com/astral-sh/uv)


**The Definitive Python Implementation of Google's Agent-to-Agent (A2A) Protocol with Model Context Protocol (MCP) Integration**

</div>

## 🌟 Overview

Python A2A is a comprehensive, production-ready library for implementing Google's [Agent-to-Agent (A2A) protocol](https://google.github.io/A2A/) with full support for the [Model Context Protocol (MCP)](https://contextual.ai/introducing-mcp/). It provides everything you need to build interoperable AI agent ecosystems that can collaborate seamlessly to solve complex problems.

The A2A protocol establishes a standard communication format that enables AI agents to interact regardless of their underlying implementation, while MCP extends this capability by providing a standardized way for agents to access external tools and data sources. Python A2A makes these protocols accessible with an intuitive API that developers of all skill levels can use to build sophisticated multi-agent systems.

## 📋 What's New in v0.5.X

- **Agent Discovery**: Built-in support for agent registry and discovery with full Google A2A protocol compatibility
- **LangChain Integration**: Seamless integration with LangChain's tools and agents
- **Expanded Tool Ecosystem**: Use tools from both LangChain and MCP in any agent
- **Enhanced Agent Interoperability**: Convert between A2A agents and LangChain agents 
- **Mixed Workflow Engine**: Build workflows combining both ecosystems
- **Simplified Agent Development**: Access thousands of pre-built tools instantly
- **Advanced Streaming Architecture**: Enhanced streaming with Server-Sent Events (SSE), better error handling, and robust fallback mechanisms
- **Task-Based Streaming**: New `tasks_send_subscribe` method for streaming task updates in real-time
- **Streaming Chunks API**: Improved chunk processing with the `StreamingChunk` class for structured streaming data
- **Multi-Endpoint Support**: Automatic discovery and fallback across multiple streaming endpoints

## 📋 What's New in v0.4.X

- **Agent Network System**: Manage and discover multiple agents with the new `AgentNetwork` class
- **Real-time Streaming**: Implement streaming responses with `StreamingClient` for responsive UIs
- **Workflow Engine**: Define complex multi-agent workflows using the new fluent API with conditional branching and parallel execution
- **AI-Powered Router**: Automatically route queries to the most appropriate agent with the `AIAgentRouter`
- **Command Line Interface**: Control your agents from the terminal with the new CLI tool
- **Enhanced Asynchronous Support**: Better async/await support throughout the library
- **New Connection Options**: Improved error handling and retry logic for more robust agent communication

## ✨ Why Choose Python A2A?

- **Complete Implementation**: Fully implements the official A2A specification with zero compromises
- **Agent Discovery**: Built-in agent registry and discovery for building agent ecosystems
- **MCP Integration**: First-class support for Model Context Protocol for powerful tool-using agents
- **Enterprise Ready**: Built for production environments with robust error handling and validation
- **Framework Agnostic**: Works with any Python framework (Flask, FastAPI, Django, etc.)
- **LLM Provider Flexibility**: Native integrations with OpenAI, Anthropic, AWS Bedrock, and more
- **Minimal Dependencies**: Core functionality requires only the `requests` library
- **Excellent Developer Experience**: Comprehensive documentation, type hints, and examples

## 📦 Installation

### Using pip (traditional)

Install the base package with all dependencies:

```bash
pip install python-a2a  # Includes LangChain, MCP, and other integrations
```

Or install with specific components based on your needs:

```bash
# For Flask-based server support
pip install "python-a2a[server]"

# For OpenAI integration
pip install "python-a2a[openai]"

# For Anthropic Claude integration
pip install "python-a2a[anthropic]"

# For AWS-Bedrock integration
pip install "python-a2a[bedrock]"

# For MCP support (Model Context Protocol)
pip install "python-a2a[mcp]"

# For all optional dependencies
pip install "python-a2a[all]"
```

### Using UV (recommended)

[UV](https://github.com/astral-sh/uv) is a modern Python package management tool that's faster and more reliable than pip. To install with UV:

```bash
# Install UV if you don't have it already
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install the base package
uv install python-a2a
```

### Development Installation

For development, UV is recommended for its speed:

```bash
# Clone the repository
git clone 
cd python-a2a

# Create a virtual environment and install development dependencies
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
uv pip install -e ".[dev]"
```

> 💡 **Tip**: Click the code blocks to copy them to your clipboard.

## 🚀 Quick Start Examples

### 1. Create a Simple A2A Agent with Skills

```python
from python_a2a import A2AServer, skill, agent, run_server, TaskStatus, TaskState

@agent(
    name="Weather Agent",
    description="Provides weather information",
    version="1.0.0"
)
class WeatherAgent(A2AServer):
    
    @skill(
        name="Get Weather",
        description="Get current weather for a location",
        tags=["weather", "forecast"]
    )
    def get_weather(self, location):
        """Get weather for a location."""
        # Mock implementation
        return f"It's sunny and 75°F in {location}"
    
    def handle_task(self, task):
        # Extract location from message
        message_data = task.message or {}
        content = message_data.get("content", {})
        text = content.get("text", "") if isinstance(content, dict) else ""
        
        if "weather" in text.lower() and "in" in text.lower():
            location = text.split("in", 1)[1].strip().rstrip("?.")
            
            # Get weather and create response
            weather_text = self.get_weather(location)
            task.artifacts = [{
                "parts": [{"type": "text", "text": weather_text}]
            }]
            task.status = TaskStatus(state=TaskState.COMPLETED)
        else:
            task.status = TaskStatus(
                state=TaskState.INPUT_REQUIRED,
                message={"role": "agent", "content": {"type": "text", 
                         "text": "Please ask about weather in a specific location."}}
            )
        return task

# Run the server
if __name__ == "__main__":
    agent = WeatherAgent()
    run_server(agent, port=5000)
```

### 2. Build an Agent Network with Multiple Agents

```python
from python_a2a import AgentNetwork, A2AClient, AIAgentRouter

# Create an agent network
network = AgentNetwork(name="Travel Assistant Network")

# Add agents to the network
network.add("weather", "http://localhost:5001")
network.add("hotels", "http://localhost:5002")
network.add("attractions", "http://localhost:5003")

# Create a router to intelligently direct queries to the best agent
router = AIAgentRouter(
    llm_client=A2AClient("http://localhost:5000/openai"),  # LLM for making routing decisions
    agent_network=network
)

# Route a query to the appropriate agent
agent_name, confidence = router.route_query("What's the weather like in Paris?")
print(f"Routing to {agent_name} with {confidence:.2f} confidence")

# Get the selected agent and ask the question
agent = network.get_agent(agent_name)
response = agent.ask("What's the weather like in Paris?")
print(f"Response: {response}")

# List all available agents
print("\nAvailable Agents:")
for agent_info in network.list_agents():
    print(f"- {agent_info['name']}: {agent_info['description']}")
```

### Real-time Streaming

Get real-time responses from agents with comprehensive streaming support:

```python
import asyncio
from python_a2a import StreamingClient, Message, TextContent, MessageRole

async def main():
    client = StreamingClient("http://localhost:5000")
    
    # Create a message with required role parameter
    message = Message(
        content=TextContent(text="Tell me about A2A streaming"),
        role=MessageRole.USER
    )
    
    # Stream the response and process chunks in real-time
    try:
        async for chunk in client.stream_response(message):
            # Handle different chunk formats (string or dictionary)
            if isinstance(chunk, dict):
                if "content" in chunk:
                    print(chunk["content"], end="", flush=True)
                elif "text" in chunk:
                    print(chunk["text"], end="", flush=True)
                else:
                    print(str(chunk), end="", flush=True)
            else:
                print(str(chunk), end="", flush=True)
    except Exception as e:
        print(f"Streaming error: {e}")
```

Check out the `examples/streaming/` directory for complete streaming examples:

- **basic_streaming.py**: Minimal streaming implementation (start here!)
- **01_basic_streaming.py**: Comprehensive introduction to streaming basics
- **02_advanced_streaming.py**: Advanced streaming with different chunking strategies
- **03_streaming_llm_integration.py**: Integrating streaming with LLM providers
- **04_task_based_streaming.py**: Task-based streaming with progress tracking
- **05_streaming_ui_integration.py**: Streaming UI integration (CLI and web)
- **06_distributed_streaming.py**: Distributed streaming architecture

### 3. Workflow Engine

The new workflow engine allows you to define complex agent interactions:

```python
from python_a2a import AgentNetwork, Flow
import asyncio

async def main():
    # Set up agent network
    network = AgentNetwork()
    network.add("research", "http://localhost:5001")
    network.add("summarizer", "http://localhost:5002")
    network.add("factchecker", "http://localhost:5003")
    
    # Define a workflow for research report generation
    flow = Flow(agent_network=network, name="Research Report Workflow")
    
    # First, gather initial research
    flow.ask("research", "Research the latest developments in {topic}")
    
    # Then process the results in parallel
    parallel_results = (flow.parallel()
        # Branch 1: Create a summary
        .ask("summarizer", "Summarize this research: {latest_result}")
        # Branch 2: Verify key facts
        .branch()
        .ask("factchecker", "Verify these key facts: {latest_result}")
        # End parallel processing and collect results
        .end_parallel(max_concurrency=2))
    
    # Extract insights based on verification results
    flow.execute_function(
        lambda results, context: f"Summary: {results['1']}\nVerified Facts: {results['2']}",
        parallel_results
    )
    
    # Execute the workflow
    result = await flow.run({
        "topic": "quantum computing advancements in the last year"
    })
    
    print(result)

if __name__ == "__main__":
    asyncio.run(main())
```

### 4. AI-Powered Router

Intelligent routing to select the best agent for each query:

```python
from python_a2a import AgentNetwork, AIAgentRouter, A2AClient
import asyncio

async def main():
    # Create a network with specialized agents
    network = AgentNetwork()
    network.add("math", "http://localhost:5001")
    network.add("history", "http://localhost:5002")
    network.add("science", "http://localhost:5003")
    network.add("literature", "http://localhost:5004")
    
    # Create a router using an LLM for decision making
    router = AIAgentRouter(
        llm_client=A2AClient("http://localhost:5000/openai"),
        agent_network=network
    )
    
    # Sample queries to route
    queries = [
        "What is the formula for the area of a circle?",
        "Who wrote The Great Gatsby?",
        "When did World War II end?",
        "How does photosynthesis work?",
        "What are Newton's laws of motion?"
    ]
    
    # Route each query to the best agent
    for query in queries:
        agent_name, confidence = router.route_query(query)
        agent = network.get_agent(agent_name)
        
        print(f"Query: {query}")
        print(f"Routed to: {agent_name} (confidence: {confidence:.2f})")
        
        # Get response from the selected agent
        response = agent.ask(query)
        print(f"Response: {response[:100]}...\n")

if __name__ == "__main__":
    asyncio.run(main())
```

### 5. Define Complex Workflows with Multiple Agents

```python
from python_a2a import AgentNetwork, Flow, AIAgentRouter
import asyncio

async def main():
    # Create an agent network
    network = AgentNetwork()
    network.add("weather", "http://localhost:5001")
    network.add("recommendations", "http://localhost:5002")
    network.add("booking", "http://localhost:5003")
    
    # Create a router
    router = AIAgentRouter(
        llm_client=network.get_agent("weather"),  # Using one agent as LLM for routing
        agent_network=network
    )
    
    # Define a workflow with conditional logic
    flow = Flow(agent_network=network, router=router, name="Travel Planning Workflow")
    
    # Start by getting the weather
    flow.ask("weather", "What's the weather in {destination}?")
    
    # Conditionally branch based on weather
    flow.if_contains("sunny")
    
    # If sunny, recommend outdoor activities
    flow.ask("recommendations", "Recommend outdoor activities in {destination}")
    
    # End the condition and add an else branch
    flow.else_branch()
    
    # If not sunny, recommend indoor activities
    flow.ask("recommendations", "Recommend indoor activities in {destination}")
    
    # End the if-else block
    flow.end_if()
    
    # Add parallel processing steps
    (flow.parallel()
        .ask("booking", "Find hotels in {destination}")
        .branch()
        .ask("booking", "Find restaurants in {destination}")
        .end_parallel())
    
    # Execute the workflow with initial context
    result = await flow.run({
        "destination": "Paris",
        "travel_dates": "June 12-20"
    })
    
    print("Workflow result:")
    print(result)

if __name__ == "__main__":
    asyncio.run(main())
```

### 6. Use the Command Line Interface

```bash
# Send a message to an agent
a2a send http://localhost:5000 "What is artificial intelligence?"

# Stream a response in real-time
a2a stream http://localhost:5000 "Generate a step-by-step tutorial for making pasta"

# Start an OpenAI-powered A2A server
a2a openai --model gpt-4 --system-prompt "You are a helpful coding assistant"

# Start an Anthropic-powered A2A server
a2a anthropic --model claude-3-opus-20240229 --system-prompt "You are a friendly AI teacher"

# Start an MCP server with tools
a2a mcp-serve --name "Data Analysis MCP" --port 5001 --script analysis_tools.py

# Start an MCP-enabled A2A agent
a2a mcp-agent --servers data=http://localhost:5001 calc=http://localhost:5002

# Call an MCP tool directly
a2a mcp-call http://localhost:5001 analyze_csv --params file=data.csv columns=price,date

# Manage agent networks
a2a network --add weather=http://localhost:5001 travel=http://localhost:5002 --save network.json

# Run a workflow from a script
a2a workflow --script research_workflow.py --context initial_data.json
```

## 🔄 LangChain Integration (New in v0.5.X)

Python A2A includes built-in LangChain integration, making it easy to combine the best of both ecosystems:

### 1. Converting MCP Tools to LangChain

```python
from python_a2a.mcp import FastMCP, text_response
from python_a2a.langchain import to_langchain_tool

# Create MCP server with a tool
mcp_server = FastMCP(name="Basic Tools", description="Simple utility tools")

@mcp_server.tool(
    name="calculator",
    description="Calculate a mathematical expression"
)
def calculator(input):
    """Simple calculator that evaluates an expression."""
    try:
        result = eval(input)
        return text_response(f"Result: {result}")
    except Exception as e:
        return text_response(f"Error: {e}")

# Start the server
import threading, time
def run_server(server, port):
    server.run(host="0.0.0.0", port=port)
server_thread = threading.Thread(target=run_server, args=(mcp_server, 5000), daemon=True)
server_thread.start()
time.sleep(2)  # Allow server to start

# Convert MCP tool to LangChain
calculator_tool = to_langchain_tool("http://localhost:5000", "calculator")

# Use the tool in LangChain
result = calculator_tool.run("5 * 9 + 3")
print(f"Result: {result}")
```

### 2. Converting LangChain Tools to MCP Server

```python
from langchain.tools import Tool
from langchain_core.tools import BaseTool
from python_a2a.langchain import to_mcp_server

# Create LangChain tools
def calculator(expression: str) -> str:
    """Evaluate a mathematical expression"""
    try:
        result = eval(expression)
        return f"Result: {expression} = {result}"
    except Exception as e:
        return f"Error: {e}"

calculator_tool = Tool(
    name="calculator",
    description="Evaluate a mathematical expression",
    func=calculator
)

# Convert to MCP server
mcp_server = to_mcp_server(calculator_tool)

# Run the server
mcp_server.run(port=5000)
```

### 3. Converting LangChain Components to A2A Servers

```python
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
from python_a2a import A2AClient, run_server
from python_a2a.langchain import to_a2a_server

# Create a LangChain LLM
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)

# Convert LLM to A2A server
llm_server = to_a2a_server(llm)

# Create a simple chain
template = "You are a helpful travel guide.\n\nQuestion: {query}\n\nAnswer:"
prompt = PromptTemplate.from_template(template)
travel_chain = prompt | llm | StrOutputParser()

# Convert chain to A2A server
travel_server = to_a2a_server(travel_chain)

# Run servers in background threads
import threading
llm_thread = threading.Thread(
    target=lambda: run_server(llm_server, port=5001),
    daemon=True
)
llm_thread.start()

travel_thread = threading.Thread(
    target=lambda: run_server(travel_server, port=5002),
    daemon=True
)
travel_thread.start()

# Test the servers
llm_client = A2AClient("http://localhost:5001")
travel_client = A2AClient("http://localhost:5002")

llm_result = llm_client.ask("What is the capital of France?")
travel_result = travel_client.ask('{"query": "What are some must-see attractions in Paris?"}')
```

### 4. Converting A2A Agents to LangChain Agents

```python
from python_a2a.langchain import to_langchain_agent

# Convert A2A agent to LangChain agent
langchain_agent = to_langchain_agent("http://localhost:5000")

# Use the agent in LangChain
result = langchain_agent.invoke("What are some famous landmarks in Paris?")
print(result.get('output', ''))

# Use in a LangChain pipeline
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

llm = ChatOpenAI(temperature=0)
prompt = ChatPromptTemplate.from_template(
    "Generate a specific, detailed travel question about {destination}."
)

# Create a pipeline with the converted agent
chain = (
    prompt |
    llm |
    StrOutputParser() |
    langchain_agent |
    (lambda x: f"Travel Info: {x.get('output', '')}")
)

result = chain.invoke({"destination": "Japan"})
print(result)
```

LangChain is automatically installed as a dependency with python-a2a, so everything works right out of the box:

```bash
pip install python-a2a
# That's it! LangChain is included automatically
```

## 🧩 Core Features

### Agent Networks

Python A2A now includes a powerful system for managing multiple agents:

```python
from python_a2a import AgentNetwork, A2AClient

# Create a network of agents
network = AgentNetwork(name="Medical Assistant Network")

# Add agents in different ways
network.add("diagnosis", "http://localhost:5001")  # From URL
network.add("medications", A2AClient("http://localhost:5002"))  # From client instance

# Discover agents from a list of URLs
discovered_count = network.discover_agents([
    "http://localhost:5003",
    "http://localhost:5004",
    "http://localhost:5005"
])
print(f"Discovered {discovered_count} new agents")

# List all agents in the network
for agent_info in network.list_agents():
    print(f"Agent: {agent_info['name']}")
    print(f"URL: {agent_info['url']}")
    if 'description' in agent_info:
        print(f"Description: {agent_info['description']}")
    print()

# Get a specific agent
agent = network.get_agent("diagnosis")
response = agent.ask("What are the symptoms of the flu?")
```

### 7. Agent Discovery and Registry

```python
from python_a2a import AgentCard, A2AServer, run_server
from python_a2a.discovery import AgentRegistry, run_registry, enable_discovery, DiscoveryClient
import threading
import time

# Create a registry server
registry = AgentRegistry(
    name="A2A Registry Server",
    description="Central registry for agent discovery"
)

# Run the registry in a background thread
registry_port = 8000
thread = threading.Thread(
    target=lambda: run_registry(registry, host="0.0.0.0", port=registry_port),
    daemon=True
)
thread.start()
time.sleep(1)  # Let the registry start

# Create a sample agent
agent_card = AgentCard(
    name="Weather Agent",
    description="Provides weather information",
    url="http://localhost:8001",
    version="1.0.0",
    capabilities={
        "weather_forecasting": True,
        "google_a2a_compatible": True  # Enable Google A2A compatibility
    }
)
agent = A2AServer(agent_card=agent_card)

# Enable discovery - this registers with the registry
registry_url = f"http://localhost:{registry_port}"
discovery_client = enable_discovery(agent, registry_url=registry_url)

# Run the agent in a separate thread
agent_thread = threading.Thread(
    target=lambda: run_server(agent, host="0.0.0.0", port=8001),
    daemon=True
)
agent_thread.start()
time.sleep(1)  # Let the agent start

# Create a discovery client for discovering agents
client = DiscoveryClient(agent_card=None)  # No agent card needed for discovery only
client.add_registry(registry_url)

# Discover all agents
agents = client.discover()
print(f"Discovered {len(agents)} agents:")
for agent in agents:
    print(f"- {agent.name} at {agent.url}")
    print(f"  Capabilities: {agent.capabilities}")
```

## 📖 Architecture & Design Principles

Python A2A is built on three core design principles:

1. **Protocol First**: Adheres strictly to the A2A and MCP protocol specifications for maximum interoperability

2. **Modularity**: All components are designed to be composable and replaceable

3. **Progressive Enhancement**: Start simple and add complexity only as needed

The architecture consists of eight main components:

- **Models**: Data structures representing A2A messages, tasks, and agent cards
- **Client**: Components for sending messages to A2A agents and managing agent networks
- **Server**: Components for building A2A-compatible agents
- **Discovery**: Registry and discovery mechanisms for agent ecosystems
- **MCP**: Tools for implementing Model Context Protocol servers and clients
- **LangChain**: Bridge components for LangChain integration
- **Workflow**: Engine for orchestrating complex agent interactions
- **Utils**: Helper functions for common tasks
- **CLI**: Command-line interface for interacting with agents

## 🗺️ Use Cases

Python A2A can be used to build a wide range of AI systems:

### Research & Development

- **Experimentation Framework**: Easily swap out different LLM backends while keeping the same agent interface
- **Benchmark Suite**: Compare performance of different agent implementations on standardized tasks
- **Streaming Research Assistants**: Create responsive research tools with real-time output using streaming

### Enterprise Systems

- **AI Orchestration**: Coordinate multiple AI agents across different departments using agent networks
- **Legacy System Integration**: Wrap legacy systems with A2A interfaces for AI accessibility
- **Complex Workflows**: Create sophisticated business processes with multi-agent workflows and conditional branching

### Customer-Facing Applications

- **Multi-Stage Assistants**: Break complex user queries into subtasks handled by specialized agents
- **Tool-Using Agents**: Connect LLMs to database agents, calculation agents, and more using MCP
- **Real-time Chat Interfaces**: Build responsive chat applications with streaming response support

### Education & Training

- **AI Education**: Create educational systems that demonstrate agent collaboration
- **Simulation Environments**: Build simulated environments where multiple agents interact
- **Educational Workflows**: Design step-by-step learning processes with feedback loops

## 🛠️ Real-World Examples


- Multi-agent customer support systems
- LLM-powered research assistants with tool access
- Real-time streaming implementations
- LangChain integration examples
- MCP server implementations for various tools
- Workflow orchestration examples
- Agent network management

## 🔄 Related Projects

Here are some related projects in the AI agent and interoperability space:

- [**Google A2A**](https://github.com/google/A2A) - The official Google A2A protocol specification
- [**LangChain**](https://github.com/langchain-ai/langchain) - Framework for building applications with LLMs
- [**AutoGen**](https://github.com/microsoft/autogen) - Microsoft's framework for multi-agent conversations
- [**CrewAI**](https://github.com/joaomdmoura/crewAI) - Framework for orchestrating role-playing agents
- [**MCP**](https://github.com/contextco/mcp) - The Model Context Protocol for tool-using agents


## 📝 Citing this Project

If you use Python A2A in your research or academic work, please cite it as:

```
@software{Arifraihan,
  author = {Arifuzzaman, Raihan},
  title = {Python A2A: A Comprehensive Implementation of the Agent-to-Agent Protocol},
  version = {0.5.0},
  year = {2025},
}
```

## ⭐ Star This Repository

If you find this library useful, please consider giving it a star on GitHub! It helps others discover the project and motivates further development.


## 🙏 Acknowledgements

- The [Google A2A team](https://github.com/google/A2A) for creating the A2A protocol
- The [Contextual AI team](https://contextual.ai/) for the Model Context Protocol
- The [LangChain team](https://github.com/langchain-ai) for their powerful LLM framework

## 👨‍💻 Author

**Arifuzzaman Raihan**

- GitHub: [Arifuzzaman Raihan](https://github.com/Arifuzzaman10)

---

Made with ❤️ by [Arifuzzaman Raihan](https://github.com/Arifuzzaman10)
