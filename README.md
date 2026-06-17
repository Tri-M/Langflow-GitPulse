# GitPulse : Multi-Agent Architectural Code Auditor & Compliance Engine

| Portfolio Matrix | Project Specifications |
| :--- | :--- |
| **Engine Architecture** | Async Multi-Agent Visual State Graph Orchestration |
| **Production Focus** | Core Context Token Trimming, Structured Data Output Handoffs |
| **Telemetry & Auditing** | Real-Time LLM Latency tracking, Token Costs, and Node-Step Execution Logs |
| **Infrastructure Overhead** | \$0.00 (Built entirely using Free Developer Tiers) |

---


Most portfolio projects are basic AI wrappers that send flat text to a single prompt box. **GitPulse** mirrors actual enterprise-grade code engineering workflows. It implements a decoupled, visual multi-agent layout to extract structural software source files via an abstract syntax tree, runs automated vulnerability scans, and outputs a strict, deterministic JSON object payload. This payload is passed across an agent boundary to trigger a documentation generation engine without a single structural parsing failure.

---

##  System Architecture & Data Flow

GitPulse splits core analysis from layout formatting by isolating processing responsibilities into specialized agent execution layers.

1. **Ingestion & Compilation Layer:** A custom Python Abstract Syntax Tree (`ast`) node opens the local codebase workspace and isolates structural metadata metrics (imports, classes, methods, and functions), removing unnecessary source filler tokens to keep context focus tight.
2. **Agent A (The Security Auditor):** An LLM execution node powered by `gemini-2.5-flash` reviews the clean AST structure and isolates architectural vulnerability leaks, hardcoded credentials, or systemic data mutation patterns. It formats its conclusions directly into a deterministic JSON object template.
3. **Agent B (The Technical Writer):** A specialized documentation model consumes the JSON payload, translates the compliance findings, and formats them into a scannable, developer-ready Markdown audit dashboard report.

---

## 💻 Source Code: Custom Ingestion Component

 It maps local Python code directly into clean text string formats readable by prompt modules.

```python
import ast
from langflow.custom import Component
from langflow.io import MessageTextInput, Output

class GitPulseCodeParser(Component):
    display_name = "GitPulse AST Parser"
    description = "Parses a local Python file into structured chunks (Classes & Functions) using AST and outputs a text string."
    icon = "code"
    name = "gitpulse_parser"

    inputs = [
        MessageTextInput(
            name="file_path", 
            display_name="Target File Path", 
            value="vulnerable_sample.py",
            info="The local path of the Python file you want to parse."
        ),
    ]

    outputs = [
        Output(display_name="Structured Code Data", name="output_data", method="parse_code"),
    ]

    def parse_code(self) -> str:
        target_path = self.file_path
        
        try:
            with open(target_path, "r", encoding="utf-8") as f:
                source_code = f.read()

            tree = ast.parse(source_code)
            structured_chunks = {
                "classes": [],
                "functions": [],
                "imports": []
            }

            for node in ast.iter_child_nodes(tree):
                if isinstance(node, (ast.Import, ast.ImportFrom)):
                    structured_chunks["imports"].append(ast.unparse(node))
                    
                elif isinstance(node, ast.ClassDef):
                    class_info = {
                        "name": node.name,
                        "docstring": ast.get_docstring(node),
                        "methods": [child.name for child in node.body if isinstance(child, ast.FunctionDef)]
                    }
                    structured_chunks["classes"].append(class_info)
                    
                elif isinstance(node, ast.FunctionDef):
                    func_info = {
                        "name": node.name,
                        "docstring": ast.get_docstring(node),
                        "args": [arg.arg for arg in node.args.args]
                    }
                    structured_chunks["functions"].append(func_info)

            formatted_text = (
                f"=== IMPORTS ===\n{structured_chunks['imports']}\n\n"
                f"=== CLASSES ===\n{structured_chunks['classes']}\n\n"
                f"=== FUNCTIONS ===\n{structured_chunks['functions']}"
            )
            
            self.status = f"Successfully parsed {target_path}."
            return formatted_text

        except FileNotFoundError:
            error_msg = f"Error: '{target_path}' not found. Verify it exists in your execution directory."
            self.status = error_msg
            return error_msg
        except Exception as e:
            error_msg = f"Parsing Error: {str(e)}"
            self.status = error_msg
            return error_msg
