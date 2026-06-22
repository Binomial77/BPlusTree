# B+ Tree Database Engine

A lightweight, file-backed database engine built on a B+ Tree index structure in C++. Supports insertion, search, deletion, and tree visualization using a simple CLI interface.

---

## What is a B+ Tree?

A B+ Tree is a self-balancing tree data structure where:
- **Internal nodes** store only keys and child pointers  they guide search.
- **Leaf nodes** store all keys along with data pointers (file handles), and are linked sequentially for efficient range scans.
- All data lives at the leaf level, maintaining sorted order at all times.

This engine uses **two separate degree parameters** — one for internal nodes and one for leaf nodes  because internal nodes hold smaller tree pointers (so they fit more per disk block) while leaf nodes hold larger data pointers (so they fit fewer).

---

## Project Structure

```
BPTree/
│
├── main.cpp                  # CLI entry point
├── src/
│   ├── insert.cpp            # Insertion into leaf & internal nodes
│   ├── search.cpp            # Key lookup and file data retrieval
│   ├── display.cpp           # Level-order and sequential display
│   ├── removal.cpp           # Deletion with redistribution & merging
│   └── utils.cpp             # Node/tree lifecycle, helpers (findParent, etc.)
│
└── include/
    └── bptree/
        └── bptree.hpp        # Class declarations for Node and BPTree
```

---

## Build & Run

### Prerequisites
- A C++17 compatible compiler (e.g. `g++`)
- A `DBFiles/` directory in the same folder as the executable (for storing tuple data)

### Steps

```bash
# 1. Create the data directory
mkdir DBFiles

# 2. Compile
g++ main.cpp src/*.cpp -Iinclude -o main

# 3. Run
./main        # Linux/macOS
.\main.exe    # Windows
```

---

## Usage

On launch, you'll be prompted to set the tree's degree parameters:

```
Please provide the value to limit maximum child Internal Nodes can have: 4
And Now Limit the value to limit maximum Nodes Leaf Nodes can have: 3
```

Then choose an operation from the menu:

```
Press 1: Insertion
Press 2: Search
Press 3: Print Tree
Press 4: Delete Key In Tree
Press 5: ABORT!
```

### Insertion
Inserts a student record (roll number as key, name/age/marks as tuple data). Each record is saved as `DBFiles/<rollNo>.txt`.

```
Please provide the rollNo: 101
What's the Name, Age and Marks acquired?: Alice 20 95
```

### Search
Looks up a roll number and prints the corresponding tuple from disk.

```
What's the RollNo to Search? 101
Hurray!! Key FOUND
Corresponding Tuple Data is: Alice 20 95
```

### Print Tree
Two display modes:
- **Hierarchical**  level order view showing tree structure
- **Sequential**  linked-list traversal of all leaf nodes in sorted order

### Delete
Shows the current tree, prompts for a key, deletes the record (and its file), then re displays the tree.

---

## Core Operations

| Operation | Complexity |
|-----------|------------|
| Search    | O(log n)   |
| Insert    | O(log n)   |
| Delete    | O(log n)   |
| Range scan (seq display) | O(k) where k = number of results |

### Insertion Details
- If the target leaf has space, the key is inserted in sorted order.
- If the leaf is full, it splits: the lower half stays, the upper half moves to a new leaf, and the new leaf's minimum key is pushed up to the parent.
- Internal node splits exclude the middle key (pushed up, not copied).

### Deletion Details
- The key and its file pointer are removed from the leaf.
- If the leaf falls below the minimum fill threshold:
  - **Redistribute**: borrow a key from a sibling if it has a spare.
  - **Merge**: combine with a sibling and pull down the separator key from the parent, recursively fixing underflows upward.

---

## Data Storage

Each record is persisted as a plain text file:

```
DBFiles/
├── 101.txt    → "Alice 20 95"
├── 102.txt    → "Bob 21 88"
└── ...
```

The B+ Tree holds open `FILE*` pointers to these files, allowing direct data access from leaf nodes.

---

## Known Limitations

- Single process, in memory tree (not persisted across restarts).
- No duplicate key support.
- The `findParent` function uses global state (`Node* parent`)   not thread safe.
- File pointers are kept open during the session  a crash may leave files in an inconsistent state.

---
