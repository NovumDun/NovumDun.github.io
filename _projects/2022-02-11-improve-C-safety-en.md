---
title: 'C language security improvement'
date: 2022-10-01
permalink: /posts/2022/02/improve-C-safety-en/
excerpt: 'Some suggestions for improving C language security.'
tags:
  - C
  - Array out of bounds
---

# Introduction

Array out-of-bounds, wild pointers, and memory leaks in C language have always been troublesome problems. Of course, these problems can be reduced by good programming and coding habits, but they are difficult to completely solve. Therefore, the safe language Rust appeared.

# Array out of bounds

The reason why the C language array is out of bounds is that the index of the array and pointer array exceeds the index range.

```c
  // Scenario 1
  int arr[4];
  int a = arr[4];

  // Scenario 2
  int b = rand();
  int c = arr[b];

  // Scenario 3
  int *p = arr;
  int d = p[4];
```
* Scenario 1

In order to ensure the access safety of the array, the compiler can now make some judgments at compile time, such as "  int a = arr[4]; ". This error can be found because the compiler knows the index range of arr. If the value of the index is known, the validity of the index value can be judged at compile time.

* Scenario 2

But when the index is a variable and the value of this variable is unknown, the compiler can't catch this error, like "  int c = arr[b];". Resolving this error requires checking the index at runtime. The current general approach in C language is to manually add code to determine the value of the array index before accessing the array, but this is a tedious process. The process of checking the index at runtime can be achieved by adding code automatically by the compiler, so the compiler needs to implement this feature. This is also how many languages handle array out of bounds.
* Scenario 3

Finally, in the case of an array of pointers, the compiler has no way of knowing the valid range of the index, so there is no way for the compiler to generate code that checks the index at runtime. To enable the compiler to generate code that checks the index at runtime, we need to make changes to the pointer. In the C language, the length of the pointer is the bit width of the operating system, and the pointer only saves the address of the memory. Now we need to change it to 2 times the operating system bit width, leaving an extra space for the operating system bit width to store the array index range.

```
                                ┌──────────────────┐
int *p = 0x22446688;       ┌────┤  memory address  │
                           │    └──────────────────┘
                           │
                           │    ┌──────────────────┬──────────────────┐
int *p = 0x22446688 @ 4;   └───►│  memory address  │   array length   │
                                └──────────────────┴──────────────────┘
```

When assigning a pointer, use the format "int *p = 0x22446688 @ 4;", 0x22446688 is the address of the memory pointed to by the pointer, and 4 is the range of the pointer array index, that is, the length of the pointer array. This allows the compiler to know the index range of the pointer array and generate code that can check the index at runtime.

## Regulation

* Getting the value of the pointer P directly returns the address of the memory pointed to by the pointer.

```c
  int *p = 0x22446688 @ 4;
  unsigned int addr = p;  // addr -> 0x22446688
```

* If the pointer p is not assigned a value when it is defined, the compiler should initialize the memory address held by p to 0 and the array index range to 0.

```c
  int *p;
  unsigned int addr = p;  // addr -> 0x00000000
```

* &p returns the memory address of pointer p.

```c
  int *p;                   // assumption p is at memory 0x12345678
  unsigned int addr = &p;   // addr -> 0x12345678
  int **pp = &p @ 1;        // pp -> 0x12345678 @ 1
```

* To get the array length, add a keyword rangeof() like sizeof().

```c
  int *p;
  unsigned int len = rangeof(p);  // len -> 0
  p = 0x22446688 @ 4;
  len = rangeof(p);               // len -> 4
```

## Check index at runtime

Checking indexes at runtime may sacrifice some performance, so this feature can be a compile option that can be turned on or off globally or locally in the code. For example, it is turned on in the code debugging and preview stages, and turned off in the official stage. It is always turned off in the code part with high performance requirements and can guarantee security, and it is always turned on in the code part where security is the highest priority, so as to achieve both performance and security. Users can configure according to their own needs.

# Wild pointers, memory leaks

Wild pointers are where the pointer points to is unknowable (random, incorrect, not explicitly limited).

The memory of malloc is not reclaimed (is permanently occupied), which causes a memory leak.

```c
  // Scenario 1
  int *p;
  p[4] = 0x00000000;

  // Scenario 2
  int *p1 = malloc(sizeof(int) * 0x10);
  free(p1);
  p1[8] = 0x00000000;

  // Scenario 3
  int *p2 = malloc(sizeof(int) * 0x10);
  int *p3 = p2;
  free(p2);
  p3[8] = 0x00000000;

  // Scenario 4
  int *p1 = malloc(sizeof(int) * 0x10);
  int *p1 = malloc(sizeof(int) * 0x10);
```

The discussion below builds on the previous discussion of changing the pointer type.

* Scenario 1

If the pointer p is not assigned a value when it is defined, the compiler should initialize the memory address held by p to 0 and the array index range to 0. If the compiler knows that the pointer memory address is 0, the compiler directly determines that it is an error.

* Scenario 2

Write a new free function, the function "void free(void *__ptr)" is changed to "void free(void **__ptr)" , and the parameter becomes the address of the pointer holding the address of the memory that should be freed. In the free function, in addition to releasing the memory saved by the pointer, the memory address saved by the pointer is also set to 0, and the index range of the array is 0.

* Scenario 3

This requires good design to avoid.

* Scenario 4

This also requires good design to avoid.

One way to solve part of the problem is to implement memory allocation logging in the memory management system. When malloc is called, malloc gets the program address where it was called, and when it was called, and records it along with the address of the allocated memory, which deletes the record when free. The user can judge whether a memory leak may have occurred through the duration of the record of the memory management system, and then check the calling code in combination with the recorded program address. This method also sacrifices performance, so it can be used in the debugging and preview stages.


2022-10-02  
Deng Bo
