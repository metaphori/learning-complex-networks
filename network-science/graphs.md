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

# Graphs

Basic imports

```python
import matplotlib.pyplot as plt
import numpy as np
import matplotlib as mpl
import random
import networkx as nx
from collections import deque

def char_range(start='a', end='z'):
    return map(chr,list(range(ord(start),ord(end)+1)))
```


## Basic graph representation and plotting

```python
g = nx.DiGraph() # Graph
default_node_attributes = {'visited': False, 'parent': None, 'label': '', 'color': '#ababab'}
g.add_nodes_from(range(0,10), **default_node_attributes)
g.add_edges_from([(1,7), (7,3), (3,6), (6,7), (8,0)])

def plot_basic_graph(G, pos = None, layout = nx.random_layout, seed = None):
    # Define positions for nodes
    if pos is None:
        pos = layout(G, seed = seed)
    # Create matplotlib figure
    fig, ax = plt.subplots(1, 1, figsize=(8,5))
    # Draw nodes and edges
    node_colors = [node[1]['color'] for node in G.nodes(data=True)]
    nx.draw(G, pos, ax = ax, with_labels=True, node_color=node_colors, edgecolors='#000', 
            linewidths=1, font_size=16, font_weight='bold', node_size=600)
    # Draw labels nearby nodes
    for n in G.nodes:
        x, y = pos[n]
        ax.text(x-0.2, y-0.1, f"{G.nodes[n]['label']}",ha='right', va='bottom', color='#00f', fontsize=14)
    # Draw edge labels
    edge_labels = nx.get_edge_attributes(G, 'weight')
    nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels)
    # Show figure
    fig.set_facecolor("#ededed")
    plt.show()

# pos = nx.random_layout(G, seed=0)
pos = {0: (4,0), 1: (0,0), 2: (0,1), 3: (1,0.5), 4: (1,1), 5: (2,1), 6: (2,0.25), 7: (1,0), 8: (4,0.5), 9: (4,1)};

plot_basic_graph(g, pos)
```

```python
COLOR_SCHEDULED = '#ddd'
COLOR_VISITED = '#fc4'
COLOR_UNSEEN = '#fff'

fig_index = 0

def bfv(g, root, f):
    q = deque()
    i = 0
    g.nodes[root]['visited'] = True
    g.nodes[root]['visit_order'] = i
    g.nodes[root]['root'] = COLOR_SCHEDULED
    i += 1
    q.append(root)
    while len(q) > 0:
        v = q.popleft()
        g.nodes[v]['color'] = COLOR_VISITED
        adjacent_nodes = list(g.neighbors(v))
        adjacent_nodes.sort()
        for nbr in adjacent_nodes:
            if not g.nodes[nbr]['visited']:
                g.nodes[nbr]['color'] = COLOR_SCHEDULED
                g.nodes[nbr]['visited'] = True
                g.nodes[nbr]['visit_order'] = i
                i += 1
                g.nodes[nbr]['parent'] = v
                q.append(nbr)
        if f(v): return v

def dfv(g, root, f, i = 0):
    g.nodes[root]['visited'] = True
    g.nodes[root]['visit_order'] = i
    g.nodes[root]['color'] = COLOR_SCHEDULED
    if f(root): return i, root
    i += 1
    adjacent_nodes = list(g.neighbors(root))
    adjacent_nodes.sort()
    for nbr in adjacent_nodes:
        if not g.nodes[nbr]['visited']:
            new_i, _ = dfv(g, nbr, f, i = i)
            i = new_i
    g.nodes[root]['color'] = COLOR_VISITED
    f(root)
    return i, root

```

## Breadth-first search

```python
G = nx.Graph()
default_node_attributes = {'visited': False, 'parent': None, 'visit_order': '', 'color': COLOR_UNSEEN}
G.add_nodes_from(char_range('A','J'), **default_node_attributes)
G.add_edges_from([('A','B'), ('A','C'), 
                  ('B','D'), ('B','E'), ('B','F'),
                  ('E','I'), ('E','J'),
                  ('C','G'), ('C','H')])
pos = nx.spring_layout(G)

def plot_graph(G, basename):
    global fig_index
    fig, ax = plt.subplots()
    node_colors = [node[1]['color'] for node in G.nodes(data=True)]
    nx.draw(G, pos, ax = ax, with_labels=True, node_color=node_colors, edgecolors='#000', 
            linewidths=1, font_size=22, font_weight='bold', node_size=1100)
    #bfv(G, 'A', lambda n: print(f"Visiting node '{n}'"))
    for n in G.nodes:
        x, y = pos[n]
        G.nodes[n]['x'] = random.randint(0,10)
        ax.text(x-0.1, y-0.02, f"{G.nodes[n]['visit_order']}",ha='right', va='bottom', color='#00f', fontsize=20)
    print(f"imgs/{basename}_{fig_index}.pdf")
    plt.savefig(f"imgs/{basename}_{fig_index}.pdf", bbox_inches='tight')
    fig_index += 1
    plt.show()

# plot_graph(G)

bfv(G, 'A', lambda x: plot_graph(G, 'bfs'))

```

## Depth-First Search (DFS)

```python
G = nx.Graph()
default_node_attributes = {'visited': False, 'parent': None, 'visit_order': '', 'color': COLOR_UNSEEN}
G.add_nodes_from(char_range('A','J'), **default_node_attributes)
G.add_edges_from([('A','B'), ('A','C'), 
                  ('B','D'), ('B','E'), ('B','F'),
                  ('E','I'), ('E','J'),
                  ('C','G'), ('C','H')])
pos = nx.spring_layout(G)

fig_index = 0 # reset figure index
dfv(G, 'A', lambda x: plot_graph(G, 'dfs'))

```

## Shortest path with Dijkstra

```python
g = nx.Graph()
default_node_attributes = {'visited': False, 'parent': None, 'distance': float("inf"), 'label': '', 'color': '#ededed'}
g.add_nodes_from(char_range('A','F'), **default_node_attributes)
g.add_weighted_edges_from([('A','B',1), ('A','F',3), ('B','C',3), ('B','E',5), ('B','F',1), 
                           ('C','D',2), ('D','E',1), ('D','F',6), ('E','F',2)], )
pos = { 'A': (0,0.5), 'B': (1,1), 'C': (2,1), 'D':(3,0.5), 'E':(2,0), 'F':(1,0) }

plot_basic_graph(g, pos)
```

```python
def dijkstra(g, src):
    for n in g.nodes:
        g.nodes[n]['visited'] = False
        g.nodes[n]['distance'] = float("inf")
        g.nodes[n]['parent'] = ''
    g.nodes[src]['distance'] = 0.0
    while any(filter(lambda n: not g.nodes[n]['visited'], g.nodes)):
        n = min(filter(lambda n: not g.nodes[n]['visited'], g.nodes), key = lambda elem: g.nodes[elem]['distance'])
        print(n)
        g.nodes[n]['visited'] = True
        for nbr in g.neighbors(n):
            d = g.nodes[n]['distance'] + g.get_edge_data(n,nbr)['weight']
            if d < g.nodes[nbr]['distance']:
                g.nodes[nbr]['distance'] = d
                g.nodes[nbr]['parent'] = n
        print("-" * 20)
        for n in g.nodes:
            print(f"{n:4}{('*' if g.nodes[n]['visited'] else ''):1} | {g.nodes[n]['distance']:6} | {g.nodes[n]['parent']:2} |")

def shortest_path(g, src, dest):
    path = deque([dest])
    curr = dest
    while curr != src and g.nodes[curr]['parent']:
        curr = g.nodes[curr]['parent']
        path.appendleft(curr)
    return path

dijkstra(g, 'A')

print(f"Shortest path from A to D is {shortest_path(g, 'A', 'D')} at cost {g.nodes['D']['distance']}")
```
