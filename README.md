# Graph Theory W10

Group 1
| Name           | NRP        |
| ---            | ---        |
| Alif Muflih Jauhary | 5025241003 |
| Makna Alam Pratama | 5025241077 |
| Rayen Yeriel Mangiwa | 5025241262 |

---

## Welsh-Powell Algorithm

```python
import networkx as nx
import matplotlib.pyplot as plt

def welsh_powell(nodes, edges):
    graph = {}
    for node in nodes:
        graph[node] = []
    
    for u, v, _ in edges:
        graph[u].append(v)
        graph[v].append(u)

    degrees = [(node, len(neighbors)) for node, neighbors in graph.items()]
    degrees.sort(key=lambda x: x[1], reverse=True)
    sorted_nodes = [node for node, _ in degrees]

    colors = {}
    color_num = 1
    
    for node in sorted_nodes:
        if node in colors:
            continue

        colors[node] = color_num

        for other_node in sorted_nodes:
            if other_node in colors:
                continue

            can_use_color = True
            for colored_node, col in colors.items():
                if col == color_num and other_node in graph[colored_node]:
                    can_use_color = False
                    break
            
            if can_use_color:
                colors[other_node] = color_num
        
        color_num += 1
    
    return colors


def draw_graph(nodes, edges, colors):
    G = nx.Graph()
    G.add_nodes_from(nodes)

    for u, v, cost in edges:
        G.add_edge(u, v, weight=cost)

    color_list = [colors[node] for node in G.nodes()]

    if len(nodes) == 1:
        pos = {nodes[0]: (0, 0)}
    else:
        pos = nx.spring_layout(G, seed=42)
    
    plt.figure(figsize=(8, 6))

    nx.draw(G, pos, 
            with_labels=True,
            node_color=color_list,
            node_size=1000,
            cmap=plt.cm.Set3,
            font_size=12,
            font_weight='bold',
            edge_color='#888888',
            width=2)

    if edges:
        edge_labels = nx.get_edge_attributes(G, 'weight')
        nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels, font_size=10)
    
    plt.title("Graph Coloring using Welsh-Powell Algorithm", fontsize=14, pad=20)
    plt.axis('off')
    plt.tight_layout()
    plt.show()


def main():
    print("=" * 50)
    print("Welsh-Powell Graph Coloring Algorithm")
    print("=" * 50)

    n = int(input("\nHow many nodes? "))
    nodes = []
    for i in range(n):
        node_name = input(f"Node {i+1}: ").strip()
        nodes.append(node_name)

    m = int(input(f"\nHow many edges? "))
    edges = []
    print("(Enter the two nodes and the edge weight/cost)")
    for i in range(m):
        print(f"\nEdge {i+1}:")
        u = input("  From: ").strip()
        v = input("  To: ").strip()
        cost = float(input("  Weight: "))
        edges.append((u, v, cost))

    print("\n" + "=" * 50)
    print("Running Welsh-Powell algorithm...")
    print("=" * 50)
    
    colors = welsh_powell(nodes, edges)

    print("\nColoring Results:")
    print("-" * 30)
    for node in nodes:
        print(f"  {node:<10} -> Color {colors[node]}")
    
    num_colors = len(set(colors.values()))
    print(f"\nTotal colors used: {num_colors}")
    print("=" * 50)

    draw_graph(nodes, edges, colors)


if __name__ == "__main__":
    main()
```

---

## How to Use
Enter within the following order:
- Number of nodes
- Names for each nodes
- Number of edges
- Connection of edges with vertices (with weight)

---

## Sample Test Case

### Input

```
How many nodes? 5
Node 1: A
Node 2: B
Node 3: C
Node 4: D
Node 5: E

How many edges? 6
(Enter the two nodes and the edge weight/cost)

Edge 1:
  From: A
  To: B
  Weight: 1

Edge 2:
  From: A
  To: C
  Weight: 2

Edge 3:
  From: B
  To: C
  Weight: 3

Edge 4:
  From: B
  To: D
  Weight: 4

Edge 5:
  From: C
  To: D
  Weight: 5

Edge 6:
  From: D
  To: E
  Weight: 6
```

---

### Output

```
==================================================
Running Welsh-Powell algorithm...
==================================================

Coloring Results:
------------------------------
  A          -> Color 1
  B          -> Color 2
  C          -> Color 3
  D          -> Color 1
  E          -> Color 2

Total colors used: 3
==================================================
```
---

## Hungarian Algorithm

```python
import numpy as np
from scipy.optimize import linear_sum_assignment
import networkx as nx
import matplotlib.pyplot as plt


def hungarian_algorithm(cost_matrix):
    row_ind, col_ind = linear_sum_assignment(cost_matrix)
    total_cost = cost_matrix[row_ind, col_ind].sum()
    return row_ind, col_ind, total_cost


def visualize_assignment(cost_matrix, row_ind, col_ind, workers, jobs):
    G = nx.Graph()
    G.add_nodes_from(workers, bipartite=0)
    G.add_nodes_from(jobs, bipartite=1)

    all_edges = []
    for i, worker in enumerate(workers):
        for j, job in enumerate(jobs):
            all_edges.append((worker, job, {'weight': cost_matrix[i, j]}))
    G.add_edges_from(all_edges)

    pos = {}
    pos.update((node, (1, -i)) for i, node in enumerate(workers))
    pos.update((node, (2, -i)) for i, node in enumerate(jobs))

    plt.figure(figsize=(8, 6))

    nx.draw_networkx_nodes(G, pos, nodelist=workers, node_color='skyblue', node_size=1000)
    nx.draw_networkx_nodes(G, pos, nodelist=jobs, node_color='lightgreen', node_size=1000)
    nx.draw_networkx_labels(G, pos, font_size=12, font_weight='bold')
    nx.draw_networkx_edges(G, pos, edgelist=all_edges, width=0.5, alpha=0.5, edge_color='gray')

    edge_labels = nx.get_edge_attributes(G, 'weight')
    nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels, font_color='gray', font_size=8)

    optimal_edges = []
    optimal_edge_labels = {}
    for i, j in zip(row_ind, col_ind):
        worker_node = workers[i]
        job_node = jobs[j]
        optimal_edges.append((worker_node, job_node))
        optimal_edge_labels[(worker_node, job_node)] = cost_matrix[i, j]

    nx.draw_networkx_edges(G, pos, edgelist=optimal_edges, width=2.5, alpha=1, edge_color='red')
    nx.draw_networkx_edge_labels(G, pos, edge_labels=optimal_edge_labels, font_color='red', font_size=10)

    plt.title("Hungarian Algorithm - Optimal Assignment")
    plt.axis('off')
    plt.show()


def main():
    print("=== Hungarian Algorithm ===")

    try:
        n_workers = int(input("Enter number of nodes in Group A (e.g., workers): "))
        workers = [input(f"Enter Group A Node {i+1} name: ").strip() for i in range(n_workers)]

        n_jobs = int(input("Enter number of nodes in Group B (e.g., jobs): "))
        jobs = [input(f"Enter Group B Node {i+1} name: ").strip() for i in range(n_jobs)]

        if n_workers != n_jobs:
            print("\nError: Number of nodes in Group A and Group B must be EQUAL.")
            return

        print("\nEnter the cost for each edge (from Group A to Group B):")
        cost_matrix = np.zeros((n_workers, n_jobs))
        for i in range(n_workers):
            for j in range(n_jobs):
                cost = float(input(f"Cost from {workers[i]} -> {jobs[j]}: "))
                cost_matrix[i, j] = cost

        print("\nCost Matrix (Input):")
        print(cost_matrix)

        row_ind, col_ind, total_cost = hungarian_algorithm(cost_matrix)

        print("\n--- Output: Optimal Pairs ---")
        for i, j in zip(row_ind, col_ind):
            print(f"{workers[i]} \t -> {jobs[j]} \t (Cost: {cost_matrix[i, j]})")

        print(f"\nTotal Minimum Cost: {total_cost}")

        visualize_assignment(cost_matrix, row_ind, col_ind, workers, jobs)

    except ValueError:
        print("\nError: Invalid input.")
    except Exception as e:
        print(f"\nAn error occurred: {e}")


if __name__ == "__main__":
    main()
```

---

### How to Use

3. Enter as follow:

   * Number of nodes in **Group A (workers)**
   * Number of nodes in **Group B (jobs)**
   * Cost for each possible assignment between them

---

### Example Test Case

#### Input

```
=== Hungarian Algorithm (Assignment Problem) ===
Enter number of nodes in Group A (e.g., workers): 3
Enter Group A Node 1 name: A1
Enter Group A Node 2 name: A2
Enter Group A Node 3 name: A3
Enter number of nodes in Group B (e.g., jobs): 3
Enter Group B Node 1 name: B1
Enter Group B Node 2 name: B2
Enter Group B Node 3 name: B3
Enter the cost for each edge (from Group A to Group B):
Cost from A1 -> B1: 4
Cost from A1 -> B2: 2
Cost from A1 -> B3: 5
Cost from A2 -> B1: 3
Cost from A2 -> B2: 7
Cost from A2 -> B3: 6
Cost from A3 -> B1: 8
Cost from A3 -> B2: 1
Cost from A3 -> B3: 9
```

---

#### Output

```
Cost Matrix (Input):
[[4. 2. 5.]
 [3. 7. 6.]
 [8. 1. 9.]]

--- Output: Optimal Pairs ---
A1 	 -> B3 	 (Cost: 5.0)
A2 	 -> B1 	 (Cost: 3.0)
A3 	 -> B2 	 (Cost: 1.0)

Total Minimum Cost: 9.0
```
