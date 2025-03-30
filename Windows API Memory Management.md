### Windows API Memory Management

Windows provides several API functions for managing memory, categorized into different types of memory allocation and management techniques. Below are some of the key memory management functions and their uses:

#### 1. **Heap Memory Management**
- **HeapAlloc/HeapFree**: Allocate and free memory blocks from a heap. This is useful for dynamic memory allocation.
  ```cpp
  // Allocate 1024 bytes of memory from the heap
  LPVOID lpMem = HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, 1024);
  if (lpMem != NULL) {
      // Use the allocated memory...
      memcpy(lpMem, "Hello, World!", 13);
      // Free the allocated memory
      HeapFree(GetProcessHeap(), 0, lpMem);
  }
  ```

- **HeapCreate/HeapDestroy**: Create and destroy a private heap.
  ```cpp
  HANDLE hHeap = HeapCreate(0, 0, 0); // Create a private heap
  if (hHeap != NULL) {
      LPVOID lpMem = HeapAlloc(hHeap, HEAP_ZERO_MEMORY, 1024); // Allocate memory from the private heap
      if (lpMem != NULL) {
          // Use the allocated memory...
          HeapFree(hHeap, 0, lpMem); // Free the allocated memory
      }
      HeapDestroy(hHeap); // Destroy the private heap
  }
  ```

#### 2. **Virtual Memory Management**
- **VirtualAlloc/VirtualFree**: Reserve or commit pages in the virtual address space of the calling process.
  ```cpp
  // Reserve and commit 1024 bytes of memory
  LPVOID lpMem = VirtualAlloc(NULL, 1024, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
  if (lpMem != NULL) {
      // Use the allocated memory...
      memcpy(lpMem, "Hello, World!", 13);
      // Free the allocated memory
      VirtualFree(lpMem, 0, MEM_RELEASE);
  }
  ```

- **VirtualProtect**: Change the protection on a region of committed pages.
  ```cpp
  DWORD oldProtect;
  VirtualProtect(lpMem, 1024, PAGE_READONLY, &oldProtect); // Change the protection to read-only
  ```

#### 3. **Global and Local Memory Management**
- **GlobalAlloc/GlobalFree**: Allocate and free global memory blocks.
  ```cpp
  HGLOBAL hGlobal = GlobalAlloc(GMEM_MOVEABLE, 1024);
  if (hGlobal != NULL) {
      LPVOID lpMem = GlobalLock(hGlobal); // Lock the memory to get a pointer
      if (lpMem != NULL) {
          // Use the allocated memory...
          GlobalUnlock(hGlobal); // Unlock the memory
      }
      GlobalFree(hGlobal); // Free the global memory block
  }
  ```

- **LocalAlloc/LocalFree**: Allocate and free local memory blocks.
  ```cpp
  HLOCAL hLocal = LocalAlloc(LHND, 1024);
  if (hLocal != NULL) {
      LPVOID lpMem = LocalLock(hLocal); // Lock the memory to get a pointer
      if (lpMem != NULL) {
          // Use the allocated memory...
          LocalUnlock(hLocal); // Unlock the memory
      }
      LocalFree(hLocal); // Free the local memory block
  }
  ```

#### 4. **File Mapping (Memory-Mapped Files)**
- **CreateFileMapping/MapViewOfFile/UnmapViewOfFile**: Create a file mapping object, map a view of a file into the address space, and unmap the view.
  ```cpp
  HANDLE hFile = CreateFile(L"example.txt", GENERIC_READ | GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
  HANDLE hMapFile = CreateFileMapping(hFile, NULL, PAGE_READWRITE, 0, 1024, NULL);
  if (hMapFile != NULL) {
      LPVOID lpMapAddress = MapViewOfFile(hMapFile, FILE_MAP_WRITE, 0, 0, 0);
      if (lpMapAddress != NULL) {
          // Use the mapped memory...
          UnmapViewOfFile(lpMapAddress);
      }
      CloseHandle(hMapFile);
  }
  CloseHandle(hFile);
  ```

### Key Concepts

- **Heap vs. Virtual Memory**: Heap memory is managed by the heap manager and is generally used for smaller, dynamically-allocated memory blocks. Virtual memory functions like `VirtualAlloc` and `VirtualFree` allow for larger, page-aligned allocations and give more control over the memory's protection and location.

- **Global vs. Local Allocations**: These are older methods of memory management, with `GlobalAlloc` and `LocalAlloc` being legacy functions from 16-bit Windows. Modern applications typically prefer heap or virtual memory functions.

- **File Mapping**: Allows a file to be mapped into memory, providing a way to read and write files through memory pointers. This is useful for large files or for sharing memory between processes.

These APIs and techniques provide a robust set of tools for managing memory in Windows applications, allowing for both high-level and low-level control over memory allocation, protection, and usage.