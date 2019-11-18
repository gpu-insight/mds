## graphics community
- raster graphics
- vector graphics

## displayed size
create a mapping between physical and abstract
- millimeters and typograhic points
- application-determined semantics

## Immediate-Mode versus Retained-Mode
IM advantages: maximum performance, minimum resource

RM advantages: reusable components, retain the scene graph and modify it

|    | IM             |           RM       |
|----|:---------------|:------------------:|
| 2D | Java awt.Graphics2D</p>Apple Quartz</p>2G GDI+ | Microsoft WPF 2D |
| 3D | OpenGL</p>DirectX | Microsoft WPF 3D |

# Human Visual Perception
The visual system is both tolerant of bad data and sensitive.

## Special properties for visual system compared to other modes of computer-to-human interaction like touch, sound  
- light is not directionally diffuse
  
## The visual system
### measuring the similarity between two images
sum-squared difference

our visual system is more sensitive to radiance(辉度) errors

changes in the intensity tend to matter more than absolute intensity

### something about our visual system
1. Our perception of things is fairly independent of lighting.
2. The early portions of the visual system tend to detect edges and assemble them into something that the brain perceives as a whole.

### the visual system's ability to detect distance to an object through two different mechanisms
1. 眼睛可以聚焦，双眼同时可以定位物体的深度
- 3D glasses
2. 眼睛对周围环境的光线的适应性和有限的动态范围，意味着我们不需要构建像素间对比度很大的显示器
