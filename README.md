# Graph Theory W10

Group 1
| Name           | NRP        |
| ---            | ---        |
| Alif Muflih Jauhary | 5025241003 |
| Makna Alam Pratama | 5025241077 |
| Rayen Yeriel Mangiwa | 5025241262 |

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
    nx.draw(G, pos, with_labels=True, node_color=node_colors,
            node_size=900, cmap=plt.cm.Set3, font_weight='bold', edge_color="gray")

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

---

## How to Run
Enter these nodes in order:
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
