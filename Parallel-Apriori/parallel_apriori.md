# Parallel Apriori Using CUDA

Below is a detailed plain-text description of how Apriori can be parallelized using NVIDIA CUDA, including space and time complexities, and a comparison with classic Apriori in terms of speed.

## 1 Introduction to Apriori

Apriori is a level-wise algorithm for discovering frequent itemsets in a transactional dataset. A transaction is a list of items, and an itemset is any subset of these items. An itemset is considered frequent if its support count is at least a user-defined threshold minSupport.

Core steps of the Apriori process:

1 Generate candidates  
   For each pass k, produce k-item candidates from the frequent k minus 1 itemsets.

2 Count support  
   Determine how many transactions contain each candidate.

3 Prune  
   Eliminate candidates whose support is below minSupport.

This process starts with k equals 1 for single items and proceeds to larger itemsets until no more frequent sets can be formed.

## 2 Why Parallelize Apriori with CUDA

The support counting step, in which each candidate is matched against all transactions, can be very time-consuming for large datasets. By using CUDA, we can distribute these membership checks across numerous GPU threads, potentially achieving significant speedups compared to a single-threaded CPU approach.

## 3 High-Level CUDA Apriori Flow

1 Read and flatten transactions  
   We store all items from the dataset in a single array. Each transaction has an offset indicating where it starts and a length indicating its number of items. This lets each GPU thread quickly locate the items belonging to its transaction.

2 Generate initial candidates for k equals 1  
   Collect all distinct items and treat each one as a candidate itemset of size one.

3 CUDA support counting  
   Flatten these candidate itemsets and transfer both them and the transactions to GPU memory. A kernel is launched so that each thread processes one transaction or a small set of transactions, checking membership for all candidates. An atomic increment is used to accumulate support counts safely.

4 Filter by minSupport  
   After the kernel completes, support counts are copied back to the CPU. Candidates with support below the threshold are removed.

5 Generate next-level candidates  
   Merge frequent itemsets of size k minus 1 to form candidate itemsets of size k. This may be done by joining pairs of itemsets that share their first k minus 2 items.

6 Repeat  
   Continue the cycle by flattening the new k-item candidates, offloading them to the GPU for counting, and filtering again. Stop when no further frequent itemsets can be generated or no more candidates exist.

## 4 Data Structures for the GPU

Transactions are stored in a single array, along with offset and length arrays. Candidate itemsets are also stored in a flattened array, with offset and length arrays. This arrangement lets the kernel iterate over each itemset in a consistent way. 

## 5 CUDA Kernel for Counting Support

A common scheme is to map each transaction to one GPU thread:

1 The thread finds its assigned transaction using the global index.  
2 For each candidate, the thread checks if all of that candidate's items are present in the transaction.  
3 If the transaction contains the candidate, the thread performs an atomic increment on that candidate's counter.

These counters are stored in GPU global memory, where threads from all blocks can safely update them with atomic operations.

## 6 Controlling GPU Execution

The CPU selects how many threads per block to use, such as 128 or 256, and how many blocks are needed to cover all transactions. If T is the total number of transactions and we want one thread per transaction, the number of blocks may be T divided by threadsPerBlock, rounded up as necessary. Each thread checks its global index and proceeds only if it falls within T.

## 7 Filtering and Next Pass

After the kernel completes, the CPU retrieves the support counts from the GPU. Any candidate whose count is below minSupport is discarded. The surviving sets are frequent and serve as a foundation for generating the next pass of candidates. The process continues until no further itemsets meet minSupport.

## 8 Space Complexity

Let N be the total number of transactions and M be the total number of unique items. The flattened transaction array typically requires space on the order of the total number of items across all transactions. If the dataset has large or dense transactions, this can be significant. 

The candidate storage space depends on the number of candidates at each pass. Denote C as the sum of candidate sets across passes, multiplied by the size of each itemset. In worst-case scenarios or for very low minSupport, C can become large.

On the GPU, we need space for these arrays:
1 Transactions array, size roughly sum of all transaction lengths  
2 Offsets and lengths arrays, size on the order of N  
3 Candidates array, size on the order of C times average itemset size  
4 Offsets for candidates, size on the order of C

Thus, the space complexity is primarily O N plus the total size of the transaction dataset plus O C for the candidate data structures. Classic Apriori has a similar space requirement, since it must store transactions and candidates in memory, but the difference is that GPU memory may be smaller and may require chunking to handle huge datasets.

## 9 Time Complexity and Speed Comparison

### 9.1 Classical Apriori Time Complexity

For each pass, if there are Ck candidates at level k and N transactions, a naive membership check can take O Ck times N operations. As k increases, the total cost is multiplied across multiple passes, summing over k. Thus, classical Apriori can be time-consuming if Ck grows large or N is big.

### 9.2 Parallel Apriori Time Complexity

Parallel Apriori still processes the same total number of membership checks in the worst case, so the theoretical complexity remains O Ck times N across all passes. However, distributing these checks across many GPU threads can drastically reduce the actual runtime. Each transaction or subset of transactions is processed concurrently by different threads, making the overall wall-clock time potentially much smaller.

### 9.3 Speed Gains

In practice, CUDA-based Apriori often speeds up the most time-consuming step, membership checking, by a factor that depends on the number of GPU cores and the efficiency of memory access. While the overall complexity remains the same in notation, the parallel execution can lead to a significant performance gain over single-threaded or moderately parallel CPU implementations.

Hence, both classical and parallel Apriori share an order of C times N time cost per pass, but parallel Apriori reduces wall-clock time by exploiting GPU concurrency. This makes it more scalable for large datasets where candidate generation and membership checks would otherwise be prohibitively slow.

## 10 Association Rules optional

After frequent itemsets are discovered, association rules of the form A implies B can be extracted. Typically, the CPU enumerates subsets of each frequent itemset and checks confidence or other metrics. CUDA is less commonly used for this rule generation step, because enumerating subsets is more irregular and is not as well-suited to highly parallel operations as membership checks.

## 11 Summary

Parallel Apriori with CUDA follows the same candidate generation, support counting, and pruning cycle as classical Apriori. The major difference lies in using GPU threads to parallelize membership checks. 

Space complexity involves storing both the transactions and candidate sets, which can be large, and chunk-based processing might be required for very large datasets. 

Time complexity remains functionally equivalent to classical Apriori, but actual running time is typically much faster due to GPU parallelism, reducing the bottleneck of support counting. This speed advantage makes CUDA-based Apriori an attractive option for mining large transactional datasets within practical runtimes.