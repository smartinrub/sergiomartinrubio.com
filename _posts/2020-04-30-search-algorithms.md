---
title: Search Algorithms
image: https://lh3.googleusercontent.com/tRRF1wkcV5EyLFBhC65L6HVye0_6Nqirlb3TXF3OOPcA7CGsgUBjrWpAOH0CUh97Pl_BLIYD0A5wCvJdHXXxmdtdgx0lpdFjvbrFQ0uUAHYglgBSSBJojHCbLn0c30rzFrUFBLUrHXlJWjMp8Vz6X4LFqSCrjRs6G4BuTqEHPxW8zLi9WU5xyOxdaHqQx64p7XAW0UCEMUfyDpP5N-ViDs9CvXhiFxJpoEx_KN8xdFYgATl954is12_NbYpxhC_4OB0BrbgSVxIqC862G3mH_2mxWS955SAaVTDv_o7fNJAiGJwp_G_ClJd-HJzasChal2K7sv5R5ZKged9AYv6WgAvNpDSNXtL_wpFXZVVNfTJ21cyHwJ8c5LmVRtXNu46JP2N7pQs4csGjuYyEkS6yoIfU_xjQZJrobaVA1LETokVWzLkHESY48duY_8R85PyAkGy8FOCuH9Z1FyJ7lCweJfYwH-f1PNiRR8J648I0tD83jhc1CTTT9X2M-XufPsfQSBh-oBqB1Fb6vrfPmnXa7IQPB7n20VIsWnhSZZWzOhGH2JFEpfDNxisg_hX_WsqaaOKWHUdpsFjbfSq-cFhupqa5xNpYctfqoU_xg0sOdgUaAgntKI3gccGnNuJxAGVRZjxQJG9skceanyRsRE8ZuTJzFiLAe1XIjjiSEGmlitN85U1SBqCaU6irm-z0=w640-h448-no?authuser=0
categories:
    - Algorithm
mermaid: false
layout: post
---

## Introduction

**Searching** is one of the most frequent operations performed in any application, so it is important to know and understand the fundamentals of search algorithms because it will determine the performance of the application.

In software engineering, searching is the process of looking up an element from a collection of elements. These elements can be stored in arrays, trees or graphs.

Algorithms to search items in arrays, trees and graphs will be covered in the following sections.

## Time Complexity Overview

The time complexity of the algorithms is calculated with the [Big O notation](https://sergiomartinrubio.com/articles/the-big-o-notation).

Algorithm | Target | Worst Running Time | Average Running Time 
---------|----------|----------|---------
 Unordered Linear Search | unordered arrays | $$O(n)$$ | $$O(n)$$
 Ordered Linear Search | ordered arrays | $$O(n)$$ | $$O(n)$$
 Binary Search | ordered arrays | $$O(log(n))$$ | $$O(log(n))$$
 Interpolation Search | ordered arrays | $$O(log(n))$$ | $$O(log(n))$$
 Binary Search Tree | trees | $$O(log(n))$$ | $$O(n)$$
 Depth First Search For Trees | trees | $$O(n)$$ | $$O(n)$$
 Breadth First Search For Trees | trees | $$O(n)$$ | $$O(n)$$
 Trie | strings | $$O(L)$$ | $$O(L)$$
 Ternary Search Tree | string | $$O(L)$$ | $$O(L)$$
 Depth First Search For Graphs | grapths | $$O(V + E)$$ | $$O(V + E)$$
 Breadth First Search For Graphs | grapths | $$O(V + E)$$ | $$O(V + E)$$

 > Binary Search Tree: worst case for unbalanced tree

 > Depth First Search For Trees: where `n` is the number of nodes

 > Breadth First Search For Trees: where `n` is the number of nodes

 > Trie: where `L` is the length of the string

 > Ternary Search Tree: where `L` is the length of the string

 > Depth First Search For Graphs:  where `V` is the number of vertices and `E` the number of edges.

 > Breadth First Search For Graphs:  where `V` is the number of vertices and `E` the number of edges.

## Arrays

### Unordered

#### Unordered Linear Search

When the elements of the array are not sorted we have to scan the entire array to check if the array contains the item. 

The time complexity of this approach is $$O(n)$$.

```java
public class UnorderedLinearSearch {


    /**
     * Search value given an unordered array
     * @param elements unordered
     * @param value to be found
     * @return position of the value or -1 if not found
     */
    public int search(int[] elements, int value) {
        for (int i = 0; i < elements.length; i++) {
            if (elements[i] == value) {
                return i;
            }
        }
        return -1;
    }
}
```

### Ordered

#### Ordered Linear Search

If the array has its elements sorted we don't have to go through the entire array to check if the element is present. We can exit the loop as soon as the target value is greater than the value in the current position.

The time complexity is $$O(n)$$ because in the worst case we still have to scan the entire array.

```java
public class OrderedLinearSearch {

    /**
     * Search value given an ordered array
     * @param elements sorted
     * @param value to be found
     * @return position of the value or -1 if not found
     */
    public int search(int[] elements, int value) {
        for (int i = 0; i < elements.length; i++) {
            if (elements[i] == value) {
                return i;
            } else if (elements[i] > value) {
                return -1;
            }
        }
        return -1;
    }
}
```

#### Binary Search

Binary search algorithm reduce the data by half every step so you are always considering half of the input list from the previous step and throwing out the other half.

The time complexity of binary search is $$O(log(n))$$:


$$T(n)=T\biggl(\frac{n}{2}\biggl) + 1$$


$$T\biggl(\frac{n}{2}\biggl)=T\biggl(\frac{n}{4}\biggl) + 1 + 1$$

$$k$$ times:

$$T\biggl(\frac{n}{2^k}\biggl)=T\biggl(\frac{n}{2^k}\biggl) + k$$

$$T(n)=T\biggl(\frac{n}{2^k}\biggl) + k$$

$$2^k = n$$

$$k = log_2(n)$$

$$T(n) = T(1) + log(n)$$

$$T(n) = O(log(n))$$

There are two ways of implementing a _Binary Search_: iterative and recursive.

Iterative implementation:

```java
/**
* Search value with iteration given an ordered array
*
* @param elements sorted
* @param value    to be found
* @return position of the value or -1 if not found
*/
public int iterativeSearch(int[] elements, int value) {
    int lowerBound = 0;
    int upperBound = elements.length - 1;
    int midPoint;

    while (lowerBound <= upperBound) {
        midPoint = lowerBound + (upperBound - lowerBound) / 2;

        if (value == elements[midPoint]) {
            return midPoint;
        } else if (value < elements[midPoint]) {
            upperBound = midPoint - 1;
        } else {
            lowerBound = midPoint + 1;
        }
    }
    return -1;
}
```

Recursive implementation:

```java
/**
* Search value with recursion given an ordered array
* @param elements sorted
* @param value to be found
* @return position of the value or -1 if not found
*/
public int recursiveSearch(int[] elements, int value) {
    return searchRecursively(elements, value, 0, elements.length - 1);
}

private int searchRecursively(int[] elements, int value, int lowerBound, int upperBound) {
    int midPoint = lowerBound + (upperBound - lowerBound) / 2;

    if (lowerBound > upperBound) {
        return -1;
    }

    if (elements[midPoint] == value) {
        return midPoint;
    } else if (value < elements[midPoint]) {
        return searchRecursively(elements, value, lowerBound, midPoint - 1);
    } else {
        return searchRecursively(elements, value, midPoint + 1, upperBound);
    }
}
```

>To avoid the [well known overflow bug of binary search](https://ai.googleblog.com/2006/06/extra-extra-read-all-about-it-nearly.html){:target="_blank"} for arrays whose length is $$2^{30}$$ or greater. This happens because the sum of the lower and upper bound ($$midPoint = (lowerBound + upperBound) / 2$$) is greater than the maximum positive int value ($$2^{31} - 1$$). The sum overflows to a negative value, and the value stays negative when divided by two. In Java, it throws `ArrayIndexOutOfBoundsException`. The fix is to first calculate the difference $$(upperBound - lowerBound)$$ and divide by 2 and them add the `lowerBound`, $$midPoint = lowerBound + (upperBound - lowerBound) / 2$$.

#### Interpolation Search

This algorithm is based on the mathematical process of interpolation and is an improvement of _Binary Search_. Instead of dividing every time by 2 we can use a more accurate constant that can lead us closer to the target value.

>Definition: Interpolation is the process of finding a value between two elements. Linear interpolation formula: $$y = y + (y_2 - y_1) \frac{x-x_1}{x_2-x_1}$$

We can modify the linear interpolation formula to fit the search algorithm. We will use the bounds of the interval and define the following formula:

$$k = \frac{value-lowerBound}{upperBound-lowerBound}$$

The time complexity of the interpolation search is $$O(log(n))$$ if the elements are uniformly distributed, where n is the number of elements to be searched.

```java
/**
* Search value given an ordered array
*
* @param elements sorted
* @param value    to be found
* @return position of the value or -1 if not found
*/
public int search(int[] elements, int value) {
    int lowerBound = 0;
    int upperBound = elements.length - 1;

    while (lowerBound < upperBound
            && elements[lowerBound] <= value
            && elements[upperBound] >= value) {
        int constant = (value - elements[lowerBound]) /
                (elements[upperBound] - elements[lowerBound]);
        int midPoint = lowerBound + (upperBound - lowerBound) * constant;

        if (elements[midPoint] == value) {
            return midPoint;
        } else if (elements[midPoint] < value) {
            lowerBound = midPoint + 1;
        } else {
            upperBound = midPoint - 1;
        }
    }

    return -1;
}
```

## Trees

### Binary Search Tree

A **Binary Search Tree** (_BST_) is a [tree](http://sergiomartinrubio.com/articles/introduction-to-java-collections#trees) in which the left child node of every node has a value less than its parent, and on the other hand, the child node on the right has a value greater than its parent.

Searching for a value in a binary search tree is a binary process, so if the tree is balanced (similar number of nodes on both sides) the time complexity is $$O(log(n))$$ , however, if the BST is completely unbalanced, which means all the nodes will only have a right child or a left child then the time complexity is $$O(n)$$

Given a _Binary Tree_ structure you can built a search operation as follows:

```java
public boolean contains(int value) {
    return recursiveSearch(root, value) != null;
}

private Node<T> recursiveSearch(Node<T> current, int value) {
    if (current == null || value == current.value) {
        return current;
    } else if (value < current.value) {
        return recursiveSearch(current.left, value);
    } else {
        return recursiveSearch(current.right, value);
    }
}
```

### Depth First Search For Trees

_Depth First Search_ (DFS) is an algorithm for searching elements in a graph or tree data structure. The algorithm traverse the structure starting from the root and it goes all the way down a given branch, then backtracks until it finds an unexplored branch. The algorithm does this until the entire structure is visited.

{% include elements/figure.html image="https://lh3.googleusercontent.com/JDSCBwlqwFcPCj_MXSlmNcDu74clxQm0P-QlutWqeYkzO9mM2O_csSsOPP73s2BOotg37LS8Fax618xkB20-5NyJUfaEtrf_EaURolXuLPvsVk9t9mBN7sTbpwTbSiROyjKQrLy9Qg=w800" caption="Depth First Search" %}

This algorithm usually uses a [Stack](http://sergiomartinrubio.com/articles/introduction-to-java-collections#stack) structure in order to keep track of the visited nodes. We always want to visit the last node seen and we keep the rest of the nodes to be visited later on.

One of the approaches to visit the nodes while moving to the leftmost node in the tree. Once there are no more children on the left of a node, the children on the right are visited. This strategy is called **pre-order**.

The time complexity of the DFS for search operations is $$O(n)$$ where `n` is the number of nodes.

This is how you can implement a pre-order DFS for a binary tree structure in Java:

```java
/**
* Search value in Binary Tree using Depth First Search Algorithm
* @param value to be found
* @return true if value is found otherwise return false
*/
public boolean PreOrderSearch(int value) {
    Stack<Node> stack = new Stack<>();
    Node currentNode;
    stack.push(this.root);

    while (!stack.empty()) {
        currentNode = stack.pop();

        if (currentNode.value == value) {
            return true;
        }

        if (currentNode.left != null) {
            stack.push(currentNode.left);
        }

        if (currentNode.right != null) {
            stack.push(currentNode.right);
        }
    }
    return false;
}
```

### Breadth First Search For Trees

_Breadth First Search_ (BFS) is also an algorithm for traversing or searching a tree or graph structure. It starts at the tree root and explores the neighbor nodes first, before moving to its children.

{% include elements/figure.html image="https://lh3.googleusercontent.com/Ctl5aXUjaypUQpY9ACx9t2Cb5M1v4vPYo6maOg0JwoCx1vgz41sB_kqcLDVOZ-cTkqBlRfOFVTZNHhqj7dMkBQK3BkowwrpInrhrVH5HttwL3g0zPKLke5jkM3rgcWWJl8Qa9iwNEA=w800" caption="Breadth First Search" %}

This algorithm usually uses a [Queue](http://sergiomartinrubio.com/articles/introduction-to-java-collections#queue-interface) structure in order to keep track of the visited nodes.

The time complexity of the BFS for search operations is $$O(n)$$ where `n` is the number of nodes.

This is how you can implement a BFS for a binary tree structure in _Java_:

```java
/**
* Search value in Binary Tree using Breadth First Search Algorithm
*
* @param value to be found
* @return true if value is found otherwise return false
*/
public boolean search(int value) {
    Queue<Node> queue = new ArrayDeque<>();

    queue.add(this.root);
    while (!queue.isEmpty()) {
        Node currentNode = queue.poll();
        LOGGER.info(String.valueOf(currentNode.value));
        if (currentNode.value == value) {
            return true;
        }

        if (currentNode.left != null) {
            queue.add(currentNode.left);
        }

        if (currentNode.right != null) {
            queue.add(currentNode.right);
        }
    }
    return false;
}
```

## Strings

### Trie

A **trie**, also known as prefix-tree, is a search tree in which each node contains an array where the elements are unique characters. If the strings are formed with English alphabet characters it will contain up to 26 elements.

The time complexity of the search and find string operations is $$O(L)$$ where `L` is the length of a string.

The main disadvantage of tries is that they require a lot of memory for storing strings. Each node has an array and in most of the cases these arrays are not full.

We can implement a Trie as follows:

1. First, the `Trie` class is created with an inner class. The inner class `TrieNode` will store a character, a list of nodes and a flag that says if a character belongs to the end of a word:

    ```java
    public class Trie {

        private TrieNode root;

        private static class TrieNode {
            char character;
            List<TrieNode> childNodes;
            boolean isLeaf;
        }
    }
    ```

2. Add a node to the Trie:

    ```java
    /**
     * Add word to trie
     * @param word to add to trie
     */
    public void add(String word) {
        if (root == null) {
            root = new TrieNode();
            root.childNodes = new ArrayList<>();
        }
        List<TrieNode> childNodes = root.childNodes;

        for (int i = 0; i < word.length(); i++) {
            char currentChar = word.charAt(i);
            TrieNode tail;
            Optional<TrieNode> trieNode = childNodes.stream()
                    .filter(node -> node.character == currentChar)
                    .findFirst();
            if (trieNode.isPresent()) {
                tail = trieNode.get();
            } else {
                tail = new TrieNode();
                tail.character = currentChar;
                tail.childNodes = new ArrayList<>();
                childNodes.add(tail);
            }
            childNodes = tail.childNodes;

            if (i == word.length() - 1) {
                tail.isLeaf = true;
            }
        }
    }
    ```

3. Search for a word:

    ```java
    /**
     * Search word in trie
     * @param word to search in trie
     * @return true if the word was found otherwise return false
     */
    public boolean search(String word) {
        List<TrieNode> childNodes = root.childNodes;
        TrieNode tail = null;
        for (int i = 0; i < word.length(); i++) {
            char character = word.charAt(i);
            Optional<TrieNode> trieNode = childNodes.stream()
                    .filter(node -> node.character == character)
                    .findFirst();
            if (trieNode.isPresent()) {
                tail = trieNode.get();
                childNodes = tail.childNodes;
            } else {
                return null;
            }
        }
        return tail != null && tail.isLeaf;
    }
    ```

4. We can also implement a lookup method to search words by prefix. The simplest solution is to use [Depth First Search](http://sergiomartinrubio.com/articles/algorithms-to-search-through-lists-and-trees-data-structures#tree-depth-first-search) (DFS) to go down the trie while building strings for each subtrie:

    ```java
    /**
     * Use _Depth First Search_ (DFS) to go down the trie while building string for each subtrie.
     * @param prefix use to find words
     * @return list of words that match the prefix
     */
    public List<String> lookup(String prefix) {
        // Use the previous function to find the last node of the prefix
        TrieNode tail = getTrieNode(prefix);
        if (tail == null) {
            return List.of();
        }
        Stack<TrieNode> stack = new Stack<>();
        stack.push(tail);
        TrieNode current;
        StringBuilder word = new StringBuilder();
        // Start building the first word with the prefix
        word.append(prefix, 0, prefix.length() - 1);
        List<String> words = new ArrayList<>();
        while (!stack.empty()) {
            current = stack.pop();
            word.append(current.character);
            // End of the word?
            if (current.isLeaf) {
                words.add(word.toString());
                word = new StringBuilder();
                // The prefix is added for each new word
                word.append(prefix);
            }
            for (TrieNode trieNode : current.childNodes) {
                stack.push(trieNode);
            }
        }
        return words;
    }
    ```

### Ternary Search Tree

A _Ternary Search Tree_ (TST) is a combination of [binary search trees](http://sergiomartinrubio.com/articles/algorithms-to-search-through-lists-and-trees-data-structures#binary-search-tree) and [tries](http://sergiomartinrubio.com/articles/algorithms-to-search-through-lists-and-trees-data-structures#trie). Unlike tries, TST is more memory efficient since each node only contains pointers to three nodes, but keeping the time efficiency benefits.

 - The node on the left contains the node whose value is less than the value in the current node.
 - The node on the middle contains the node whose value is equals to the value in the current node.
 - The node on the right contains the node whose value is greater than the value in the current node.

 Each node also contains a flag to specify if the current node belongs to the end of word.

 The time complexity for search operations is $$O(L)$$ where `L` is the length of the string to be found.

 Implementation in Java:

 1. Define the ternary search tree class and the node for this structure:

    ```java
    public class TernarySearchTree {

        private TernaryTreeNode root;

        static class TernaryTreeNode {
            char value;
            boolean isLeaf;
            TernaryTreeNode left;
            TernaryTreeNode middle;
            TernaryTreeNode right;

            public TernaryTreeNode(char value) {
                this.value = value;
            }
        }
    }
    ```

2. Add recursively a word to the ternary search tree:

    ```java
    /**
    * Add string to ternary search tree with recursion
    *
    * @param word to be inserted
    */
    public void add(String word) {
        int position = 0;
        if (root == null) {
            root = new TernaryTreeNode(word.charAt(position++));
            if (position == word.length()) {
                root.isLeaf = true;
                return;
            }
        }
        recursiveInsert(root, word, position);
    }

    private void recursiveInsert(TernaryTreeNode node, String word, int position) {

        if (word.charAt(position) < node.value) {
            if (node.left != null) {
                recursiveInsert(node.left, word, position);
            } else {
                node.left = new TernaryTreeNode(word.charAt(position++));
                if (position == word.length()) {
                    node.left.isLeaf = true;
                } else {
                    recursiveInsert(node.left, word, position);
                }
            }
        } else if (word.charAt(position) > node.value) {
            if (node.right != null) {
                recursiveInsert(node.right, word, position);
            } else {
                node.right = new TernaryTreeNode(word.charAt(position++));
                if (position == word.length()) {
                    node.right.isLeaf = true;
                } else {
                    recursiveInsert(node.right, word, position);
                }
            }
        } else {
            if (node.middle != null) {
                recursiveInsert(node.middle, word, position);
            } else {
                node.middle = new TernaryTreeNode(word.charAt(position++));
                if (position == word.length()) {
                    node.middle.isLeaf = true;
                } else {
                    recursiveInsert(node.middle, word, position);
                }
            }
        }
    }
    ```

3. Method to search in the ternary search tree:

    ```java
    /**
    * Search word in the ternary search tree
    *
    * @param word to be searched
    * @return true if word is found, otherwise return false
    */
    public boolean search(String word) {
        TernaryTreeNode currentNode = root;
        int position = 0;
        while (currentNode != null) {
            if (word.charAt(position) < currentNode.value) {
                currentNode = currentNode.left;
            } else if (word.charAt(position) > currentNode.value) {
                currentNode = currentNode.right;
            } else {
                if (position == word.length() - 1 && currentNode.isLeaf) {
                    return true;
                }
                if (currentNode.middle != null) {
                    currentNode = currentNode.middle;
                }
                position++;
            }
        }
        return false;
    }
    ```

## Graphs

### Depth First Search For Graphs

Depth First Search for graphs is similar to Depth First Search for trees. Nodes in a graph can be connected to more than three nodes, because they do not follow a tree structure. This means that they might contain cycles and a node may be visited twice. To avoid visiting a node more than once we have to mark those nodes already visited.

The time complexity is $$O(V + E)$$  where `V` is the number of vertices and `E` the number of edges.

Implementation in _Java_ for traversing a graph with DFS algorithm:

1. Create graph class:

    ```java
    public class Graph {

        private static final Logger LOGGER = Logger.getLogger(Graph.class.getName());

        private final Map<Node, LinkedList<Node>> neighborsByNode;

        public Graph() {
            neighborsByNode = new HashMap<>();
        }

    }
    ```

2. Add elements to graph:

    ```java
    /**
    * Add edge to the graph
    *
    * @param origin      value
    * @param destination value
    */
    public void addEdge(Node origin, Node destination) {
        if (!neighborsByNode.containsKey(origin)) {
            neighborsByNode.put(origin, null);
        }
        if (!neighborsByNode.containsKey(destination)) {
            neighborsByNode.put(destination, null);
        }

        add(origin, destination);
        add(destination, origin);
    }

    private void add(Node origin, Node destination) {
        LinkedList<Node> neighbors = neighborsByNode.get(origin);

        if (neighbors != null) {
            neighborsByNode.remove(origin);
        } else {
            neighbors = new LinkedList<>();
        }
        neighbors.add(destination);
        neighborsByNode.put(origin, neighbors);
    }
    ```

3. Method to search value in graph using recursion:

    ```java
    /**
    * Search node in graph with recursion
    * @param node to be found
    * @return true if found, otherwise return false
    */
    public boolean depthFirstSearch(Node node) {
        for (Node currentNode : neighborsByNode.keySet()) {

            if (node.value == currentNode.value) {
                return true;
            }
            if (!currentNode.visited) {
                if (depthFirstSearchRecursive(currentNode, node)) {
                    return true;
                }
            }
        }
        return false;
    }

    private boolean depthFirstSearchRecursive(Node currentNode, Node node) {

        currentNode.visited = true;

        LinkedList<Node> allNeighbors = neighborsByNode.get(currentNode);
        if (allNeighbors == null) {
            return false;
        }

        for (Node neighbor : allNeighbors) {
            LOGGER.info(String.valueOf(currentNode.value));
            if (node.value == neighbor.value) {
                return true;
            }
            if (!neighbor.visited) {
                depthFirstSearchRecursive(neighbor, node);
            }
        }
        return false;
    }
    ```

### Breadth First Search For Graphs

Breadth First Search for graphs is similar to Breadth First Search for trees. BFS also requires to keep track of the visited nodes.

The time complexity is $$O(V + E)$$  where `V` is the number of vertices and `E` the number of edges.

The graph and node definition is the same as the one described for DFS algorithm, and we can implement the BFS as follows:

```java
/**
* Search node in graph
*
* @param node to be found
* @return true if found otherwise return false
*/
public boolean breadthFirstSearch(Node node) {

    Node firstNode = neighborsByNode.keySet().iterator().next();

    if (firstNode == null) {
        return false;
    }

    ArrayDeque<Node> queue = new ArrayDeque<>();
    queue.add(firstNode);

    while (!queue.isEmpty()) {
        Node currentNode = queue.poll();

        // Check if node was found.
        LOGGER.info(String.valueOf(currentNode.value));
        if (currentNode.value == node.value) {
            return true;
        }

        if (currentNode.visited) {
            continue;
        }

        currentNode.visited = true;
        LinkedList<Node> allNeighbors = neighborsByNode.get(currentNode);

        if (allNeighbors == null) {
            continue;
        }

        for (Node neighbor : allNeighbors) {
            if (!neighbor.visited) {
                queue.add(neighbor);
            }
        }
    }
    return false;
}
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/java-search-algorithms.git" text="Examples" %}
</p>
