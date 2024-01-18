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
import matplotlib.pyplot as plt
from pathlib import Path
import math 
# data_dir = Path('.') / 'data'
```
## Introduction


**G(N,p) model**: each pair of N labeled nodes is connected with probability p (i.e., every possible edge
occurs independently with prob p)

```python
f = plt.figure(figsize = (10, 4))
axes = f.subplots(nrows = 2, ncols = 3) # one figure with 2x3 axes

n = 10
g1 = nx.gnp_random_graph(n, p = 0)
g1.p = 0
g1.name = f"g({n},p={g1.p})"

g2 = nx.gnp_random_graph(n, p = 0.5)
g2.p = 0.5
g2.name = f"g({n},p={g2.p})"

g3 = nx.gnp_random_graph(n, p = 1)
g3.p = 1.0
g3.name = f"g({n},p={g3.p})"

n = 100
g4 = nx.gnp_random_graph(n, p = 0)
g4.p = 0
g4.name = f"g({n},p={g4.p})"

g5 = nx.gnp_random_graph(n, p = 0.5)
g5.p = 0.5
g5.name = f"g({n},p={g5.p})"

g6 = nx.gnp_random_graph(n, p = 1)
g6.p = 1.0
g6.name = f"g({n},p={g6.p})"

nets = [g1, g2, g3, g4, g5, g6]

for k,n in enumerate(nets):
    nx.draw_circular(n, ax = axes[k//3, k % 3])

plt.show()
```

```python
def print_network_characteristics(g):
    n = g.number_of_nodes()
    print(f"### {g}")
    print("Number of nodes: ", n)
    expected_nedges = g.p * g.number_of_nodes() * (n - 1) / 2
    print("Number of edges: ", g.number_of_edges(), " - Expected number of edges (p * N * (N-1)/2): ",  expected_nedges)
    print("Min degree = ", min([d for n, d in g.degree()]), " -- Max degree = ", max([d for n, d in g.degree()]) , )
    # print("Degree assortativity: ", nx.degree_assortativity_coefficient(g))
    expected_avg_degree = g.p * (n - 1)
    print("Average degree: ", sum([d for n, d in g.degree()]) / n, 
          " -- Expected: <k> = (p * (N-1)): ", expected_avg_degree, " = 2<L> / N = ", 2 * expected_nedges / n)
    print("Variance in expected degree (p*(1-p)*(N-1)): ", g.p * (1 - g.p) * (n - 1))
    print("Average number of links: ", 2 * g.number_of_edges() / n)
    print("Average clustering coefficient: ", nx.average_clustering(g), " -- Expected: <C> = <k>/N = ", expected_avg_degree / n)
    if nx.is_connected(g):
        print("Diameter: ", nx.diameter(g))
        print("Average shortest path length: ", nx.average_shortest_path_length(g))
    else:
        print("Diameter: not connected")
        print("Average shortest path length: not connected")
    print("")
    
for n in nets: print_network_characteristics(n)
```

**G(n,m) model**: n labeled nodes are connected with m randomly places links (i.e., a graph is chosen
uniformly at random from the set of all graphs of n nodes and m links)


```python
g1 = nx.gnm_random_graph(n = 10, m = 3)
nx.draw_spring(g1)
```

## Regimes of a random network

```python
N = 50

def analyse_for_giant_components(g):
    print("### ", g.name)
    print(f"Number of nodes: {g.number_of_nodes()}, number of edges: {g.number_of_edges()}")
    print(f"Number of connected components: {nx.number_connected_components(g)}")
    connected_components = sorted(nx.connected_components(g), key = len, reverse = True)
    print(f"Size of largest connected component: {len(connected_components[0])}")
    print(f"Sizes of connected components: {[len(c) for c in connected_components]}")
    if len(connected_components) > 1:
        print(f"2nd and 3rd largest connected components: ", connected_components[1:3], "\n\tEdges: ", 
          g.subgraph(connected_components[1]).edges(), g.subgraph(connected_components[2]).edges(),
          "\n\tAre these both trees? ", nx.is_tree(g.subgraph(connected_components[1])) and nx.is_tree(g.subgraph(connected_components[2])))
    #print(f"Does the graph contains cliques?", len(list(nx.simple_cycles(g))) > 0)
    #print(f"Does the largest connected component contains loops? ", 
    #      len(list(nx.simple_cycles(g.subgraph(connected_components[0])))) > 0)
    print("")

# subcritical regime for <k> < 1 (p < 1/N)
p = 1 / (2 * N)
rn1 = nx.gnp_random_graph(N, p)
rn1.name = f"Random network with N = {N}, p = {p} (subcritical regime)"

# critical point <k> = 1
p = 1 / N
rn2 = nx.gnp_random_graph(N, p)
rn2.name = f"Random network with N = {N}, p = {p} (critical point)"

# supercritical regime for <k> > 1 (p > 1/N)
p = 1.2 / N
rn3 = nx.gnp_random_graph(N, p)
rn3.name = f"Random network with N = {N}, p = {p} (supercritical regime)"

# connected regime for <k> > ln(N) (p > ln(N)/N)
p = 1.1 * math.log(N, math.e) / N
rn4 = nx.gnp_random_graph(N, p)
rn4.name = f"Random network with N = {N}, p = {p} (connected regime)"

nets = [rn1, rn2, rn3, rn4]

for n in nets: analyse_for_giant_components(n)

```

```python
f = plt.figure(figsize = (20, 8))
axes = f.subplots(nrows = 1, ncols = 4)
for n in nets: nx.draw_circular(n, ax = axes[nets.index(n)])
```
