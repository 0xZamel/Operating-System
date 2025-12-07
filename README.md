FOS OS – Memory Management Implementation

This repository contains my implementation of the Kernel Heap (kheap) and User Heap (uheap) for the FOS’25 educational operating system.
These modules provide the OS with full dynamic memory allocation capabilities in both kernel space and user space.

🔥 Overview

I implemented two major memory components:

🟦 1. Kernel Heap (Group Module)

kmalloc() / kfree()

Block allocator (small objects)

Page allocator (large objects)

🟩 2. User Heap (Individual Module)

malloc() / free() (user-side)

allocate_user_mem() / free_user_mem() (kernel-side)

Virtual allocation via system calls

Real allocations performed lazily via page faults

✨ Memory Architecture
┌──────────────────────────────────────┐
│              KERNEL HEAP             │
│  [Block Allocator]   [Page Allocator]│
│       kmalloc/kfree   (4KB pages)    │
└──────────────────────────────────────┘

┌──────────────────────────────────────┐
│               USER HEAP              │
│  [Block Allocator]   [Page Allocator]│
│ malloc/free (user)   VA reservation  │
│     sys_allocate_user_mem()          │
│     sys_free_user_mem()              │
└──────────────────────────────────────┘

🟦 Kernel Heap Implementation

The kernel heap supports dynamic memory allocation for kernel modules and system structures.

✔ Block Allocator (small allocations ≤ MAX)

Power-of-two buckets

Free-block lists

Page-based splitting

O(1) allocation best-case

Minimal fragmentation

Used whenever the requested size is small.

✔ Page Allocator (large allocations > MAX)

Implements a CUSTOM FIT strategy:

Exact fit

Worst fit

Extend heap break

Fail gracefully

All pages are allocated & mapped using OS paging routines.

✔ kfree()

Automatically merges adjacent free segments

Shrinks heap break when the last region is freed

🟩 User Heap Implementation (My Individual Module)

User processes need a safe way to allocate/free memory without having kernel privileges.
My work implements that interface cleanly.

🧠 How malloc() works
If size ≤ MAX → Block Allocator

Entirely user-space using dynamic allocator logic (no system calls).

If size > MAX → Page Allocator

Steps:

Search available regions using custom fit

Reserve VA space

Call:

sys_allocate_user_mem(va, size);


🟢 No RAM is allocated at this point
Physical memory is allocated only when a page fault occurs and the page fault handler calls placement logic.

🧠 How free() works
If pointer in block region

→ Freed via user block allocator.

If pointer in page region

Determine allocation size

Remove segment from allocator metadata

Call:

sys_free_user_mem(va, size);

🟦 Kernel Side Logic (Triggered by system calls)
✔ allocate_user_mem(e, va, size)

Marks pages with PERM_UHPAGE

Creates page tables if necessary

Reserves VA space (no frames allocated yet)

✔ free_user_mem(e, va, size)

Unmarks user heap pages

Frees frames only if they exist in memory

Removes them from page file

Shrinks heap break if possible

This ensures correctness and perfect integration with the page fault system.

🧪 Testing & Validation

I validated my implementation through:

Small block allocations

Multi-page allocations

Free + merge scenarios

Break extension & shrinking

Page fault–triggered lazy allocation

Syscall correctness (sys_allocate_user_mem, sys_free_user_mem)

Kernel/user allocator consistency

📁 Important Files
kern/mem/kheap.c                → kmalloc, kfree implementations
kern/mem/dynamic_allocator.c     → shared block allocator logic
kern/mem/chunk_operations.c      → allocate_user_mem, free_user_mem
lib/uheap.c                      → user-side malloc/free
inc/uheap.h                      → heap boundaries & constants

👤 Author

Omar Zamel
Kernel Heap & User Heap Developer – FOS’25
Ain Shams University, Computer Science
