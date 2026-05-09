# Python Installation and Setup

## Overview

- Python is a high-level, interpreted programming language known for its readability and massive ecosystem.
- It is the dominant language for Data Science, Machine Learning, and Artificial Intelligence, but also highly capable for backend web development.
- Python code runs on an interpreter (like CPython) which translates it to bytecode.

## Core Use Cases
**Web Backends & APIs**: Very popular for building RESTful and GraphQL APIs using frameworks like FastAPI, Django, and Flask.
**AI & Machine Learning**: The undisputed leader, with libraries like TensorFlow, PyTorch, and Scikit-Learn.
**Data Analysis**: Tools like Pandas and NumPy make it perfect for crunching large datasets.
**Scripting & Automation**: Excellent for writing system scripts, web scrapers (BeautifulSoup, Scrapy), and background cron jobs.

## Key Benefits
**Readability**: Clean, indentation-based syntax that feels like reading plain English.
**Massive Ecosystem**: The Python Package Index (PyPI) has over 400,000 packages for almost any task imaginable.
**Versatility**: You can write a small 10-line script or build a massive enterprise web application.

### When NOT to use it
Python is generally **not recommended for mobile app development** or **high-performance systems** where microsecond latency is required (like high-frequency trading), as its dynamic typing and Global Interpreter Lock (GIL) make it slower than compiled languages like Go, Rust, or C++.

## Installation
Unlike Node.js, Python comes pre-installed on many systems (macOS, Linux), but it's often an older "system version". You should **never** use the system Python for your projects.

The best way to install and manage multiple Python versions is using `pyenv`.
- [Installation for Mac/Linux (pyenv)](https://github.com/pyenv/pyenv)
- [Installation for Windows (pyenv-win)](https://github.com/pyenv-win/pyenv-win)

## Virtual Environments
In Python, installing packages globally is a huge anti-pattern. Instead, you create a "Virtual Environment" for every project. This isolates project dependencies from each other.

To create a basic virtual environment (using built-in `venv`):
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

## Package Managers
The default package manager is `pip`, but modern Python development relies on tools that handle both dependency management and virtual environments seamlessly.

The two prominent modern tools are `Poetry` and `uv` (by Astral).

### Comparison between `pip`, `Poetry`, and `uv`

| Feature | `pip` + `venv` | `Poetry` | `uv` |
| --- | --- | --- | --- |
| **Type** | Basic Package Manager | Full Dependency & Build Manager | Ultra-fast Package & Env Manager |
| **Speed** | Moderate | Slow/Moderate | Blazing Fast (Written in Rust) |
| **Lockfile Support**| Manual (`requirements.txt`) | Built-in (`poetry.lock`) | Built-in (`uv.lock`) |
| **Virtual Env Setup**| Manual | Automatic | Automatic |
| **Best For** | Docker images, quick scripts | Enterprise apps, strict dependency resolution | Modern projects, massive monorepos, speed |

#### Quick Take
| Use Case | pip | Poetry | uv |
| --- | :---: | :---: | :---: |
| Single file script | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| Enterprise backend | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Lightning-fast CI/CD | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Maximum standard compatibility | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |

#### Frequent Commands
| Action | pip | Poetry | uv |
| --- | --- | --- | --- |
| Create project | `mkdir app && cd app && python -m venv venv` | `poetry new app` | `uv init` |
| Install all deps | `pip install -r requirements.txt` | `poetry install` | `uv sync` |
| Add package | `pip install fastapi` | `poetry add fastapi` | `uv add fastapi` |
| Run script | `python main.py` | `poetry run python main.py` | `uv run main.py` |

#### Recommendation
- Choose **`pip` + `venv`** if you are writing a simple Dockerfile and want zero extra tooling.
- Choose **`Poetry`** if you want the most mature, heavily adopted dependency manager for complex enterprise projects.
- Choose **`uv`** if you want the absolute fastest, most modern toolchain (highly recommended for new projects).
