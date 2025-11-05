# Graph Theory W10

Group 1
| Name           | NRP        |
| ---            | ---        |
| Alif Muflih Jauhary | 5025241003 |
| Makna Alam Pratama | 5025241077 |
| Rayen Yeriel Mangiwa | 5025241262 |

---

```python
import networkx as nx
import matplotlib.pyplot as plt

def welsh_powell(nodes, edges):
    # --- Build adjacency list ---
    graph = {v: [] for v in nodes}
    for u, v, cost in edges:
        graph[u].append(v)
        graph[v].append(u)  # undirected graph

    # --- Step 1: Compute degrees ---
    degrees = {v: len(adj) for v, adj in graph.items()}

    # --- Step 2: Sort by descending degree ---
    sorted_vertices = sorted(degrees, key=degrees.get, reverse=True)

    # --- Step 3–5: Welsh–Powell Coloring ---
    colors = {}
    current_color = 1

    for vertex in sorted_vertices:
        if vertex not in colors:
            colors[vertex] = current_color
            for other in sorted_vertices:
                if other not in colors:
                    # check adjacency with all same-colored vertices
                    if all((other not in graph[v]) for v, c in colors.items() if c == current_color):
                        colors[other] = current_color
            current_color += 1

    return colors


def visualize_coloring(nodes, edges, colors):
    G = nx.Graph()
    G.add_nodes_from(nodes)  # ✅ Make sure single nodes appear even if no edges
    for u, v, cost in edges:
        G.add_edge(u, v, weight=cost)

    # Prepare color map
    node_colors = [colors[n] for n in G.nodes()]

    # --- Fix: handle single-node or edgeless graphs ---
    if len(G.nodes) == 1:
        pos = {list(G.nodes())[0]: (0, 0)}
    else:
        pos = nx.spring_layout(G, seed=42)

    # Draw nodes and edges
    plt.figure(figsize=(7, 5))
    nx.draw(
        G, pos, with_labels=True,
        node_color=node_colors,
        node_size=900,
        cmap=plt.cm.Set3,
        font_weight='bold',
        edge_color="gray"
    )

    # Draw edge weights (if any)
    if edges:
        labels = nx.get_edge_attributes(G, 'weight')
        nx.draw_networkx_edge_labels(G, pos, edge_labels=labels)

    # If no edges, show a message in the plot
    if not edges:
        plt.text(0, -0.2, "(no edges in graph)", ha='center', fontsize=10, color='gray')

    plt.title("Welsh–Powell Graph Coloring")
    plt.axis("off")
    plt.show()


def main():
    print("=== Welsh–Powell Graph Coloring ===")
    n = int(input("Enter number of nodes: "))
    nodes = [input(f"Enter node {i+1} name: ").strip() for i in range(n)]

    m = int(input("Enter number of edges: "))
    edges = []
    for i in range(m):
        u = input(f"Edge {i+1} - from node: ").strip()
        v = input(f"Edge {i+1} - to node: ").strip()
        cost = float(input(f"Edge {i+1} - cost: "))
        edges.append((u, v, cost))

    # Run algorithm
    colors = welsh_powell(nodes, edges)

    # Output result
    print("\nNode Color Assignments:")
    for node in nodes:
        print(f"{node} → Color {colors[node]}")

    # Visualization
    visualize_coloring(nodes, edges, colors)


if __name__ == "__main__":
    main()
```

---

## How to Run
Enter these data in order:
   * Number of nodes
   * Node names
   * Number of edges
   * Edge connections and weights (costs)

---

## Sample Test Case

### Input

```
=== Welsh–Powell Graph Coloring ===
Enter number of nodes: 5
Enter node 1 name: A
Enter node 2 name: B
Enter node 3 name: C
Enter node 4 name: D
Enter node 5 name: E
Enter number of edges: 6
Edge 1 - from node: A
Edge 1 - to node: B
Edge 1 - cost: 1
Edge 2 - from node: A
Edge 2 - to node: C
Edge 2 - cost: 1
Edge 3 - from node: B
Edge 3 - to node: D
Edge 3 - cost: 1
Edge 4 - from node: B
Edge 4 - to node: E
Edge 4 - cost: 1
Edge 5 - from node: C
Edge 5 - to node: D
Edge 5 - cost: 1
Edge 6 - from node: D
Edge 6 - to node: E
Edge 6 - cost: 1
```

---

### Output

```
Node Color Assignments:
A → Color 1
B → Color 2
C → Color 2
D → Color 1
E → Color 3
```
---

## Hungarian Algorithm (Assignment Problem)

```python
import numpy as np
from scipy.optimize import linear_sum_assignment
import networkx as nx
import matplotlib.pyplot as plt


def hungarian_algorithm(cost_matrix):
    """Solve the assignment problem using the Hungarian algorithm."""
    row_ind, col_ind = linear_sum_assignment(cost_matrix)
    total_cost = cost_matrix[row_ind, col_ind].sum()
    return row_ind, col_ind, total_cost


def visualize_assignment(cost_matrix, row_ind, col_ind, workers, jobs):
    """Visualize the optimal worker-job assignment using NetworkX."""
    G = nx.Graph()
    G.add_nodes_from(workers, bipartite=0)
    G.add_nodes_from(jobs, bipartite=1)

    # Add all edges with weights
    all_edges = []
    for i, worker in enumerate(workers):
        for j, job in enumerate(jobs):
            all_edges.append((worker, job, {'weight': cost_matrix[i, j]}))
    G.add_edges_from(all_edges)

    # Positioning for bipartite graph layout
    pos = {}
    pos.update((node, (1, -i)) for i, node in enumerate(workers))
    pos.update((node, (2, -i)) for i, node in enumerate(jobs))

    plt.figure(figsize=(8, 6))

    # Draw all edges and nodes
    nx.draw_networkx_nodes(G, pos, nodelist=workers, node_color='skyblue', node_size=1000)
    nx.draw_networkx_nodes(G, pos, nodelist=jobs, node_color='lightgreen', node_size=1000)
    nx.draw_networkx_labels(G, pos, font_size=12, font_weight='bold')
    nx.draw_networkx_edges(G, pos, edgelist=all_edges, width=0.5, alpha=0.5, edge_color='gray')

    # Display edge weights (costs)
    edge_labels = nx.get_edge_attributes(G, 'weight')
    nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels, font_color='gray', font_size=8)

    # Highlight optimal edges (red)
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
    print("=== Hungarian Algorithm (Assignment Problem) ===")

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

### How to Run

3. Follow the prompts to enter:

   * Number of nodes in **Group A (workers)**
   * Number of nodes in **Group B (jobs)**
   * Cost for each possible assignment between them

---

### Example Test Case

#### Input

```
=== Hungarian Algorithm (Assignment Problem) ===
Enter number of nodes in Group A (e.g., workers): 3
Enter Group A Node 1 name: W1
Enter Group A Node 2 name: W2
Enter Group A Node 3 name: W3
Enter number of nodes in Group B (e.g., jobs): 3
Enter Group B Node 1 name: J1
Enter Group B Node 2 name: J2
Enter Group B Node 3 name: J3

Enter the cost for each edge (from Group A to Group B):
Cost from W1 -> J1: 9
Cost from W1 -> J2: 2
Cost from W1 -> J3: 7
Cost from W2 -> J1: 6
Cost from W2 -> J2: 4
Cost from W2 -> J3: 3
Cost from W3 -> J1: 5
Cost from W3 -> J2: 8
Cost from W3 -> J3: 1
```

---

#### Output

```
Cost Matrix (Input):
[[9. 2. 7.]
 [6. 4. 3.]
 [5. 8. 1.]]

--- Output: Optimal Pairs ---
W1 	 -> J2 	 (Cost: 2.0)
W2 	 -> J3 	 (Cost: 3.0)
W3 	 -> J1 	 (Cost: 5.0)

Total Minimum Cost: 10.0
```
