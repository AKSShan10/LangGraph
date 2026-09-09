# LangGraph Learning Projects

A collection of introductory Jupyter notebooks for learning how to build workflows with LangGraph. The examples progress from checking the Python environment to a BMI calculation workflow and a simple LLM question-answering workflow.

## Projects

| Notebook | Description |
| --- | --- |
| [01_test_installation.ipynb](01_test_installation.ipynb) | Prints the active Python executable and verifies that `StateGraph` can be imported. |
| [1_bmi_workflow.ipynb](1_bmi_workflow.ipynb) | Calculates BMI and assigns a category using two graph nodes and a shared typed state. |
| [2_simple_llm_workflow.ipynb](2_simple_llm_workflow.ipynb) | Sends a question to an OpenAI chat model and stores its answer in graph state. |

## Requirements

- Python 3.11 (the notebooks were saved with Python 3.11 kernels)
- Jupyter Notebook, JupyterLab, or an editor with Jupyter support
- An OpenAI API key with available API credits for the LLM notebook

The installation and BMI notebooks do not require an OpenAI API key. Rendering the BMI graph with `draw_mermaid_png()` may require an internet connection.

Dependencies are pinned in [requirements.txt](requirements.txt), including LangGraph, LangChain, `langchain-openai`, `python-dotenv`, and Jupyter.

## Setup

Run the following commands in PowerShell from the repository directory:

```powershell
cd D:\BJIT\LangGraph
py -3.11 -m venv myenv
.\myenv\Scripts\python.exe -m pip install -r requirements.txt
.\myenv\Scripts\python.exe -m ipykernel install --user --name myenv_LangGraph --display-name "Python 3.11 (myenv_LangGraph)"
.\myenv\Scripts\python.exe -m jupyter lab
```

Adjust the repository path if you cloned it elsewhere. If `myenv` already exists, skip the virtual environment creation command. These commands use the environment's Python directly, so activating it is optional.

Open a notebook, select **Python 3.11 (myenv_LangGraph)** as its kernel, and run its cells from top to bottom. Start with the installation notebook to confirm the selected interpreter.

### Configure the LLM notebook

Create a `.env` file in the repository root with your own key:

```dotenv
OPENAI_API_KEY=your_openai_api_key_here
```

The notebook calls `load_dotenv()` before creating `ChatOpenAI()`. It does not explicitly select a model; it uses the library's default. To choose a model available to your API account, pass `model="your-model-id"` when constructing `ChatOpenAI`.

The `.env` file is ignored by Git. Keep actual API keys out of notebooks, saved outputs, and commits.

## How the Workflows Work

Both workflow examples follow the same steps:

1. Define the shared state with `TypedDict`.
2. Write node functions that read and update state.
3. Create a `StateGraph` and register the nodes.
4. Connect the nodes with edges from `START` to `END`.
5. Compile the graph and execute it with `workflow.invoke(initial_state)`.

### BMI workflow

The extended workflow is:

```text
START -> calculate_bmi -> label_bmi -> END
```

`BMIState` holds `weight_kg`, `height_m`, `bmi`, and `category`. The first node calculates `weight_kg / height_m ** 2` and rounds it to two decimal places. The second node assigns a category using the calculated value.

Example input:

```python
initial_state = {"weight_kg": 80, "height_m": 1.73}
final_state = workflow.invoke(initial_state)
print(final_state)
```

Saved output:

```python
{'weight_kg': 80, 'height_m': 1.73, 'bmi': 26.73, 'category': 'Overweight'}
```

The notebook currently includes the edges for both the simple and extended versions in the same graph-building cell, including a direct `calculate_bmi -> END` edge. For a clean extended example, keep only the three edges shown in the path above. Input validation is not implemented, so use positive height and weight values.

### LLM question-answering workflow

```text
START -> llm_qa -> END
```

`LLMState` contains `question` and `answer`. The `llm_qa` node creates a prompt from the question, calls `model.invoke(prompt)`, and stores the response content in `answer`.

The notebook's example question is:

```python
initial_state = {"question": "How far is moon from the earth?"}
final_state = workflow.invoke(initial_state)
```

After a successful invocation, display the answer in another cell:

```python
print(final_state["answer"])
```

## Troubleshooting

| Issue | What to check |
| --- | --- |
| `ModuleNotFoundError` | Install the requirements and select the kernel registered from `myenv`. |
| Wrong Python interpreter | Run the installation notebook and check that `sys.executable` points to `myenv\Scripts\python.exe`. |
| Missing API key | Ensure `.env` is in the repository root and rerun the `load_dotenv()` cell from that directory. |
| OpenAI `429` with `insufficient_quota` or `credit_balance_exhausted` | The saved LLM execution reports exhausted API credits. Check the API account's billing and available credits before retrying. |
| Graph image fails to render | Check internet access or skip the visualization cell; it is separate from workflow execution. |

## Repository Structure

```text
LangGraph/
|-- 01_test_installation.ipynb
|-- 1_bmi_workflow.ipynb
|-- 2_simple_llm_workflow.ipynb
|-- requirements.txt
|-- README.md
|-- .gitignore
|-- .env                     # Local API credentials; ignored by Git
`-- myenv/                   # Local Python environment; ignored by Git
```

These notebooks are learning examples. They currently demonstrate linear workflows without conditional routing, tools, persistence, or conversation memory.
