# Examples

This page provides examples of how to use LLM Agent X, both from the command line and programmatically. For a more comprehensive set of examples, including sample outputs, please refer to the [`samples`](../samples/) directory in the repository.

## Command-Line Interface (CLI) Examples

These examples demonstrate common use cases for the `llm-agent-x` CLI tool. See the [CLI Documentation](./cli.md) for a full list of arguments.

1.  **Basic Research Task:**
    Ask the agent to research a topic. It will use its search tool and decompose the task as needed.
    ```sh
    llm-agent-x "Research the impact of renewable energy on climate change mitigation."
    ```

2.  **Saving Output to a File:**
    Use the `--output` flag to save the agent's final report to a specified file (within the `OUTPUT_DIR`).
    ```sh
    llm-agent-x "What are the latest developments in AI-driven drug discovery?" --output ai_drug_discovery.md
    ```

3.  **Using a Specific LLM Model:**
    Specify a particular language model for the agent to use (ensure it's accessible via your configuration).
    ```sh
    llm-agent-x "Explain the concept of blockchain technology to a non-technical audience." --model gpt-4-turbo
    ```

4.  **Controlling Task Decomposition:**
    Use `--task_limit` to control how many subtasks can be generated at each level.
    ```sh
    llm-agent-x "Develop a brief marketing plan for a new eco-friendly coffee shop." --task_limit "[2,2,0]"
    ```
    This allows 2 subtasks at the first level, 2 sub-subtasks for each of those, and no further decomposition.

5.  **Enabling Python Execution (with Sandbox):**
    Allow the agent to write and execute Python code. Ensure the [Python Sandbox](./sandbox.md) is running for safe execution.
    ```sh
    llm-agent-x "Write a Python script to calculate the factorial of 5 and explain the code." --enable-python-execution
    ```

6.  **Providing Specific User Instructions:**
    Use `--u_inst` to give more targeted instructions to the agent.
    ```sh
    llm-agent-x "Summarize the plot of the novel 'Dune'." --u_inst "Keep the summary under 500 words and focus on the main character's journey."
    ```

## Python API Example

This example demonstrates a basic programmatic use of `RecursiveAgent`. See the [API Documentation](./api.md) for more details.

```python
import asyncio
from llm_agent_x import RecursiveAgent, RecursiveAgentOptions, TaskLimit
from pydantic_ai.models.openai import OpenAIModel
from pydantic_ai.providers.openai import OpenAIProvider
from openai import AsyncOpenAI
from llm_agent_x.tools.brave_web_search import brave_web_search # Ensure this tool is available

# Ensure NLTK data is available (used indirectly)
import nltk
try:
    nltk.data.find('tokenizers/punkt')
except nltk.downloader.DownloadError:
    nltk.download('punkt', quiet=True)

async def run_agent_programmatically():
    # Configure LLM (ensure OPENAI_API_KEY is in your environment)
    client = AsyncOpenAI()
    llm = OpenAIModel("gpt-4o-mini", provider=OpenAIProvider(openai_client=client))

    # Configure Agent Options
    agent_options = RecursiveAgentOptions(
        llm=llm,
        tools=[brave_web_search],
        tools_dict={"web_search": brave_web_search, "brave_web_search": brave_web_search},
        task_limits=TaskLimit.from_array([2, 1, 0]), # Max 2 subtasks (L0), 1 subtask (L1)
        allow_search=True,
        allow_tools=True,
        mcp_servers=[], # No MCP servers for this basic example
    )

    # Define Task
    task = "Explore the pros and cons of remote work for software development teams."
    user_instructions = "Provide a balanced view and include at least three points for each side."

    # Create and Run Agent
    agent = RecursiveAgent(
        task=task,
        u_inst=user_instructions,
        agent_options=agent_options
    )

    print(f"Starting agent for task: \"{task}\"")
    try:
        result = await agent.run()
        print("\n--- Agent's Final Report ---")
        print(result)
        print(f"\nEstimated Cost: ${agent.cost:.4f}")
    except Exception as e:
        print(f"An error occurred during agent execution: {e}")

if __name__ == "__main__":
    asyncio.run(run_agent_programmatically())
```

## More Examples

For more detailed examples, including the structure of input tasks and the corresponding outputs generated by LLM Agent X, please see the files within the [`samples/`](../samples/) directory of the project repository. These samples often showcase more complex scenarios and different configurations.

The `samples/README.md` file in that directory may also provide additional context for the examples contained therein.
