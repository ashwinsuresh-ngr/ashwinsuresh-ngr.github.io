Title: Common Python Errors in AI Projects
Date: 2026-03-01
Category: GenAI
Tags: GenAI, Python, debugging, developers, best-practices
Slug: common-python-errors-in-ai-projects

AI code breaks in predictable ways. Here are the traps that waste hours, with the fixes that save them.

## 1. Mutable Default Arguments

```python
# WRONG: All calls share the same list
def predict(inputs, cache=[]):
    cache.append(inputs)
    return model(inputs)

# RIGHT: Fresh list every call
def predict(inputs, cache=None):
    if cache is None:
        cache = []
    cache.append(inputs)
    return model(inputs)
```

**Why it hurts:** In AI pipelines, mutable defaults accumulate training batches across function calls, silently poisoning your data.

## 2. Shallow Copies of Arrays

```python
# WRONG: Points to same memory
import numpy as np
a = np.array([1, 2, 3])
b = a  # Same object
b[0] = 999  # Mutates a too

# RIGHT: Independent copy
b = a.copy()

# Or for PyTorch:
b = tensor.clone().detach()
```

**Why it hurts:** Data augmentation and preprocessing mutate "copies" that are actually views, corrupting your training set.

## 3. Modifying Data While Iterating

```python
# WRONG
for row in df.index:
    if df.loc[row, 'score'] < 0.5:
        df.drop(row, inplace=True)  # RuntimeError or skipped rows

# RIGHT
df = df[df['score'] >= 0.5]
```

**Why it hurts:** Data cleaning loops silently skip elements or crash when the container size changes mid-iteration.

## 4. Forgetting `.item()` on Tensors

```python
# WRONG: Returns a tensor, not a Python number
loss = criterion(pred, target)
if loss < 0.1:  # Works by accident, but dangerous
    print("Converged")

# RIGHT
if loss.item() < 0.1:
    print("Converged")
```

**Why it hurts:** Comparing tensors creates computation graph dependencies. In PyTorch, this leaks memory and breaks gradient flows.

## 5. Shape Mismatches

```python
# WRONG
import torch
a = torch.randn(32, 10)
b = torch.randn(10, 32)
c = a @ b  # RuntimeError: mat1 and mat2 shapes incompatible

# RIGHT
c = a @ b.T  # Or ensure inner dims match
```

**Why it hurts:** AI is tensor algebra. A shape error in the middle of a forward pass gives a stack trace 20 layers deep into library code.

## 6. Not Setting Random Seeds

```python
# WRONG: Results change every run
model = Transformer()
train(model)

# RIGHT
import random, numpy as np, torch

def set_seed(seed=42):
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)
```

**Why it hurts:** You can't debug what you can't reproduce. Different embeddings, different splits, different metrics every run.

## 7. Memory Leaks with DataFrames

```python
# WRONG: Chained indexing creates copies in loops
for i in range(len(df)):
    val = df[df['id'] == i]['value']  # Hidden copy every iteration

# RIGHT: Vectorize or use .loc/.iloc explicitly
vals = df.loc[df['id'].isin(range(n)), 'value']
```

**Why it hurts:** Large datasets in loops with chained indexing duplicate memory until your kernel dies.

## 8. Pickling Lambda Functions

```python
# WRONG: Can't serialize lambdas
import pickle
preprocess = lambda x: x.lower()
pickle.dump(preprocess, open("prep.pkl", "wb"))  # PicklingError

# RIGHT: Use top-level named functions
def preprocess(x):
    return x.lower()

pickle.dump(preprocess, open("prep.pkl", "wb"))
```

**Why it hurts:** Saving preprocessing pipelines or model wrappers with lambdas breaks model deployment and sharing.

## 9. Ignoring NaN Propagation

```python
# WRONG: NaN silently kills your metric
import numpy as np
preds = np.array([0.9, np.nan, 0.1])
accuracy = np.mean(preds == labels)  # NaN result, no warning

# RIGHT: Explicit check
if np.isnan(preds).any():
    raise ValueError("NaN detected in predictions")
```

**Why it hurts:** A single NaN in a batch gradient or prediction poisons the entire training run, and the loss just stops decreasing with no clear error.

## 10. Threading CPU-Bound ML Work

```python
# WRONG: GIL prevents parallelism
import threading
for batch in batches:
    t = threading.Thread(target=preprocess, args=(batch,))
    t.start()

# RIGHT: Use multiprocessing
from multiprocessing import Pool
with Pool(4) as p:
    p.map(preprocess, batches)
```

**Why it hurts:** Python's Global Interpreter Lock means threads don't parallelize NumPy/Pandas CPU work. You get overhead, not speedup.

## Bottom Line

These aren't exotic bugs — they're the ones that cost AI teams afternoons. Mutable defaults, tensor shapes, random seeds, and NaNs. Check them before you debug anything else.
