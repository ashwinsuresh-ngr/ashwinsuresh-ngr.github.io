Title: Conda Environments for AI Development
Date: 2026-02-20
Category: GenAI
Tags: GenAI, Python, conda, virtual-environments, developers
Slug: conda-environments-for-ai-development

The previous post covered venv as the standard, built-in way to isolate Python dependencies. But anyone who's spent time in AI or data science circles has run into its limits fast — usually the moment a project needs a specific CUDA version, a non-Python library, or a complex scientific computing stack that pip alone struggles to manage cleanly. That's where conda comes in. Here's what it does differently, and when it's actually worth reaching for.

## What Conda Actually Is

Conda is a package and environment manager — but unlike pip and venv, which only manage Python packages, conda manages packages and dependencies across any language, including compiled binaries, system libraries, and non-Python tools. It emerged from the data science and scientific computing world specifically because that ecosystem depends heavily on complex, non-Python components — linear algebra libraries, GPU drivers, compiled numerical code — that pip was never really designed to handle.

This distinction matters a lot in AI development: deep learning frameworks like PyTorch and TensorFlow don't just depend on Python packages, they depend on specific versions of CUDA (NVIDIA's GPU computing toolkit), cuDNN, and other low-level system libraries. Conda can manage all of that within an isolated environment; venv fundamentally can't, because it only ever touches the Python layer.

## Creating and Using a Conda Environment

The basic workflow parallels venv, with a few key differences:

```bash
conda create --name my-ai-project python=3.11
```

This creates a new, named environment (rather than a folder you cd into) with a specific Python version — conda can install and manage the Python interpreter itself, not just packages within an existing one.

```bash
conda activate my-ai-project
```

Once activated, installing packages works through conda's own package manager:

```bash
conda install pytorch torchvision -c pytorch
```

The `-c pytorch` here specifies a channel — a package repository conda pulls from. Many specialized AI and scientific packages are distributed through specific channels (like `pytorch`, `conda-forge`, or `nvidia`) rather than conda's default channel, and getting the right combination of packages and channels correctly matched is one of conda's more involved but genuinely valuable capabilities.

Deactivating works the same as venv:

```bash
conda deactivate
```

## Why Conda Handles GPU Dependencies Better

This is conda's single biggest practical advantage for AI development. Installing a GPU-accelerated version of PyTorch with pip requires you to already have the correct CUDA toolkit version installed system-wide, matched precisely to what that PyTorch build expects — a notoriously fiddly, error-prone process, especially when different projects on the same machine want different CUDA versions.

Conda can install CUDA itself, scoped to the environment:

```bash
conda create --name gpu-project python=3.11
conda activate gpu-project
conda install pytorch torchvision pytorch-cuda=12.1 -c pytorch -c nvidia
```

This pulls in a CUDA toolkit version inside the environment itself, isolated from whatever else might be installed on the system — meaning two projects on the same machine can genuinely use different CUDA versions without conflict, something that's very difficult to achieve cleanly with pip and venv alone.

## Conda vs. pip and venv: The Practical Differences

| Aspect | venv + pip | Conda |
|--------|-----------|-------|
| Scope | Python packages only | Python packages, non-Python libraries, system dependencies |
| Python interpreter | Uses whatever's already installed | Can install and manage Python versions itself |
| GPU/CUDA handling | Relies on system-wide installation | Can install and isolate CUDA within the environment |
| Package sources | PyPI only | Multiple channels (conda-forge, pytorch, nvidia, etc.) |
| Built into Python | Yes (venv is standard library) | No, separate installation required (Miniconda or Anaconda) |
| Typical use case | General Python projects, lightweight AI scripting | Heavier ML/AI work, GPU dependencies, scientific computing |

Neither tool is strictly better — they're suited to somewhat different situations, and it's common to see conda used specifically for the deep learning/GPU layer of a project while pip handles everything else within that same conda environment.

## Miniconda vs. Anaconda: Which to Install

Conda itself doesn't come with Python — it needs to be installed through one of two distributions:

- **Anaconda** — a large, batteries-included distribution that bundles conda with hundreds of pre-installed data science packages (NumPy, Pandas, Jupyter, and more) out of the box. Convenient for getting started quickly, but heavy — often several gigabytes — and includes far more than most individual AI projects actually need.
- **Miniconda** — a minimal installer that includes just conda and Python itself, letting you install only the packages your specific project actually needs. Generally the better starting point for AI development specifically, since it avoids bloating every environment with packages you didn't ask for.

For most people doing focused generative AI work rather than broad general data science, Miniconda plus deliberate `conda install` and `pip install` calls per project tends to be the more practical, lighter-weight choice.

## Mixing Conda and Pip: A Common, Reasonable Pattern

It's entirely normal — and often the practical reality — to use both within the same conda environment: conda for the heavier, GPU-dependent, or non-Python packages, and pip for everything else, especially newer or more specialized AI libraries (like `anthropic` or `openai`) that are typically distributed via PyPI rather than conda channels:

```bash
conda create --name genai-project python=3.11
conda activate genai-project
conda install pytorch pytorch-cuda=12.1 -c pytorch -c nvidia
pip install anthropic openai pydantic python-dotenv
```

This pattern is common enough to be considered standard practice in AI development — conda handles the tricky GPU/compiled-dependency layer, and pip handles the fast-moving, pure-Python AI SDK layer on top. The one thing worth being careful about: install conda packages first, then pip packages, since installing in the other order can sometimes cause conda's dependency resolver to lose track of what pip already put in place.

## Exporting and Recreating Conda Environments

Just as venv projects use `requirements.txt`, conda environments can be exported to a reproducible specification file:

```bash
conda env export > environment.yml
```

This captures not just Python packages, but the full environment — Python version, conda packages, channels, and (if included) pip-installed packages too:

```yaml
name: genai-project
channels:
  - pytorch
  - nvidia
  - conda-forge
dependencies:
  - python=3.11
  - pytorch=2.4.0
  - pytorch-cuda=12.1
  - pip:
      - anthropic==0.39.0
      - pydantic==2.9.2
```

Recreating it elsewhere — a teammate's machine, a cloud training instance, a fresh setup — is a single command:

```bash
conda env create -f environment.yml
```

This connects directly to the reproducibility concerns covered in the previous post: given how much a project's actual behavior can depend on exact CUDA and framework versions in AI work specifically, this kind of complete, portable specification matters more here than in most general Python development.

## When Conda Is Genuinely Worth the Extra Setup

Conda comes with real overhead — a separate installation, its own mental model, occasionally slower dependency resolution than pip — so it's worth being deliberate about when that trade-off pays off:

- You're working with GPU-accelerated deep learning frameworks and need reliable, isolated CUDA version management
- Your project depends on non-Python scientific computing libraries that need careful version coordination
- You're doing model training or fine-tuning locally, rather than purely calling hosted AI APIs
- You're collaborating across different machines or operating systems where exact binary compatibility matters
- You're working within an existing data science team or codebase that's already standardized on conda

## When Plain venv Is Still the Better Choice

For a large share of generative AI application development — the kind covered throughout most of this series, calling hosted APIs like Anthropic's or OpenAI's rather than training models locally — conda's extra capabilities often go entirely unused:

- You're primarily calling hosted LLM APIs rather than running models locally, so there's no GPU/CUDA dependency to manage at all
- Your dependencies are pure Python — SDKs, Pydantic, request-handling libraries — exactly the kind pip handles natively and well
- You want the lightest possible setup, without a separate conda installation to maintain
- You're building and deploying to standard cloud or server environments that expect a conventional pip-based Python setup

In that very common case — an application built around API calls to a hosted model rather than local training or inference — plain venv and `requirements.txt`, as covered in the previous post, is usually simpler and entirely sufficient. Conda earns its place specifically when local, GPU-dependent computation enters the picture.

## The Bottom Line

Conda solves a problem venv fundamentally can't: managing complex, non-Python dependencies — GPU drivers, CUDA toolkits, compiled scientific libraries — alongside Python packages, all isolated per project. That makes it a genuinely valuable tool for AI development that involves local model training, fine-tuning, or GPU-accelerated inference, where getting framework and CUDA versions to line up correctly is a real, recurring source of pain. But for the large share of generative AI application work that's really about calling hosted APIs and building software around their responses, that extra capability often goes unused — and plain venv, covered in the previous post, remains the simpler, entirely sufficient choice. Knowing which situation you're actually in is most of what matters here: reach for conda when GPUs and compiled dependencies are genuinely part of the picture, and stick with venv when they're not.
