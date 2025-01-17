<h1 align='center'> AdaptOctree </h1>

AdaptOctree is an library to *quickly* **build** and **balance** adaptive linear octrees written in Python, and Python's ample numeric libraries.

Adaptive linear octrees are a data structure useful in a large variety of scientific and numerical codes. AdaptOctree has been designed for use within [PyExaFMM](https://github.com/exafmm/pyexafmm), a Pythonic Kernel Independent Fast Multipole Method implementation. However, it is quite happy to work on its own too.

AdaptOctree is no longer directly supported, and serves now only as an example of how to find Morton encodings and use them to construct octrees.

# Installation


```bash
python -m pip install .
```

# Usage

The code consists of two basic modules:

## 1) `adaptoctree.morton`

This module is used for generating Morton encodings and decodings for 3D Cartesian Points.

We provide a number of optimised methods for handling Morton encodings, for example neighbour searching, children/parent searching etc.


e.g. Generating the encodings for some points.

```python
import numpy as np

import adaptoctree.morton as morton

# Generate randomly distributed 3D points in a unit cube domain
n_points = int(1e5)
points = np.random.rand(n_points, 3)

# Calculate implicit octree parameters
## Furthest corners of octree root node, with respect to origin
max_bound, min_bound = morton.find_bounds(points)

## Octree root node center
x0 = morton.find_center(max_bound, min_bound)

## Octree root node half side length
r0 = morton.find_radius(x0, max_bound, min_bound)

# Apply Morton encoding, at a given octree level
level = 1
keys = morton.encode_points(points, level, x0, r0)
```

## 2) `adaptoctree.tree`

This module is used for building and balancing trees, from a set of points.

e.g. Building and balancing.

```python
import numpy as np

import adaptoctree.morton as morton
import adaptoctree.tree as tree

# Generate randomly distributed 3D points in a unit cube domain
n_points = int(1e5)
points = np.random.rand(n_points, 3)

# Build parameters
max_level = 16 # Maximum level allowed in AdaptOctree
max_points = 100 # The maximum points per node constraint
start_level = 1 # Initial level, to start tree construction from

unbalanced = tree.build(points, max_level, max_points, start_level)

# Now balance the unbalanced tree
depth = np.max(morton.find_level(unbalanced)) # Maximum depth achieved in octree

balanced = tree.balance(unbalanced, depth)
```

Note: The first import of AdaptOctree will generate a cache of Numba compiled functions, and therefore might take some time.

### Computing Interaction Lists

The third main functional use of this module is to compute the interaction lists
required by the particle FMM. We follow standard definitions as in [4].


```python
# Compute all interaction lists for leaves
import adaptoctree.tree as tree

# Find linear representation as above ...

# Complete the linear tree, i.e. including all ancestors
complete = tree.complete_tree(balanced)

u, x, v, w = tree.find_interaction_lists(balanced, complete, depth)

print(u.shape) # (len(balanced), 90)
print(x.shape) # (len(balanced), 9)
print(v.shape) # (len(balanced), 189)
print(w.shape) # (len(balanced), 208)
```

The size of each list is pre-allocated the maximum possible size of each list
for a given node, so in practice many of their members will be '-1', which is
used to indicate no entry.

# Contributing

We welcome any contributions, check the open issues for currently troublesome bugs or feature requests.

We follow the same [developer guidelines](https://github.com/exafmm/pyexafmm/wiki/Contributing-%F0%9F%92%BB) as the PyExaFMM project.

# Citation

If you decide to cite our work, please do drop us an [email](mailto:srinathkailasa@gmail.com)!

We'd love to know what kind of projects you plan to use AdaptOctree in.

## References

[1] Sundar, H., Sampath, R. S., & Biros, G. (2008). Bottom-up construction and 2: 1 balance refinement of linear octrees in parallel. SIAM Journal on Scientific Computing, 30(5), 2675-2708.

[2] Tu, T., O'Hallaron, D. R., & Ghattas, O. (2005, November). Scalable parallel octree meshing for terascale applications. In SC'05: Proceedings of the 2005 ACM/IEEE conference on Supercomputing (pp. 4-4). IEEE.

[3] [The ExaFMM Project](https://github.com/exafmm)

[4] Lashuk, I., Chandramowlishwaran, A., Langston, H., Nguyen, T. A., Sampath, R., Shringarpure, A., & Biros, G. (2009, November). A massively parallel adaptive fast-multipole method on heterogeneous architectures. In Proceedings of the Conference on High Performance Computing Networking, Storage and Analysis (pp. 1-12). IEEE.
