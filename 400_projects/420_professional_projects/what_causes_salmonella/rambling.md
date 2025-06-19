## Rambling
- Brainstorm how to handle comments within NORS
- Ramblings about time series
	- A smoothed time period e.g. a monthly progression would be better?
	- Model the temperature
	- Think about the occurence of salmonella as a process (Salmonella inception -> Salmonella observation)
- Is it possible to infer location of contamination?
- Can we get outputs in terms of probabilities?
- How can we impute missing data?
- Maybe we can use LllamaIndex/Llamaparse to
	- Clean and Merge datasets.
	- add the data documentations.
- Provide a list of URLs the LLM should scrape when trying to answer a question.
- Find all APIs for datasets.
- Need to discuss how to incorporate data on same thing e.g. poultry sampling, but for different time periods. For example 2013 will be different from 2014.
- Build a knowledge graph
	- Build a knowledge graph that links entities like county's economic profile with salmonella rates and encodes data like correlations, causal hypothesis.
	- Incorporate ontologies and domain-specific taxonomies to ensure that relationships are semantically meaningful and can be interpreted universally. (Recommendation by gpt o3 mini)
	- By fusing datasets into a semantic network, the system creates a unified representation that reveals hidden relationships. The knowledge graph paves the way for advanced querying (e.g., using SPARQL) and allows for richer insights through the exploitation of network structures and connectivity
- On providing info about data
	- Unless we use a bug model, we should do it manually because if we iterate over the datasets and provide df.head(), the datasets which are excel files would quickly consume the context window. 
- Using langgraph
	- We don't need to add messages to a node unless its the executor because the executor needs to use its own previous executions. 
	- A candidate structure for the state object
```python
	class State(TypedDict):
		query: str
		plan: Plan # create 'Plan' object before
		results: List[StateMessage]
```
- Add the system prompts and llms in the [runtime configuration](https://langchain-ai.github.io/langgraph/how-tos/graph-api/#add-runtime-configuration).
```python
from langchain.chat_models import init_chat_model
from langchain_core.runnables import RunnableConfig
from langgraph.graph import MessagesState
from langgraph.graph import END, StateGraph, START
from typing_extensions import TypedDict


class ConfigSchema(TypedDict):
    model: str


MODELS = {
    "anthropic": init_chat_model("anthropic:claude-3-5-haiku-latest"),
    "openai": init_chat_model("openai:gpt-4.1-mini"),
}


def call_model(state: MessagesState, config: RunnableConfig):
    model = config["configurable"].get("model", "anthropic")
    model = MODELS[model]
    response = model.invoke(state["messages"])
    return {"messages": [response]}


builder = StateGraph(MessagesState, config_schema=ConfigSchema)
builder.add_node("model", call_model)
builder.add_edge(START, "model")
builder.add_edge("model", END)

graph = builder.compile()

# Usage
input_message = {"role": "user", "content": "hi"}
# With no configuration, uses default (Anthropic)
response_1 = graph.invoke({"messages": [input_message]})["messages"][-1]
# Or, can set OpenAI
config = {"configurable": {"model": "openai"}}
response_2 = graph.invoke({"messages": [input_message]}, config=config)["messages"][-1]

print(response_1.response_metadata["model_name"])
print(response_2.response_metadata["model_name"])
```
- Parallelise certain computations e.g. retrieval from various sources.
	- https://youtu.be/l7lvoiCvcVU?si=TBzufVs9xmojL0Kx
- Store data context as separate vector stores, which the language models will access based on need.
- https://youtu.be/WL7V9JUy2sE?si=4baP6-DUWSgUU0Hr
- Give the Executor the docs for huggingface.

---

- Adjust the df loading function as per this chat:
  Excellent question. This gets to the core of how agent tool execution and state management work.

The short answer is: **Yes, the agent will be able to access `df` in subsequent tool calls, but this design has significant drawbacks.**

Let's break down why it works and why it's a potentially fragile pattern.

### How it Works: The `global` Keyword

1.  **Shared Execution Environment:** When an agent uses tools, those tools are typically executed within the same, single Python process. This process has its own memory and global scope.

2.  **The `global df` Statement:** This is the key. When your `load_dataset` function is called, the line `global df` tells the Python interpreter: "The `df` variable I am about to assign to is not a new variable local to this function. It is the `df` variable that exists in the global scope of this entire script/process."

3.  **State Persistence:** Because the DataFrame is assigned to a *global* variable, it persists in the Python process's memory after the `load_dataset` function has finished executing.

### The Agent's Workflow

Here is how the agent would interact with this and another potential tool (like a Python REPL):

1.  **User Request:** "Load the data from `my_data.csv` and show me the first 5 rows."

2.  **Tool Call 1: `load_dataset`**
    *   The agent determines it needs to load the data first.
    *   It calls your tool: `load_dataset(file_path='my_data.csv')`.
    *   Your function executes, creating the DataFrame and storing it in the **global** variable `df`.
    *   The function returns the *string*: `"Loaded dataset into variable `df`.\n\nPreview"`.

3.  **Agent's Reasoning:**
    *   The agent sees the successful return message. The message explicitly tells it that the data is now in a variable named `df`.

4.  **Tool Call 2: A different tool (e.g., a `python_repl` tool)**
    *   To fulfill the second part of the request ("show me the first 5 rows"), the agent now generates code that uses the `df` variable.
    *   It might call a `python_repl` tool with the code: `print(df.head())`.
    *   This new tool call executes in the **same Python process**. Since `df` exists in the global scope, this code runs successfully and returns the head of the DataFrame.

5.  **Final Answer:** The agent uses the output from the second tool call to give the user the final answer.

### The Problems with this Global State Approach

While this works for simple cases, it's considered a fragile design for several reasons:

1.  **Name Collisions:** What happens if the user wants to load a second dataset? `load_dataset('other_data.csv')` would simply **overwrite** the global `df` variable. The first dataset would be lost. There is no way to work with two dataframes at once.
2.  **Hidden State:** The fact that `df` is available is a "side effect" of the `load_dataset` tool. It's not explicitly passed between tools. This makes the agent's logic harder to trace and debug. The dependency is implicit and "magical."
3.  **Lack of Scalability:** This pattern fails in any concurrent or multi-user environment. If two users made requests at the same time in the same process, they would be fighting over the single global `df`.
4.  **Error Prone:** If another tool accidentally modifies or deletes the global `df`, it could break the entire chain of thought for the agent.

### A More Robust Alternative: Using Handles/Identifiers

A much better and more common pattern is to avoid global state and instead manage state explicitly, returning a "handle" or "ID" to the agent.

Here's how you could rewrite your tools:

```python
import pandas as pd
import uuid

# This dictionary will act as our in-memory store for dataframes.
# It is still a global variable, but it's a structured manager, not a single-use variable.
dataframe_store = {}

@tool
def load_dataset_pro(file_path, sheet_name=None):
    """
    Loads a dataset and returns a unique ID to access it later.
    Args:
        file_path (str): Path to the dataset file.
        sheet_name (str, optional): Name of the Excel sheet to load.
    Returns:
        str: A message containing the unique ID for the loaded dataframe.
    """
    if file_path.endswith('.csv'):
        try:
            df = pd.read_csv(file_path)
        except Exception as e:
            return f"PythonError: {e}"
    elif file_path.endswith('.xlsx') and sheet_name:
        try:
            df = pd.read_excel(file_path, sheet_name=sheet_name)
        except Exception as e:
            return f"PythonError: {e}"
    else:
        raise ValueError("Unsupported file format or missing sheet name for Excel file.")
    
    # Generate a unique ID for this dataframe
    df_id = f"df_{uuid.uuid4().hex[:6]}"
    
    # Store the dataframe in our manager dictionary
    dataframe_store[df_id] = df
    
    return f"Successfully loaded dataset. Use the ID `{df_id}` to refer to it in other tools."

@tool
def get_dataframe_head(df_id: str, n: int = 5):
    """
    Returns the first n rows of a loaded dataframe using its ID.
    Args:
        df_id (str): The unique ID of the dataframe, given by `load_dataset_pro`.
        n (int): The number of rows to display.
    """
    if df_id not in dataframe_store:
        return f"Error: Dataframe with ID '{df_id}' not found."
    
    return dataframe_store[df_id].head(n)
```

**How the improved workflow looks:**

1.  **Agent calls:** `load_dataset_pro('my_data.csv')`
2.  **Tool returns:** `"Successfully loaded dataset. Use the ID `df_a1b2c3` to refer to it in other tools."`
3.  **Agent now knows** it needs to use the ID `df_a1b2c3`.
4.  **Agent calls:** `get_dataframe_head(df_id='df_a1b2c3', n=5)`
5.  **Tool returns:** The actual head of the DataFrame, which the agent can then show to the user.

This approach allows the agent to manage multiple dataframes, makes dependencies explicit, and is far more robust.

---
- The datasets cannot be set by the planner in the Plan object because then, the executor doesn't have autonomy in case it wants to analyze a different dataset. We need a way to dynamically change the plan with each iteration of the executor.

# References
-   [Argonne Researchers Win Gordon Bell Special Prize for Using Language Models to Track Virus Variants - High-Performance Computing News Analysis | insideHPC](https://insidehpc.com/2022/12/argonne-researchers-win-gordon-bell-special-prize-for-using-language-models-to-track-virus-variants/)
-  [Argonne researchers win Gordon Bell Special Prize for adapting language models to track virus variants | Argonne National Laboratory](https://www.anl.gov/article/argonne-researchers-win-gordon-bell-special-prize-for-adapting-language-models-to-track-virus#:~:text=Scientists%20from%20the%20U.S.%20Department,identifying%20how%20a%20virus%20evolves.)
-  [GenSLMs: Genome-scale language models reveal SARS-CoV-2 evolutionary dynamics](https://www.biorxiv.org/content/10.1101/2022.10.10.511571v1.full.pdf)
-  [Protein Generation via Genome-scale Language Models with Bio-physical Scoring](https://dl.acm.org/doi/pdf/10.1145/3624062.3626087)
-  [2310.04607] A Comprehensive Performance Study of Large Language Models on Novel AI Accelerators](https://arxiv.org/abs/2310.04607)
-  [CSV | 🦜️🔗 LangChain](https://python.langchain.com/v0.1/docs/use_cases/sql/csv/#%EF%B8%8F-security-note-%EF%B8%8F)
-  [An integrated deep learning and stochastic optimization approach for resource management in team-based healthcare systems](https://pdf.sciencedirectassets.com/271506/1-s2.0-S0957417421X0019X/1-s2.0-S0957417421012781/main.pdf?X-Amz-Security-Token=IQoJb3JpZ2luX2VjECoaCXVzLWVhc3QtMSJIMEYCIQCx6X2aU47F%2BSVWklnSFhBY35EUrvGvIQqN%2BgWfBYPG8wIhAKUsDJVed6ETOI3Bd8w7Mx05sIsb5tbK22nFiWvlFchqKrsFCOL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQBRoMMDU5MDAzNTQ2ODY1IgyUfby4f%2BS0wAw%2FzwYqjwX2LbNDra%2FuNZsQ%2FgOIAcjm7%2Fuf5cpMsWj422RpmRjZrMPQwMd8OEbyqN5n6HF16cfiRohxCzuPVK1dnFniB0AaYo4QqbvTIELB4GQWYIOblEu6dhZXzTsC1ixmo%2BJhi0ScgE7wQq3Kl%2F09SuKwvsQR9X27QM4%2BaGlDPNjeBSZnIUx6Lf9VhfBKZg1cnj3F10tpo8G7wm0OV0SDJwuiMlgyJNJdjUxk6uy%2B9oGX7U%2FaTUGDS%2F4ZGswjO9CnPfp38Oyxis7reqIrgcTS%2FR3h%2FbVQfoBCrLVaJVYloR9oZwS6WwDIrUYgRCQ3Ei%2FbvfuhiEdeflUqR9Nw%2FikP6G%2BgFOMmlwhDDN3oleRIqqmrzsk3P9NRUSkyVYpPZlXNTMPz2mOeMPXGRqVG0BUVBnojlYbXcGSJiAtwJAZ8x68J2UU5s6Ly0hWlfPzIG66RjQI5KG%2FIR44KJJSGMECj6lifQ44VRFBoAjqoKxcGbuDry1Z%2Bq9ugGj6cvpST%2FS8YQtftpr0G91f%2FxvPAQoQhkH99%2FG15ufGBXX8WOr4UzHE4oDo7U4%2FzdBwzNu5FgXXJrrp2g5vHE8%2FoLvjsJqzrTf7sg5MiqTkbEuWDgzeOwxjAuSUm4%2Few0TAPqUDID7WepNWm0SltCnyyin9pI6li9m4saP4YZfcGeK4lgMMaIaVdBqFsQJXf8me98H4JBpCmPTeY6VRun98u5qANOJn3ln2Vcoic2urC6GMJxSXz%2BasmUDOeFUVXj2RdFjSOaq6YYOHbQI8hShgO8aYmBaA00AwIYtHDnLW4gGfPPZ2yuTF6VSYGUsOHGekT3AZ9a9y%2F4ovix%2FJuDzRr2iaZ3U3dC8TzEhVPK9hmjBG6kWrrwe3WxXZUMKG087oGOrABk8BAMxfq63KSRkVpYJ8kygITSUGvM%2Fdt3xXxNa1hvYTWWLFTl74%2FGm5RKbJs3Wv6uVL26dHCik5Q%2FEy6i0BEqv6Q1EU1uMU8j88Cp6wrXtMOq8pIz9ezvYmzoddsT7%2BTrUM%2ByALxackQ%2Ff9Qyv9C8mEF1zKYySuLbghJsy6j8gNScJC7jFAcmV%2B8SbfghzDkvRJMH%2FWw4FG5uqnYfnhAKwBRdniZmldDctqR81%2F9Rzk%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20241214T013550Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAQ3PHCVTYWK7NNPZE%2F20241214%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=9cef4d9504ae741f4591f8122cf19f3cb8e8c9017cb2f80faeb68e582ed5966e&hash=7d27286bb1934712cb484af70ca0b6589ac9284f2f34781ef87100c10007c484&host=68042c943591013ac2b2430a89b270f6af2c76d8dfd086a07176afe7c76c2c61&pii=S0957417421012781&tid=spdf-569ff6de-d7a7-4f80-9199-85cc87335702&sid=ed6405dd1336624aa18a597772c9d71985a8gxrqa&type=client&tsoh=d3d3LnNjaWVuY2VkaXJlY3QuY29t&ua=171659035307565204&rr=8f1a75a63ce86363&cc=us)
-  https://blog.langchain.dev/benchmarking-question-answering-over-csv-data/
	- [ ] https://github.com/langchain-ai/langchain-benchmarks?tab=readme-ov-file
-  https://blog.langchain.dev/benchmarking-rag-on-tables/
-  https://docs.cloud.llamaindex.ai/llamaparse/getting_started/python
	- [ ] https://github.com/run-llama/llama_cloud_services/tree/main/examples/parse
-  https://python.langchain.com/docs/how_to/tool_results_pass_to_model/
- https://blog.langchain.dev/memory-for-agents/
- https://docs.smith.langchain.com/evaluation/how_to_guides/compare_experiment_results
- https://www.youtube.com/watch?v=0i9NzY_b3pg
- https://python.langchain.com/docs/how_to/lcel_cheatsheet/#invoke-a-runnable
- https://www.kaggle.com/code/ksmooi/langgraph-question-answering-system-ai-agent
- https://youtu.be/l7lvoiCvcVU?si=TBzufVs9xmojL0Kx