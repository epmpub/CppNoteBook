



# instrinsic demo code

```C++
#include <immintrin.h>
#include <iostream>

void compute_avx512(float* a, float* b, float* result, size_t n) {
    for (size_t i = 0; i < n; i += 16) {
        __m512 va = _mm512_load_ps(a + i);
        __m512 vb = _mm512_load_ps(b + i);
        __m512 vresult = _mm512_add_ps(va, vb);
        _mm512_store_ps(result + i, vresult);
    }
}

void compute_avx(float* a, float* b, float* result, size_t n) {
    for (size_t i = 0; i < n; i += 8) {
        __m256 va = _mm256_load_ps(a + i);
        __m256 vb = _mm256_load_ps(b + i);
        __m256 vresult = _mm256_add_ps(va, vb);
        _mm256_store_ps(result + i, vresult);
    }
}

void compute_sse(float* a, float* b, float* result, size_t n) {
    for (size_t i = 0; i < n; i += 4) {
        __m128 va = _mm_load_ps(a + i);
        __m128 vb = _mm_load_ps(b + i);
        __m128 vresult = _mm_add_ps(va, vb);
        _mm_store_ps(result + i, vresult);
    }
}

void compute_scalar(float* a, float* b, float* result, size_t n) {
    for (size_t i = 0; i < n; ++i) {
        result[i] = a[i] + b[i];
    }
}

int main() {
    alignas(64) float a[16] = {1.0f, 2.0f, 3.0f, 4.0f, 5.0f, 6.0f, 7.0f, 8.0f, 
                               1.0f, 2.0f, 3.0f, 4.0f, 5.0f, 6.0f, 7.0f, 8.0f};
    alignas(64) float b[16] = {5.0f, 6.0f, 7.0f, 8.0f, 1.0f, 2.0f, 3.0f, 4.0f, 
                               1.0f, 2.0f, 3.0f, 4.0f, 5.0f, 6.0f, 7.0f, 8.0f};
    alignas(64) float result[16];

    // 运行时检查 CPU 支持的指令集
    if (__builtin_cpu_supports("avx512f")) {
        std::cout << "Using AVX-512\n";
        compute_avx512(a, b, result, 16);
    } else if (__builtin_cpu_supports("avx")) {
        std::cout << "Using AVX\n";
        compute_avx(a, b,result,16);
    }
}
```

