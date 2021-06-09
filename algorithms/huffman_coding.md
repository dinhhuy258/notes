---
title: Huffman coding
parent: Algorithms
---

# Huffman coding

Huffman Coding is a technique of compressing data to reduce its size without losing any of the details.

Huffman Coding is generally useful to compress the data in which there are frequently occurring characters.

## How huffman coding works

Suppose the following stringis to be sent over a network: BCAADDDCCACACAC

Each character occupies `8` bits. There are a total of `15` characters in the above string. Thus, a total of `8 * 15 = 120` bits are required to send this string.

Using the Huffman Coding technique, we can compress the string to a smaller size.

Huffman coding is done with the help of the following steps.

**Step 1**: Calculate the frequency of each character in the string.

|  B  |  C  |  A  |  D  |
| :-: | :-: | :-: | :-: |
|  1  |  6  |  5  |  3  |

**Step 2**: Create a leaf node for each unique character and sort all nodes in increasing order of the frequency. We can store these nodes in a min heap.

|  B  |  D  |  A  |  C  |
| :-: | :-: | :-: | :-: |
|  1  |  3  |  5  |  6  |

```
[B - 1] [D - 3] [A - 5] [C - 6]
```

**Step 3**: Extract 2 nodes with the minimum frequency from the min heap, create a new internode with a frequency equal to the sum of the two nodes frequencies. Make the first extracted node as its left child and the other extracted node as its right child. Add this node to the min heap.

| \*  |  A  |  C  |
| :-: | :-: | :-: |
|  4  |  5  |  6  |

```
    [* - 4]
    /     \
[B - 1] [D - 3] [A - 5] [C - 6]
```

**Step 4**: Repeat step 2 and step 3 util the min heap contains only one node. The remaining node is the root node and the tree is complete.

|  C  | \*  |
| :-: | :-: |
|  6  |  9  |

```
         [* - 9]
        /       \
    [* - 4]      \
    /     \       \
[B - 1] [D - 3] [A - 5] [C - 6]
```

| \*  |
| :-: |
| 15  |

```
              [* - 15]
              /      \
          [C - 6]     [* - 9]
                     /       \
                 [* - 4]      \
                 /     \       \
             [B - 1] [D - 3] [A - 5]
```

**Step 5**: For each non-leaf node, assign 0 to the left edge and 1 to the right edge

```
              [* - 15]
              /      \
             0        1
            /          \
          [C - 6]     [* - 9]
                      /     \
                     0       \
                    /         \
                 [* - 4]       1
                  /    \        \
                 0      1        \
                /        \        \
             [B - 1] [D - 3] [A - 5]
```

For sending the above string over a network, we have to send the tree as well as the above compressed-code. The total size is given by the table below.

|    Character     | Frequency | Code |    Size     |
| :--------------: | :-------: | :--: | :---------: |
|        A         |     5     |  11  | 5 \* 2 = 10 |
|        B         |     1     | 100  | 1 \* 3 = 3  |
|        C         |     6     |  0   |  6 \*1 = 6  |
|        D         |     3     | 101  | 3 \* 3 = 9  |
| 4 \* 8 = 32 bits |  15 bits  |      |   28 bits   |

Without encoding, the total size of the string was `120` bits. After encoding the size is reduced to `32 + 15 + 28 = 75`.

## Decoding the code

For decoding the code, we can take the code and traverse through the tree to find the character.

Let 101 is to be decoded, we can traverse from the root as in the figure below.

```
              [* - 15]
              /      \
             0       (1)
            /          \
          [C - 6]     [* - 9]
                      /     \
                    (0)      \
                    /         \
                 [* - 4]       1
                  /    \        \
                 0     (1)       \
                /        \        \
             [B - 1] [D - 3] [A - 5]
```

101 ~ D

## Huffman Coding Complexity

The time complexity for encoding each unique character based on its frequency is O(nlog n).

Extracting minimum frequency from the priority queue takes place 2\*(n-1) times and its complexity is O(log n). Thus the overall complexity is O(nlog n).
