# CSCI161-CH09-EXAMPLES
Examples of Chapter 9 from Data Structures and Algorithms in Java

---

## EX09_02 — [PriorityQueue.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/EX09_02/PriorityQueue.java)

The **priority queue ADT interface**, the contract every other fragment in this repository implements. Unlike a regular queue (FIFO), a priority queue removes the entry with the smallest key, regardless of insertion order.

Five methods define the ADT:
- `size()` and `isEmpty()` — basic collection queries.
- `insert(key, value)` — adds a key-value pair and returns the `Entry` created.
- `min()` — returns (without removing) an entry with the minimal key.
- `removeMin()` — removes and returns the entry with the minimal key.

There is no code to run here — this file only defines behavior that the later folders implement. One design decision worth discussing in class: the interface returns `null` from `min()` and `removeMin()` on an empty queue rather than throwing an exception.

---

## EX09_03 — [StringLengthComparitor.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/EX09_03/StringLengthComparitor.java)

A first look at **how key ordering is customized**. The class implements Java's `Comparator<String>` interface, ordering strings by length instead of alphabetical order: `compare(a, b)` returns -1, 0, or 1 depending on whether `a` is shorter, equal in length, or longer.

This fragment is self-contained and runnable. Its `main` method sorts an array of fruit names with `Arrays.sort` using this comparator and prints them shortest-first, so students can see comparator-driven ordering before any priority-queue machinery appears. Note the filename uses the spelling `StringLengthComparitor` while the class inside is `StringLengthComparator` — a mismatch that matters when compiling (see the build notes below).

---

## EX09_04 — [DefaultComparitor.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/EX09_04/DefaultComparitor.java)

A **general-purpose comparator** used by every implementation in this repository. `DefaultComparator<E>` orders elements using their own natural ordering: it casts an element to `Comparable<E>` and calls `compareTo`. If an element is not `Comparable`, the cast throws a `ClassCastException`.

This is the comparator a priority queue uses when constructed with its no-argument constructor — for example, when keys are `Integer` or `String`, whose `compareTo` defines their natural order. Custom comparators like the one in EX09_03 take its place when passed to a constructor. This file declares `package Extras` (like `Entry.java`), even though it sits in the EX09_04 folder.

---

## EX09_05 — [AbstractPriorityQueue.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/EX09_05/AbstractPriorityQueue.java)

The **abstract base class** that all concrete implementations extend. It is the design-pattern example of the chapter: shared code lives here so each subclass only supplies a storage strategy. It provides three things:

1. **A nested `PQEntry` class** — the concrete `Entry<K,V>` implementation that stores a key and value. It implements `getKey()` and `getValue()`, and the protected `setKey()` and `setValue()` mutators (used later by the adaptable queue) are intentionally not part of the public `Entry` interface, so outside code cannot modify keys.
2. **Comparator support** — a private `Comparator<K>` field, a constructor that accepts a comparator, and a default constructor that falls back to `DefaultComparator` for natural ordering. The protected `compare(a, b)` helper delegates to the comparator to order two entries by key.
3. **A concrete `isEmpty()`** built on the abstract `size()` method, so subclasses only implement `size()`.

It also supplies the `createEntry` factory method subclasses use to build entries — a detail the adaptable heap exploits.

---

## EX09_06 — [UnsortedPriorityQueue.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/EX09_06/UnsortedPriorityQueue.java)

The **first concrete implementation**: entries are kept in a `LinkedPositionalList` in whatever order they arrive. `insert` simply calls `addLast`, which makes insertion fast.

The interesting method is the private `findMin()`, which walks the entire list comparing keys to locate the minimum entry. Both `min()` and `removeMin()` call it, so every removal pays the full scan.

Complexity for n entries: `insert` is O(1); `min` and `removeMin` are O(n). This is the classic fast-insert / slow-remove trade-off, and the reason the sorted version exists.

---

## EX09_07 — [SortedPriorityQueue.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/EX09_07/SortedPriorityQueue.java)

The **mirror-image trade-off**: the same `LinkedPositionalList` storage, but entries are kept sorted by key at all times. `insert` walks backward from the end of the list (`list.last()`, then `list.before(...)`) until it finds the right spot, then inserts with `addAfter` — or `addFirst` if the new key is the smallest.

Because the list is always sorted, `min()` returns the element at `list.first()` and `removeMin()` removes that first position — both constant-time list operations.

Complexity for n entries: `insert` is O(n) in the worst case; `min` and `removeMin` are O(1). Comparing this folder with EX09_06 is the heart of Section 9.2: a list lets you make insertion cheap or removal cheap, but not both.

---

## EX09_08_09_10 — [HeapPriorityQueue.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/EX09_08_09_10/HeapPriorityQueue.java)

The **heap implementation**, the chapter's centerpiece and the fastest general-purpose version here. A binary heap stores entries in an `ArrayList<Entry<K,V>>` and reads the tree structure out of the array indices: `parent(j) = (j-1)/2`, `left(j) = 2j+1`, `right(j) = 2j+2`. The heap property (every parent's key is at most its children's keys) puts the minimum entry at index 0.

Key methods:
- `upheap(j)` — after an insert at the end, swaps the new entry upward with its parent until the heap property holds.
- `downheap(j)` — after a removal, swaps the displaced entry downward toward its smaller child until the property holds.
- `insert` — appends the new entry and upheaps it; `removeMin` — saves the root, swaps the last entry to the root, removes it, then downheaps the new root.
- `heapify()` — bottom-up heap construction: downheap every internal node starting from the parent of the last entry, building a heap from n entries in O(n) rather than O(n log n). The `HeapPriorityQueue(keys, values)` constructor demonstrates this.
- `sanityCheck()` — a private debugging helper that scans for heap-property violations.

Complexity for n entries: `min` is O(1); `insert` and `removeMin` are O(log n), with insertion O(log n) amortized because the underlying `ArrayList` occasionally resizes. This is the balanced trade-off neither list version achieves.

---

## EX09_11 — [PQSort.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/EX09_11/PQSort.java)

**Priority-queue sorting**: any empty priority queue plus two loops is a sorting algorithm. The static `pqSort(S, P)` method takes a `PositionalList<E>` to sort and a `PriorityQueue<E,?>` that must be initially empty, and works in two phases:

1. Remove every element from S and insert each into P, using the element itself as the key (the value is `null`).
2. Repeatedly `removeMin()` from P and `addLast` the result back into S.

The output order depends on P's comparator; the implementation determines the strategy and cost. With the unsorted list the two-phase pattern behaves like selection-sort (O(n²) worst case), with the sorted list like insertion-sort (O(n²) worst case), and with the heap it gives heap-based sorting in O(n log n) — all from the same client code. Note the file declares `package com.zybooks.dsaj.pq`, the only file in the repository outside the default package and `Extras`, which affects how it must be compiled (see below).

---

## EX09_12 — [AdaptablePriorityQueue.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/EX09_12/AdaptablePriorityQueue.java) and [HeapAdaptablePriorityQueue.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/EX09_12/HeapAdaptablePriorityQueue.java)

The **adaptable extension**, two files that add update operations to the ADT:

`AdaptablePriorityQueue.java` is the interface: it extends `PriorityQueue` with three methods that take an `Entry` previously returned by `insert` — `remove(entry)`, `replaceKey(entry, key)`, and `replaceValue(entry, value)` — each throwing `IllegalArgumentException` for a stale or foreign entry. Ordinary priority queues already hand back entries from `insert`, `min`, and `removeMin`; what adaptability adds is the ability to act on those entries later: removing an arbitrary entry or updating its key.

`HeapAdaptablePriorityQueue.java` implements the interface by extending `HeapPriorityQueue`, and its trick is **location-aware entries**. The nested `AdaptablePQEntry` class adds an `index` field recording where the entry sits in the heap array; `createEntry` is overridden to build these, and `swap` is overridden to keep each entry's index current. That makes `validate(entry)` able to confirm an entry really belongs to this queue and locate it in O(1).

With the entry's position known, `remove` swaps it to the end, deletes it, and repairs with `bubble`; `replaceKey` sets the new key and calls `bubble`, which moves the entry up or down depending on whether the key decreased or increased; `replaceValue` needs no repair since values never affect ordering. Costs: `remove` and `replaceKey` are O(log n), `replaceValue` is O(1).

---

## Extras — supporting classes

These files are not Chapter 9 fragments but the machinery the priority-queue code reuses:
- [Entry.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/Extras/Entry.java) — the key-value pair interface used by every priority queue here (declared in `package Extras`).
- [Position.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/Extras/Position.java) and [PositionalList.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/Extras/PositionalList.java) — the position abstraction and positional-list ADT carried over from Chapter 7.
- [LinkedPositionalList.java](https://github.com/WillistonStateCSCI/CSCI161-CH09-EXAMPLES/blob/main/Extras/LinkedPositionalList.java) — the doubly linked list implementation of that ADT; it is what `UnsortedPriorityQueue` and `SortedPriorityQueue` actually store their entries in.