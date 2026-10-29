<!---
{
  "id": "e2ab3230-2ec2-4a65-a2e6-57db1908770f",
  "teaches": "Cpp: Dynamic Linked Containers with std::list",
  "depends_on": [
    "6a5c3351-73bd-4cff-8119-01db07dbb801",
    "c32afdd3-48c3-4ff2-b5f7-a2179a2f8093"
  ],
  "author": "Stephan Bökelmann",
  "first_used": "2026-10-29",
  "keywords": [
    "C++",
    "std::list",
    "linked list",
    "container",
    "iterator",
    "splice",
    "g++"
  ]
}
--->

# Cpp: Dynamic Linked Containers with std::list

> In this exercise you use the linked list from the standard library. You compare its memory layout with a vector, insert and erase in the middle, and find out why there is no `list[i]` and why `std::sort` does not accept a list. No `sudo` is required.

## Introduction

`std::list<T>` is a doubly linked list: every element lives in its own node on the heap, and every node knows its predecessor and its successor. You have written such a structure with pointers in C. The standard version is typed, cleans up after itself and works with iterators.

The price of the layout is that there is no way to jump to the n-th element. The gain is that inserting or erasing at a known position changes only two pointers, and that no element ever moves in memory.

### 1.1) Further Readings and Other Sources

* [cppreference: std::list](https://en.cppreference.com/w/cpp/container/list)
* [cppreference: std::list::splice](https://en.cppreference.com/w/cpp/container/list/splice)

## Tasks

### Task 1: Fill and print

```bash
mkdir -p ~/cpp-list && cd ~/cpp-list
```

```c++
// file: basic.cpp
#include <iostream>
#include <list>

int main() {
    std::list<int> linked_list{};
    linked_list.push_back(1);
    linked_list.push_back(20);
    linked_list.push_front(0);

    for (auto &x : linked_list)
        std::cout << x << std::endl;
    std::cout << "size: " << linked_list.size() << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra basic.cpp -o basic && ./basic
```

`push_front` is cheap for a list. Try `linked_list[1]` and read the compiler error: the operator does not exist.

### Task 2: Nodes are scattered

```c++
// file: layout.cpp
#include <iostream>
#include <list>
#include <vector>

int main() {
    std::vector<int> v{1, 2, 3, 4};
    std::list<int>   l{1, 2, 3, 4};

    std::cout << "vector elements:" << std::endl;
    for (auto &x : v) std::cout << "  " << &x << std::endl;
    std::cout << "list elements:" << std::endl;
    for (auto &x : l) std::cout << "  " << &x << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra layout.cpp -o layout && ./layout
```

The vector addresses are exactly `sizeof(int)` apart. The list addresses are further apart, with the size of a node (two pointers plus the value) in between. This is the cost of the list: every access is a pointer chase, and the cache cannot help.

### Task 3: Insert and erase at an iterator

```c++
// file: edit.cpp
#include <iostream>
#include <iterator>
#include <list>

int main() {
    std::list<int> l{10, 20, 30, 40};

    auto it = l.begin();
    std::advance(it, 2);                 // it points to 30
    std::cout << "it points to " << *it << std::endl;

    l.insert(it, 25);                    // insert BEFORE 30
    it = l.erase(it);                    // erase 30, it now points to 40

    for (int x : l) std::cout << x << " ";
    std::cout << std::endl << "it still valid: " << *it << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra edit.cpp -o edit && ./edit
```

`std::advance` has to walk two nodes, it cannot jump. After the `insert` the iterator `it` is still valid and points to the same element. The result of `erase` is the iterator to the element after the removed one.

### Task 4: References stay valid

```c++
// file: stable.cpp
#include <iostream>
#include <list>

int main() {
    std::list<int> l{1, 2, 3};
    int *p = &l.front();
    std::cout << "p points to " << *p << " at " << p << std::endl;
    for (int i = 0; i < 100; ++i) l.push_back(i);
    std::cout << "front is now " << l.front() << " at " << &l.front() << std::endl;
    std::cout << "p still points to " << *p << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra stable.cpp -o stable && ./stable
```

Compare with the dangling pointer in the vector exercise: because no element ever moves, pointers and iterators into a list stay valid until that exact element is erased.

### Task 5: No random access, no `std::sort`

```c++
// file: sorting.cpp
#include <algorithm>
#include <iostream>
#include <list>

int main() {
    std::list<int> l{5, 2, 9, 1};
    // std::sort(l.begin(), l.end());
    l.sort();
    for (int x : l) std::cout << x << " ";
    std::cout << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra sorting.cpp -o sorting && ./sorting
```

Remove the comment in front of `std::sort` and read the error: the algorithm needs random-access iterators that can jump with `it + n`. A list has only bidirectional iterators, so it ships its own `sort()` member.

### Task 6: Move nodes with `splice`

```c++
// file: splice.cpp
#include <iostream>
#include <list>

int main() {
    std::list<int> a{1, 2, 3};
    std::list<int> b{10, 20};

    a.splice(a.end(), b);               // move all nodes of b to the end of a

    for (int x : a) std::cout << x << " ";
    std::cout << "| b has " << b.size() << " elements" << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra splice.cpp -o splice && ./splice
```

No element was copied or constructed, only pointers were rewired. This kind of operation is the real reason for using a list.

## Questions

- Why does `std::list` have no `operator[]`?
- What is the layout of the memory of a list compared to a vector, and what does it do to the speed of a loop over all elements?
- Why does `insert` on a list not invalidate other iterators?
- Why can you not call `std::sort` on a list, and what do you call instead?
- When is a list the better choice than a vector?

## Advice

In practice a `std::vector` is faster than a `std::list` for most workloads, even with many insertions in the middle, because of the cache. Choose the list if you need stable addresses of elements, cheap splicing or very frequent insertion and removal at positions you already hold an iterator to.
