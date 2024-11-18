---
layout: post
title:  "APAW Week 16: Trying out JAX"
date:   2024-10-27 00:19:13 +0100
categories: numpy
---

# Introduction
This week I wanted to try out JAX, a GPU accelerated implementation of numpy designed to improve performance significantly for complex computations. I use numpy at work and I was interested in if JAX could potentially be implemented in some of the computer vision projects we have for better performance. 

# Writing A Benchmark
I wanted to try out JAX with a quick benchmark, so I wrote the following code (based off of [this](https://gist.github.com/markus-beuckelmann/8bc25531b11158431a5b09a45abd6276) code)
```python
import numpy as np
import timeit

def benchmark():
    size = 1024
    A, B = np.random.random((size, size)), np.random.random((size, size))

    N = 20

    for i in range(N):
        np.dot(A, B)

time = timeit.timeit(benchmark, number=50)

print(time)
```

# Porting The Benchmark To JAX
To port the benchmark to JAX I had to rely on the original numpy for some of the code:
```python
import jax.numpy as np
import numpy as oldnp
import timeit

def benchmark():
    size = 1024
    A, B = oldnp.random.random((size, size)), oldnp.random.random((size, size))

    N = 20

    for i in range(N):
        np.dot(A, B)

time = timeit.timeit(benchmark, number=50)
print(time)

```

# The Results
<b><u>JAX was amazingly fast!</u></b>

The results in seconds:
```bash
> python numpy_cpu.py
27.300557171000037
```

```bash
> python numpy_gpu.py
12.697763189000398
```

# Conclusion
I am amazed by JAX, and it will likely be seeing use in my future projects. I expect that since it's niche it might be best to wrap it in a guard with the original numpy, though this might be an issue if it makes too many checks for JAX's missing APIs in the code:
```python
# Attempt to import GPU numpy (JAX), if not present use CPU operations
try:
    import jax.numpy as np
except ModuleImportError:
    import numpy as np
```