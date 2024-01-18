---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.14.1
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---

# Random networks

```python
import networkx as nx
from networkx.algorithms import bipartite
import matplotlib.pyplot as plt
from pathlib import Path
data_dir = Path('.') / 'data'


G = nx.gnm_random_graph(n = 10, m = 20)
nx.draw_spring(G)
```

