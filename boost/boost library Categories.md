The Boost C++ Libraries are organized into several functional categories to help developers find the tools they need for specific tasks. Below are the libraries listed by their official categories:

### **Algorithms**

- **Algorithm:** A collection of useful generic algorithms.
- **Foreach:** A loop construct that automates iterating over a sequence.
- **Geometry:** Provides geometric algorithms, primitives, and spatial indices.
- **GIL:** A Generic Image Library for image processing.
- **Graph:** The Boost Graph Library (BGL), providing generic graph components.
- **GraphParallel:** Parallel extensions to the graph library.
- **Histogram:[直方图]** Fast multi-dimensional histograms with a C++14 interface.
- **Min-Max:** Standard library extensions for simultaneous min/max computations.
- **Polygon:** Tools for Voronoi [沃罗诺伊图] diagram construction and planar polygon operations.
- **QVM:** A library for working with Quaternions [四元数], Vectors, and Matrices.
- **Range:** New infrastructure for generic algorithms building on iterator concepts.
- **Sort:** High-performance templated sort functions.
- **String Algo:** A library dedicated to string algorithms.

### **Concurrent Programming**

- **Asio:** Portable networking and low-level I/O (sockets, timers, etc.).
- **Atomic:** C++11-style atomic types.
- **Beast:** HTTP, WebSocket, and network operations built on Asio.
- **Cobalt:** Basic coroutine algorithms and types.
- **Compute:** A parallel and GPU-computing library.
- **Context:** Context switching library (C++11).
- **Coroutine / Coroutine2:** Specialized coroutine implementations.
- **Fiber:** Userland [用户空间] threads library.
- **Interprocess:** Shared memory, memory-mapped files, and process-shared mutexes. 
  - [如何让复杂 C++ 对象安全地跨进程存在与访问]
- **Lockfree:** Specialized lockfree data structures.
- **MPI:** Message Passing Interface for distributed-memory applications.
- **MQTT5 / MySQL / Redis:** Specialized client libraries built on top of Asio.
- **Thread:** Portable C++ multi-threading.

### **Containers**

- **Array:** STL-compliant wrapper for fixed-size arrays.
- **Bimap:** Bidirectional maps library.
- **Bloom:** Implementation of Bloom filters.
- **Circular Buffer:** A ring or cyclic buffer container.
- **Container:** Standard library container extensions.
- **ICL:** Interval Container Library for sets and maps of intervals.
- **Intrusive:** High-performance intrusive containers and algorithms.
- **Multi-Array:** Generic N-dimensional array implementations.
- **Multi-Index:** Containers maintaining multiple indices with different semantics.
- **Pointer Container:** Containers for heap-allocated polymorphic objects.
- **Property Tree:** Tree structure for storing configuration data.
- **Static String:** Fixed-capacity, dynamically sized strings.
- **Unordered:** Unordered associative containers.
- **Variant / Variant2:** Type-safe union and discriminated union containers.

### **Correctness and Testing**

- **Assert:** Customizable assert macros.
- **Contract:** Support for contract programming (invariants, pre/postconditions).
- **Test:** Comprehensive support for unit testing and program monitoring.
- **Stacktrace:** Gathers and prints backtraces.
- **Static Assert:** Compile-time assertions.

### **Data Structures**

- **Any:** Safe, generic container for single values of different types.
- **Fusion:** Tools for working with tuples and heterogeneous sequences.
- **Heap:** Priority queue data structures.
- **JSON:** JSON parsing, serialization, and DOM tools.
- **Optional:** A wrapper for representing nullable objects.
- **PFR:** Basic reflection for user-defined types.
- **Tuple:** Simplifies defining functions that return multiple values.
- **Uuid:** Universally unique identifier implementation.

### **Domain Specific**

- **Chrono:** Useful time utilities.
- **Date Time:** Date-time libraries based on generic programming concepts.
- **Units:** Zero-overhead dimensional analysis and unit manipulation.
- **CRC:** Cyclic redundancy code computation.

### **Error Handling and Recovery**

- **Exception:** Supports transporting arbitrary data in exception objects.
- **LEAF:** A lightweight error-handling library.
- **System:** Extensible error reporting framework.
- **ThrowException:** Infrastructure for throwing exceptions from Boost libraries.

### **Function Objects and Higher-Order Programming**

- **Bind:** Generalized argument binder for functions and member functions.
- **Lambda / Lambda2:** Libraries for defining unnamed function objects at call sites.
- **Phoenix:** Defines small unnamed function objects.
- **Signals2:** Managed signals and slots (thread-safe version).
- **HOF:** Higher-order functions for C++.

### **Generic Programming**

- **Call Traits:** Defines types for passing parameters.
- **Enable If:** Selective inclusion of function template overloads.
- **Type Traits:** Templates for fundamental properties of types.
- **YAP / Proto:** Expression template and DSL toolkit libraries.

### **Math and Numerics**

- **Integer:** Headers designed to take advantage of standard integer types.
- **Interval:** Extends arithmetic functions to mathematical intervals.
- **Math:** Large collection including statistical distributions and root finding.
- **Multiprecision:** Extended precision arithmetic for various types.
- **Random:** A complete system for random number generation.
- **uBLAS:** Tensor, matrix, and vector classes with linear algebra routines.

### **String and Text Processing**

- **Format:** Type-safe mechanism for printf-like formatting.
- **Lexical Cast:** General literal text conversions (e.g., string to int).
- **Locale:** Localization and Unicode handling tools.
- **Regex / Xpressive:** Regular expression libraries.
- **Tokenizer:** Breaks sequences into a series of tokens.
- **Wave:** Standards-conformant C++ preprocessor implementation.

### **Other Categories**

- **Broken compiler workarounds:** **Config**.
- **Image Processing:** **GIL**.
- **Input/Output:** **Asio, Iostreams, Serialization, URL**.
- **Inter-language support:** **Python**.
- **Memory:** **Align, Pool, Smart Ptr**.
- **Parsing:** **Parser, Spirit**.
- **Preprocessor Metaprogramming:** **Preprocessor, VMD**.
- **System:** **Filesystem, Process, DLL**.
- **Template Metaprogramming:** **Hana, MPL, Mp11**.