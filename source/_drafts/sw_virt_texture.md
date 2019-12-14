Software Virtual Textures

Feb.25th, 2012

## Aspects referring to the solutions
- address translation
- texture filtering
- oversubscription
- LOD snapping
- compression
- caching
- streaming

## Problem needed to be addressed
Even though texture mapping enables efficient management and rendering of surface detail,
unique texture data requires **significant storage** and **bandwidth**

### Current solutions
- Tiling texture that can be repeated many times on a single surface.
- Mutiple textures can be blended to composite a variety of detail.

These above are considered specialized forms of texture compression that will burden on
GPU

### Proposed solution
virtual texture with a sparse representation

#### Components for virtual texture
- physical texture pages
- page table that maps virtual addresses to physical ones

#### Differences from virtual memory system
- possible to fall back to slightly blurrier data without stalling execution
- lossy compression of the data is acceptable

#### Previous work
The clip-map is one of the first effective schemes for virtual texture. It consists of a
stack of images similar to mip-map hierarchy.



