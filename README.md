# Structured Output Agent using LangChain and LangGraph

This repository demonstrates how to build a **Structured Output Agent** using LangChain and LangGraph. The agent interacts with external tools (like web search APIs) and outputs structured responses validated through Pydantic schemas.

## Key Features

- **Structured Output**: The agent generates predictable, validated responses using Pydantic's `BaseModel`.
- **Tool Integration**: The agent uses tools like `TavilySearchResults` to fetch real-time data (e.g., city details).
- **State Management**: LangGraph orchestrates the workflow, managing tool calls and response generation.
- **Graph Visualization**: Workflow diagrams are provided using `graphviz`.

---

## Prerequisites

Before running the code, ensure you have the following:

1. **Python 3.8+**
2. Required Python packages:
   - `langchain`
   - `langgraph`
   - `pydantic`
   - `tavily-api`
   - `graphviz`

3. An API key for Tavily (to enable web search).

---

## Setup

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/Tanujkumar24/LANGGRAPH-STRUCTURED-OUTPUT-AGENT.git
   cd LANGGRAPH-STRUCTURED-OUTPUT-AGENT
   ```

2. **Install Dependencies**:
   Use `pip` to install the required libraries:
   ```bash
   pip install langchain langgraph pydantic tavily-api graphviz
   ```

3. **Set Up Tavily API Key**:
   Add your Tavily API key to the environment variables:
   ```bash
   export TAVILY_API_KEY="your-api-key"
   ```
   Or, update it in the code directly where needed.

---

## Code Walkthrough

### 1. **Structured Output Schema**
The `CityDetails` Pydantic model defines the structure of the agent's output:
```python
from pydantic import BaseModel, Field

class CityDetails(BaseModel):
    state_name: str = Field(description="State name of the city")
    state_capital: str = Field(description="State capital of the city")
    country_name: str = Field(description="Country name of the city")
    country_capital: str = Field(description="Country capital of the city")
```

### 2. **Tool Definition**
A tool to fetch real-time data using Tavily's web search:
```python
from langchain.tools import tool
from langchain_community.tools.tavily_search import TavilySearchResults

tavily_tool = TavilySearchResults()

@tool
def get_city_details(prompt):
    """Should do a web search to find the required city details"""
    response = tavily_tool.invoke(prompt)
    return response
```

### 3. **Workflow with LangGraph**
The agent's state transitions are defined using LangGraph:
```python
from langgraph.graph import StateGraph, END
from langchain.schema.runnable import Runnable, RunnableBranch
from langchain_core.messages import AIMessage, HumanMessage
from langchain.agents import ToolNode

# Workflow state management
def should_continue(state):
    messages = state["messages"]
    return "continue" if messages[-1].content.startswith("Tool:") else "respond"

# Adding nodes to the graph
workflow = StateGraph({"messages": list})
workflow.add_node("llm", call_model)
workflow.add_node("tools", ToolNode([get_city_details]))
workflow.add_node("respond", respond)
workflow.add_conditional_edges("llm", should_continue, {"continue": "tools", "respond": "respond"})
workflow.add_edge("tools", "llm")
workflow.add_edge("respond", END)
```

### 4. **Graph Visualization**
Use Graphviz to visualize the workflow:
```python
from graphviz import Digraph

# Generate graph visualization
def visualize_workflow():
    dot = Digraph(comment="Structured Agent Workflow")

    # Nodes
    dot.node("START", "Start")
    dot.node("LLM", "LLM (Generate Prompt)")
    dot.node("TOOLS", "Tools (Web Search API)")
    dot.node("RESPOND", "Respond (Structured Output)")
    dot.node("END", "End")

    # Edges
    dot.edge("START", "LLM", label="Initial Query")
    dot.edge("LLM", "TOOLS", label="Call Tools (if required)")
    dot.edge("TOOLS", "LLM", label="Return Results")
    dot.edge("LLM", "RESPOND", label="Generate Response")
    dot.edge("RESPOND", "END", label="Final Output")

    # Save and render graph
    dot.render("workflow_graph", format="png", cleanup=True)
    print("Workflow graph generated as 'workflow_graph.png'")

# Run visualization
visualize_workflow()
```

### 5. **Invocation**
Run the graph to query city details:
```python
if __name__ == "__main__":
    graph = workflow.compile()
    answer = graph.invoke(
        input={"messages": [("human", "Tell me about the city details for gwalior?")]}
    )["final_response"]
    print(answer)
```

---

## Example Output

**Input**: "Tell me about the city details for Gwalior?"

**Output**:
```json
{
    "state_name": "Madhya Pradesh",
    "state_capital": "Bhopal",
    "country_name": "India",
    "country_capital": "New Delhi"
}
```

---

## Running the Code

1. Run the Python script:
   ```bash
   python main.py
   ```
2. Enter prompts like:
   - "Tell me about the city details for Bangalore."
   - "What are the details of Mumbai?"

The agent will fetch real-time data and output it in a structured format.

---

## Workflow Diagram
After running the visualization code, a **workflow_graph.png** file will be generated. Here's the structure:

![Workflow Diagram](https://github.com/Tanujkumar24/LANGGRAPH-STRUCTURED-OUTPUT-AGENT/blob/main/graph.jpg)

---

## Contributions
Feel free to fork the repository and submit pull requests for improvements.

---

## License
This project is licensed under the MIT License.

---

## Contact
For any queries or suggestions, please reach out to:
- **Name**: Your Name
- **Email**: your.email@example.com
- **Portfolio**: [Your Portfolio](https://example.com)
