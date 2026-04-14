# TLA+ Simple Concurrency Example

[![Syntax-check models](https://github.com/lucformalmethodscourse/simpleconcurrency-tla/actions/workflows/main.yml/badge.svg)](https://github.com/lucformalmethodscourse/simpleconcurrency-tla/actions/workflows/main.yml)

This example models a very basic system where N threads concurrently attempt to increment a shared variable.

## References

- Section 3 in the instructor's [book chapter on concurrency](https://arxiv.org/abs/1705.02899)
- Sections 6.1-6.3 in the [programming languages lecture notes](https://lucproglangcourse.github.io/concurrency.html)
- [Excercises 23 and 24](homes.cs.aau.dk/~kgl/esv04/exercises/#Exercise_23) from the Aarhus systems validation course

# Simple Concurrency TLA+ - Team 5

## Team Members
- Maja Swiderska
- Melanie Abarca

## Exercise #23: How Much Can We Lose?

### Problem Description
We have K threads, each performing N increments on a shared variable. 
However, each increment is NOT atomic. It consists of 3 steps:
1. READ shared value into local register
2. Increment local register by 1
3. WRITE local register back to shared

This allows lost updates when threads interleave.

### Objective
Find the smallest possible final value of the shared variable when all K threads complete their N increments.

### Key Insight
Because increments are not atomic, multiple threads can read the same value before any writes occur, causing lost updates.

### Formula Discovered

| Condition | Minimum Value |
|-----------|---------------|
| K = 1 (single thread) | N |
| K ≥ 2, N = 1 | 1 |
| K ≥ 2, N ≥ 2 | 2 |

### Experimental Results (K = 1 to 4, N = 1 to 4)

| K \ N | 1 | 2 | 3 | 4 |
|-------|---|---|---|---|
| 1 | 1 | 2 | 3 | 4 |
| 2 | 1 | 2 | 2 | 2 |
| 3 | 1 | 2 | 2 | 2 |
| 4 | 1 | 2 | 2 | 2 |

### Pattern Summary
- K = 1 row: increases by 1 each time (1, 2, 3, 4)
- K = 2, 3, 4 rows: all same pattern (1, 2, 2, 2)
- Once K ≥ 2, results are independent of K
- Once N ≥ 2, results are independent of N

### Key Findings
- Intuition "minimum = K" is WRONG
- With 2+ threads and 2+ increments, minimum is ALWAYS 2
- Adding more threads beyond 2 does not lower the minimum
- Adding more increments beyond 2 does not lower the minimum
- Maximum lost increments = (K × N) - 2

### Example
With K = 4 threads, each doing N = 4 increments:
- Maximum possible = 16
- Actual minimum found = 2
- Lost increments = 14

### Why K Doesn't Matter
Increasing the number of threads increases overlapping operations, but does not reduce the minimum further. Once K ≥ 2, the worst-case behavior is already achieved.

### Why Minimum Cannot Go Below 2 (When N ≥ 2)
- Round 1: All threads read 0, all write 1 → shared = 1
- Round 2: All threads read 1, all write 2 → shared = 2
- After round 2, shared = 2 and cannot decrease

### Testing Method
1. Set K, N, Min in `ConcurrentMultiple.cfg`
2. Start Min = K × N (maximum possible)
3. Run TLC model checker
4. If counterexample found, decrease Min by 1
5. Repeat until no counterexample
6. Largest passing Min = answer

### TLA+ Property Used
