Title: Managing AI Dependencies & API Keys: A Developer's Survival Guide
Date: 2026-02-22
Category: GenAI
Tags: GenAI, Python, dependencies, security, API-keys, developers
Slug: managing-ai-dependencies-and-api-keys

AI projects collapse for two predictable reasons: dependencies that won't install, and API keys that leak on GitHub. Here's how to handle both without the drama.

## Lock Down Your Dependencies

AI's dependency stack is a house of cards. PyTorch wants CUDA 11.8. Transformers needs a specific tokenizers version. FlashAttention won't compile on Windows. A sloppy `requirements.txt` guarantees your project works on exactly one machine — yours.

### What Actually Works

**Pin everything.** Not just direct dependencies, but the transitive ones too.

```txt
# requirements.txt
torch==2.1.0+cu118
transformers==4.35.0
numpy==1.24.3
--index-url https://download.pytorch.org/whl/cu118
```

Generate the full lock file with `pip freeze > requirements.txt` after testing in a clean environment. Or better, use `pip-tools`:

```bash
# requirements.in
torch==2.1.0+cu118
transformers>=4.30.0

# Compile to a lock file
pip-compile requirements.in -o requirements.txt
```

### The Golden Rules

- **One env per project.** Never install PyTorch globally.
- **Encode CUDA variants.** `torch==2.1.0+cu118` is not optional — it's a different package than the CPU build.
- **Separate dev and prod.** Don't ship Jupyter or pytest to production containers.
- **Containerize early.** A `Dockerfile` + pinned `requirements.txt` is the only reproducibility guarantee that survives across machines.

## Keep API Keys Out of Your Code

Hardcoding `api_key="sk-abc123..."` in a script takes five seconds. Rotating that key after it hits GitHub history takes five hours. Environment variables are cheaper.

### The Pattern

Create a `.env` file (and **immediately** add it to `.gitignore`):

```bash
OPENAI_API_KEY=sk-...
HUGGINGFACE_TOKEN=hf-...
PINECONE_API_KEY=pc-...
```

Load it in Python:

```python
from dotenv import load_dotenv
import os

load_dotenv()
client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
```

### Fail Fast, Not Silent

Don't let a missing key crash your app mid-inference. Validate at startup:

```python
from dataclasses import dataclass

@dataclass
class Config:
    openai_key: str
    hf_token: str

    @classmethod
    def load(cls):
        return cls(
            openai_key=os.environ["OPENAI_API_KEY"],  # KeyError = instant death
            hf_token=os.environ["HUGGINGFACE_TOKEN"],
        )
```

### Production Escalation

`.env` files are for local development only. In production, use real secret management:

- **AWS Secrets Manager / GCP Secret Manager / Azure Key Vault**
- **Kubernetes Secrets** (never ConfigMaps)
- **GitHub Actions encrypted secrets** for CI/CD

Pass secrets to Docker at runtime, never bake them into images:

```bash
docker run -e OPENAI_API_KEY="$OPENAI_API_KEY" my-app
```

## The Checklist

| Task | Status |
|------|--------|
| `.env` added to `.gitignore` | ☐ |
| `requirements.txt` pinned with exact versions | ☐ |
| CUDA variant specified for PyTorch/TensorFlow | ☐ |
| Config validation fails fast on missing keys | ☐ |
| Dev/prod keys are separate | ☐ |
| Production uses a secret manager, not `.env` | ☐ |
| No API keys in notebooks or source code | ☐ |

## Bottom Line

Reproducibility and security aren't afterthoughts in AI — they're the foundation. Pin your dependencies like your job depends on it. Treat API keys like credit card numbers. Get these two things right, and you've eliminated 80% of the "works on my machine" and "who spent $5,000 on OpenAI?" disasters before they start.
