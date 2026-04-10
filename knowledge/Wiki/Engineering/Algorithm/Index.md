Techniques

Hashing

SHA256

HMAC

RSA

AES

# Techniques

1. **Sliding window**: The Sliding Window technique is a commonly used algorithmic technique for efficiently solving problems that involve subarrays or substrings. It works by maintaining a "window" or a fixed-size subset of elements within the given array or string and efficiently updating the window as it "slides" through the input. For examples:
    - Longest Substring Without Repeating
2. **Two Pointers**: This technique involves using two pointers to traverse the array or string in tandem or to maintain a specific distance between them. It is often used for solving problems related to searching, partitioning, or validating elements in the array. Examples include the two-pointer approach for finding pairs with a given sum in a sorted array or for checking if a string is a palindrome.
3. **Prefix Sum**: The prefix sum technique involves precomputing and storing cumulative sums of elements in an array or string. This technique is useful for solving problems that involve calculating sums or finding subarrays with a specific sum efficiently. Examples include finding subarrays with a target sum or counting the number of subarrays with a given sum.
4. **Binary Search**: Binary search is a divide-and-conquer technique used for efficiently searching for an element in a sorted array. It repeatedly divides the search space in half until the target element is found or determined to be absent. Binary search is a fundamental technique for solving problems related to searching and can also be applied to other scenarios such as finding the rotation point in a rotated sorted array.
5. **Sliding Window + Hashing**: This technique combines the Sliding Window technique with hashing to efficiently solve problems that involve substring or subarray matching. It uses a hash table or a similar data structure to store the frequency or occurrence of elements within the sliding window. This technique is often used for solving problems like finding the longest substring with distinct characters or finding anagrams in a string.
6. **Sweep Line**: The sweep line technique involves simulating the movement of a line or a sweep across the input space to identify and process events or intersections efficiently. It is often used in geometric or interval-related problems. Examples include finding overlapping intervals or counting the number of points inside a rectangle.
7. **Graph Traversal**: Graph traversal techniques involve systematically exploring or visiting nodes or vertices in a graph. Common graph traversal algorithms include Breadth-First Search (BFS) and Depth-First Search (DFS). These techniques are useful for solving problems such as finding the shortest path, detecting cycles, or traversing connected components in a graph.
8. **Dynamic Programming**: Dynamic Programming (DP) is a technique that involves solving complex problems by breaking them down into simpler overlapping subproblems and storing the solutions to these subproblems for later use. By reusing these solutions, DP can significantly reduce the time complexity of the overall problem. It is often used for optimization problems and problems that exhibit overlapping substructure.
9. **Topological Sorting**: Topological sorting is a technique used for ordering the vertices of a directed acyclic graph (DAG) in such a way that for every directed edge (u, v), vertex u comes before vertex v in the ordering. Topological sorting is commonly used in tasks such as task scheduling, dependency resolution, and finding a linear order of events.
10. **Branch and Bound**: Branch and Bound is a technique used for solving optimization problems by exploring the search space in a systematic way. It involves branching into different paths or subsets of the problem space and using bounds or heuristics to eliminate branches that cannot lead to an optimal solution. This technique is often used for solving problems such as the traveling salesman problem or the knapsack problem.
11. **Randomized Algorithms**: Randomized algorithms introduce randomness into the algorithm design to achieve desired properties or improve efficiency. These algorithms use randomization techniques such as random sampling, random choices, or randomization in data structures. Randomized algorithms are often used for problems like randomized sorting, randomized selection, or approximating solutions.
12. **Divide and Conquer**: This technique involves breaking down a problem into smaller, more manageable subproblems, solving them recursively, and combining the solutions to obtain the final result. Examples of algorithms that use this technique include merge sort and quicksort.
13. **Greedy Algorithms**: Greedy algorithms make locally optimal choices at each step with the hope that they will lead to a globally optimal solution. At each step, the algorithm selects the best available option without considering future consequences. The classic example is the greedy algorithm for finding the minimum spanning tree in a graph, known as Kruskal's algorithm.
14. **Dynamic Programming**: Dynamic programming is a technique for solving problems by breaking them down into overlapping subproblems and solving each subproblem only once. The solutions to subproblems are stored and reused to solve larger subproblems until the main problem is solved. Dynamic programming is commonly used for optimization problems, such as finding the shortest path or the maximum value. The classic example is the dynamic programming solution for the Fibonacci sequence.
15. **Backtracking**: Backtracking is a general algorithmic technique for exploring all possible solutions to a problem by incrementally building candidates and abandoning them if they cannot lead to a valid solution. It is often used for problems that involve searching for solutions in a large search space, such as the N-Queens problem or the traveling salesman problem.
16. **Brute Force**: Brute force involves systematically trying all possible solutions to a problem until the correct one is found. While this approach may be inefficient for large problem instances, it can be a useful technique for small or simple problems, providing a baseline solution for comparison and validation.

  

# Hashing

## SHA256

SHA 256 is a part of the SHA 2 family of algorithms, where SHA stands for Secure Hash Algorithm. Published in 2001, it was a joint effort between the NSA and NIST to introduce a successor to the SHA 1 family, which was slowly losing strength against [brute force attacks.](https://www.simplilearn.com/tutorials/cryptography-tutorial/brute-force-attack)

  

## HMAC

[HMACSHA256](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.hmacsha256?view=net-8.0) is a type of keyed hash algorithm that is constructed from the SHA-256 hash function and used as a Hash-based Message Authentication Code (HMAC). The HMAC process mixes a secret key with the message data, hashes the result with the hash function, mixes that hash value with the secret key again, and then applies the hash function a second time. The output hash is 256 bits in length.

## RSA

- **Public Key** & **Private Key**
- Usage
    - **Encrypt and send.** Using the [public key](https://www.okta.com/identity-101/public-key-encryption/) and an agreed-upon padding scheme, you'll scramble your note and send it along. When the message arrives, the person will use a private key to undo the work and see what's inside
    - **Digital signature**: Sign by private key. The recipient will use the hash value and your public key to reverse the process

## AES