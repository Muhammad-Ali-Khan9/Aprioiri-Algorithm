# Apriori Algorithm: A Comprehensive Guide

The **Apriori Algorithm** is a fundamental approach in data mining, widely used for **frequent itemset mining** and **association rule learning**. This document provides a detailed explanation of the algorithm, its underlying formulas, and the step-by-step process it follows to uncover hidden patterns in data.

---

## What is the Apriori Algorithm?

The Apriori algorithm is a rule-based method for identifying **frequent patterns** in transactional datasets. It works by finding frequent itemsets (groups of items that frequently appear together) and using these itemsets to generate **association rules**.

### Key Applications
- **Market Basket Analysis**: Identify products that are frequently purchased together.
- **Web Usage Mining**: Discover patterns in user browsing behavior.
- **Healthcare**: Detect correlations between symptoms and diseases.

---

## Key Concepts

### 1. Support
The **support** of an itemset is the proportion of transactions in which the itemset appears. It indicates the popularity of the itemset.

Formula:
Support(A) = Number of transactions containing A / Total number of transactions

### 2. Confidence
The **confidence** of a rule measures the likelihood of the consequent occurring given the antecedent.

Formula:
Confidence(A => B) = Support(A union B) / Support(A)

### 3. Lift
The **lift** of a rule measures how much more likely the consequent is to occur given the antecedent, compared to random chance.

Formula:
Lift(A => B) = Confidence(A => B) / Support(B)

---

## Steps of the Apriori Algorithm

### Step 1: Dataset Preprocessing
- Convert the dataset into a transactional format where each row represents a transaction and each column represents an item.
- Transactions contain only items present in them (e.g., "Bread, Milk, Eggs").

### Step 2: Candidate Generation
- Generate candidate itemsets of size k (Ck) from the frequent itemsets of size k-1 (Fk-1).
- Start with single-item candidates (C1), then expand iteratively to two-item candidates (C2), three-item candidates (C3), and so on.

### Step 3: Prune Infrequent Itemsets
- Use the Apriori Property: If a candidate itemset has an infrequent subset, it cannot be frequent.
- Eliminate candidates whose subsets do not meet the minimum support threshold.

### Step 4: Support Counting
- Scan the dataset to count the occurrences of each candidate itemset.
- Retain only those itemsets whose support meets or exceeds the minimum support threshold. These become the frequent itemsets (Fk).

### Step 5: Generate Association Rules
- From the frequent itemsets, generate association rules in the form of "If A, then B".
- Calculate metrics like support, confidence, and lift for each rule.
- Retain rules that meet the minimum confidence and lift thresholds.

### Step 6: Output Results
- Frequent itemsets and association rules are the final outputs, providing insights into the data.

---
### Pseudo Code for Apriori Algorithm ###
```plaintext
Algorithm Apriori(T, minSupport, minConfidence):
    // T: Transactions, minSupport: Minimum support, minConfidence: Minimum confidence

    // Step 1: Initialize
    C1 = Generate all single-item candidates
    F1 = Filter candidates in C1 based on minSupport

    // Step 2: Iterative Process
    k = 1
    while Fk is not empty:
        k = k + 1
        // Generate candidate itemsets of size k from Fk-1
        Ck = GenerateCandidates(Fk-1)
        
        // Prune candidates with infrequent subsets
        Ck = PruneCandidates(Ck, minSupport)

        // Count support for remaining candidates
        for each transaction t in T:
            for each candidate c in Ck:
                if c is in t:
                    increment count of c

        // Generate frequent itemsets of size k
        Fk = {c in Ck | Support(c) >= minSupport}

    // Step 3: Generate Association Rules
    Rules = []
    for each frequent itemset F in all Fk:
        for each subset S of F:
            if S is not empty and S != F:
                Confidence = Support(F) / Support(S)
                if Confidence >= minConfidence:
                    Rule = "If S then (F \\ S)"
                    Lift = Confidence / Support(F \\ S)
                    if Lift > 1:
                        Add Rule to Rules

    return Rules
```
---

### Data Structure used
# Data Structures Used in Apriori Algorithm

The **Apriori algorithm** is used for mining frequent itemsets and association rules from transactional data. Several data structures are used in the algorithm to facilitate the efficient generation and pruning of candidate itemsets. Here's an overview of the main data structures:

## 1. Transaction Database (TDB)
The **Transaction Database (TDB)** is the core input to the Apriori algorithm, consisting of a collection of transactions. Each transaction contains a set of items.

- **Data Structure:** 
  - **List of Sets** or **Binary Matrix**.
  - Each transaction is represented as a set of items, e.g., `T = {A, B, C}`.
  - In a binary matrix representation, the presence or absence of items is marked by 1s and 0s across a matrix.

## 2. Candidate Itemsets (Ck)
At each iteration, the algorithm generates candidate itemsets of size **k** (denoted as `Ck`). These are potential itemsets that are being tested for frequency.

- **Data Structure:**
  - **Hash Table** or **Dictionary**.
  - A hash table is used to store candidate itemsets and their counts (support). 
  - Example: `{(A, B): 3, (B, C): 2}` where the key is the itemset and the value is its frequency.

## 3. Frequent Itemsets (Lk)
Frequent itemsets are those that meet the minimum support threshold. At the end of each iteration, **Lk** represents the frequent itemsets of size **k**.

- **Data Structure:**
  - **List of Sets** or **Array**.
  - Stores frequent itemsets after pruning the candidate itemsets based on the support threshold.
  - Example: `L2 = {(A, B), (B, C)}` where the itemsets have support greater than or equal to the minimum support.

## 4. Hash Table (for Counting)
A **Hash Table** is used for counting the support of candidate itemsets during the scanning of the transaction database.

- **Data Structure:**
  - **Hash Table** or **Dictionary**.
  - Each candidate itemset is stored as a key, and its corresponding count is stored as the value.
  - Example: `{(A, B): 3, (B, C): 2}` where `(A, B)` has a count of 3 and `(B, C)` has a count of 2.

## 5. Trie (Prefix Tree)
A **Trie** (or prefix tree) can be used to store and generate frequent itemsets efficiently, especially when there are common prefixes in the itemsets. 

- **Data Structure:**
  - **Trie (Prefix Tree)**.
  - Each node in the tree represents a character (or item) in a string (or itemset), and the edges between nodes represent adding more items.
  - **Usage**: Tries help in pruning and efficiently storing itemsets by sharing common prefixes.
  
  Example Trie for storing itemsets:
      root
     /    \
    c      b
   / \      \
  a   r      a
 / \          \
t   r          t

- Each level of the tree corresponds to an itemset size (e.g., level 1 for single items, level 2 for pairs, etc.).

## 6. Apriori Tree
In some implementations, an **Apriori Tree** is used to generate candidate itemsets and prune them efficiently. It is a hierarchical tree structure that aids in the candidate generation process.

- **Data Structure:**
- **Tree Structure**.
- Nodes represent itemsets, and branches represent candidate itemsets generated by combining smaller frequent itemsets.
- The tree structure helps in reducing the number of candidate itemsets generated and facilitates pruning.

## 7. List of Candidate Itemsets (Ck)
During each iteration of the algorithm, candidate itemsets (Ck) are generated by joining frequent itemsets of size **k-1** to create itemsets of size **k**.

- **Data Structure:**
- **List or Array**.
- Stores candidate itemsets generated by joining frequent itemsets from the previous iteration.

Example:
L1 = {A, B, C} # Frequent 1-itemsets C2 = { (A, B), (A, C), (B, C) } # Candidate 2-itemsets


## 8. Binary Matrix for Transaction Representation
In some implementations, a **binary matrix** is used to represent transactions. The rows correspond to transactions, and columns represent items. A `1` indicates that an item is present in the transaction, and a `0` indicates that it is absent.

- **Data Structure:**
- **Binary Matrix**.
- Each row represents a transaction, and each column represents an item in the dataset.
- Example:
  ```
  T1: 1 1 0 0 1
  T2: 0 1 1 1 0
  T3: 1 0 1 0 0
  ```

---

## Summary of Data Structures in Apriori

| **Step**                     | **Data Structure**                     | **Description**                                        |
|------------------------------|----------------------------------------|--------------------------------------------------------|
| **Transaction Database (TDB)** | List of Sets / Binary Matrix           | Stores the transactions (input data).                  |
| **Candidate Itemsets (Ck)**    | Hash Table / Dictionary                | Stores candidate itemsets and their support counts.    |
| **Frequent Itemsets (Lk)**     | List of Sets / Array                   | Stores frequent itemsets after pruning.                |
| **Counting Itemsets**          | Hash Table / Dictionary                | Used for counting the support of candidate itemsets.   |
| **Trie (Prefix Tree)**         | Trie                                   | Used for efficiently storing and generating itemsets.  |
| **Apriori Tree**               | Tree                                   | Used for generating candidate itemsets in some variations. |
| **Candidate Itemsets List**    | List / Array                           | Stores candidate itemsets generated in each iteration. |

---

## Complexity Analysis

### Space Complexity
1. **Candidate Itemsets**: The algorithm stores candidate itemsets (Ck) and frequent itemsets (Fk) during each iteration. The worst-case scenario occurs when all possible subsets of items are frequent.
   - Maximum possible itemsets: 2^n, where n is the number of items in the dataset.

2. **Dataset Representation**: Memory is required to store the entire dataset and transaction data during multiple scans.

Overall Space Complexity: O(2^n)

### Time Complexity
1. **Generating Candidate Itemsets**: The number of candidate itemsets grows exponentially with the number of items, making this step computationally expensive in large datasets.
   - Complexity: O(2^n), where n is the number of items.

2. **Support Counting**: Requires scanning the dataset multiple times to count the occurrences of candidate itemsets.
   - Complexity: O(k * t), where k is the total number of candidate itemsets and t is the total number of transactions.

Overall Time Complexity: O(2^n * t), where n is the number of items and t is the number of transactions.

---

## Example Walkthrough

1. **Input Dataset**:
   Transactions:
   - Transaction 1: Bread, Milk
   - Transaction 2: Bread, Diaper, Beer, Eggs
   - Transaction 3: Milk, Diaper, Beer, Coke
   - Transaction 4: Bread, Milk, Diaper, Beer
   - Transaction 5: Bread, Milk, Diaper, Coke

2. **Generate Candidate Itemsets (C1)**:
   Items: Bread, Milk, Diaper, Beer, Eggs, Coke
   Support is calculated for each item.

3. **Prune Infrequent Items**:
   Retain items meeting the minimum support threshold to create F1 (frequent 1-itemsets).

4. **Generate Candidate Itemsets (C2)**:
   Combine F1 itemsets to generate two-item candidates (e.g., Bread and Milk).

5. **Prune and Count Support**:
   Calculate support for each 2-item candidate. Retain those meeting the threshold to form F2.

6. **Repeat**:
   Generate larger itemsets (C3, C4, etc.) until no more frequent itemsets can be generated.

7. **Generate Rules**:
   For example:
   - Rule: If Bread and Milk, then Diaper
     - Support: 40 percent
     - Confidence: 80 percent
     - Lift: 1.2

---

## Strengths of Apriori
- Easy to understand and implement.
- Works well for small-to-medium-sized datasets.

---

## Limitations of Apriori
- Computationally expensive for large datasets due to multiple scans.
- Many candidate itemsets are generated, leading to high memory usage.

---

## Learn More
Explore the details of the algorithm and its implementation in **Apriori-analysis-learning.ipynb**, available in this repository.
