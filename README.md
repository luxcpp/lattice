# Lux Lattice Library

GPU-accelerated lattice cryptography library for post-quantum cryptographic operations.

## Features

- **NTT Operations**: Fast Number Theoretic Transform for polynomial multiplication
- **Polynomial Arithmetic**: Add, subtract, multiply polynomials in NTT domain
- **Gaussian Sampling**: Discrete Gaussian sampling for lattice-based schemes
- **GPU Acceleration**: Metal (macOS), CUDA (Linux), with CPU fallback
- **pkg-config Support**: Easy integration with CGO and other build systems

## Installation

### From Pre-built Releases

```bash
# Download and install (requires curl)
curl -sSL https://raw.githubusercontent.com/luxfi/lattice-cpp/main/scripts/install.sh | bash

# Or specify version and prefix
./scripts/install.sh 1.0.0 /usr/local
```

### From Source

```bash
git clone https://github.com/luxfi/lattice-cpp
cd lattice-cpp
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
sudo make install
```

## Usage with CGO (Go)

The library provides a pkg-config file for easy CGO integration:

```go
//go:build cgo

package gpu

/*
#cgo pkg-config: lux-lattice
#cgo CXXFLAGS: -std=c++17 -O3
#cgo darwin LDFLAGS: -framework Metal -framework Foundation -lstdc++
#cgo linux LDFLAGS: -lstdc++

#include <lattice.h>
*/
import "C"
```

Ensure `PKG_CONFIG_PATH` includes the installation path:

```bash
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
```

## API Reference

### NTT Context

```c
// Create NTT context for given ring parameters
LatticeNTTContext* lattice_ntt_create(uint32_t N, uint64_t Q);
void lattice_ntt_destroy(LatticeNTTContext* ctx);

// NTT operations
int lattice_ntt_forward(LatticeNTTContext* ctx, uint64_t* data, uint32_t batch);
int lattice_ntt_inverse(LatticeNTTContext* ctx, uint64_t* data, uint32_t batch);
```

### Polynomial Operations

```c
int lattice_poly_mul_ntt(LatticeNTTContext* ctx, uint64_t* result,
                         const uint64_t* a, const uint64_t* b);
int lattice_poly_add(uint64_t* result, const uint64_t* a, const uint64_t* b,
                     uint32_t N, uint64_t Q);
int lattice_poly_sub(uint64_t* result, const uint64_t* a, const uint64_t* b,
                     uint32_t N, uint64_t Q);
```

### Sampling

```c
int lattice_sample_gaussian(uint64_t* result, uint32_t N, uint64_t Q,
                            double sigma, const uint8_t* seed);
int lattice_sample_uniform(uint64_t* result, uint32_t N, uint64_t Q,
                           const uint8_t* seed);
int lattice_sample_ternary(uint64_t* result, uint32_t N, uint64_t Q,
                           double density, const uint8_t* seed);
```

### Utility Functions

```c
bool lattice_gpu_available(void);
const char* lattice_get_backend(void);
uint64_t lattice_find_primitive_root(uint32_t N, uint64_t Q);
uint64_t lattice_mod_inverse(uint64_t a, uint64_t Q);
bool lattice_is_ntt_prime(uint32_t N, uint64_t Q);
```

## Performance

Benchmarks on Apple M1 Max:

| Operation | CPU (ns) | GPU (ns) | Speedup |
|-----------|----------|----------|---------|
| NTT Forward (N=4096) | 45,000 | 1,200 | 37x |
| NTT Inverse (N=4096) | 48,000 | 1,300 | 37x |
| Poly Mul (N=4096) | 95,000 | 2,800 | 34x |

## License

Apache 2.0 - See [LICENSE](LICENSE) for details.

## Related Projects

- [luxfi/lattice](https://github.com/luxfi/lattice) - Go lattice cryptography package
- [luxfi/fhe](https://github.com/luxfi/fhe) - Fully Homomorphic Encryption
- [luxfi/crypto](https://github.com/luxfi/crypto) - Core cryptographic primitives
