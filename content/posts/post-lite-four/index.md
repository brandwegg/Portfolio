---
title: "Recreating the tentacles from returnal"
summary: "A little summary, how quaint."
categories: ["Post","Blog",]
tags: ["post","lorem","ipsum"]
#externalUrl: ""
#showSummary: true
date: 2026-03-31
draft: false
---

## Intro
For this project I wanted to recreate an effect I’d seen from Returnal, namely the tentacles they use generously throughout the game, both as environment and parts of enemies.

Without any prior knowledge of generating meshes on the GPU, I delved right in and based my experimentation on the fantastic GDC-presentation by Risto Jankkila and Sharman Jagadeesan from 2022.

## Breakdown 

### Setup
Fundamentally, there are 4 buffers in play. Two for the particles, one for the vertices and one for the indices. 
The particles are updated on the GPU and the vertices generated in the same step. 
The reason for two particle buffers is that one is read-only representing the previous frame’s data in order to avoid syncing millions of reads from the same buffer that is being written to. 
As mentioned in the GDC talk, this introduces a lag between each particle and the parent, which can be used as smoothing over time. 
It does come with complications, however, especially with fast movement, long dependency chains and variable frame rates. More on that later.

### Movement
The particles follow a combination of basic behaviours that are applied to the particles' velocities with adjustable weights. These behaviours are:
* Wriggle: a simple sin-wave to generate tentacle-esque movement.
* Straighten: a uniform force along the tentacle according to the orientation of the base.
* Stiffen: a force according to the orientation of the parent in order to reduce the amount of bend at each joint.
* Gravity: a uniform force downwards
* Look-at-camera: a uniform force towards the camera

[example code]()

The goal with these behaviours was mainly to demonstrate the flexibility that comes with blending behaviours and the capability of adjusting them both interactively and programmatically.

[gif of adjusting behaviours and combining them]()

### Mesh

Now that the positioning of the particles is out of the way, we can move on to the orientation. 
Each particle is logically oriented somewhere “in-between” the parent and the child, but how do we find that orientation? 
What I decided to be the “forward” in this situation, the direction along the curve, is straightforward enough: the direction from the parent of the node to the child. 
The “right” direction is trickier since it requires an optimal rotational alignment along the length of the shape to avoid twisting the vertices and thus deforming the triangles.

[insert Björn’s math]()

If we rotate a vector around the forward vector we get a ring of points around each particle, or “joint.” This serves perfectly as positions for the vertices. 
Using the right vector as the starting point, we ensure that all the points will be aligned.

For the indices, it helps visualising the vertices:

[insert image]()

For every ring, we have 5 vertices. For every ring, we offset the count by 5*n, where n is the index of the current particle. Now all we have to do is construct triangles and find a pattern in the data.

[insert notes]()

## Development
### Etiam sollicitudin
Etiam sollicitudin, ante ac fermentum varius, lorem ante congue mi, auctor dictum magna sem sed nibh. In et est id neque gravida aliquet quis a felis. Mauris tempor lectus ut gravida ornare. Curabitur at elementum tortor, in feugiat elit. Aenean auctor diam ut egestas rhoncus. Quisque tristique venenatis risus vitae suscipit. Nunc feugiat purus sed dolor gravida, non ullamcorper metus suscipit. Sed et tortor odio. Pellentesque at scelerisque nulla. In ut aliquam metus. Vivamus congue augue at pellentesque rhoncus. Donec a lectus tincidunt, aliquet libero sit amet, commodo arcu. Vivamus hendrerit quis augue eu lacinia. Sed sodales velit condimentum eros varius vulputate.


## Evaluation
### Proin tempor lorem
Proin tempor lorem quam, ac maximus lectus sodales et. Sed laoreet orci vel metus luctus lobortis. Nam ex velit, vehicula id tristique sed, blandit eu nisi. Quisque semper libero nec massa malesuada congue. In faucibus lorem at diam fringilla, vel viverra magna lobortis. Ut commodo est urna, ut aliquet enim sagittis ut. Nulla posuere arcu sed lobortis accumsan. Phasellus fringilla dolor id est lobortis feugiat. Quisque enim elit, faucibus a mauris non, mattis aliquet orci. Nunc sagittis viverra erat, id condimentum lacus suscipit quis.