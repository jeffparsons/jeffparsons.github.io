---
layout:     post
title:      "PlanetKit week 4: filing off some rough edges"
date:       2016-11-18
summary:    Fixing the display of cells that straddle chunk boundaries, and chipping away at the most obvious overdraw.
tags:       rust planetkit
published:  false
---

PlanetKit is a project aimed at creating a toolkit for making interactive virtual worlds. I'm writing it in the Rust programming language, which I'm finding delightful to work with, and a great fit for the task.

Last week we turned our lumpy blob into world of hexagonal prism voxels. That ended up looking like this:

![Hexagonal prisms](/images/hexagons/globe-with-hexagonal-prisms.png)

But there were a couple of really obvious problems.


## Accidental rendering of back-faces

First and more boring of those was how much redundant geometry I was rendering. If you peek inside the globe, you'd see something like this:

![Unnecessary faces](/images/hexagons/globe-with-unnecessary-faces.png)

I noticed from my swimming around inside the globe that I could see the tops of the hexagonal prisms from underneath. That's not great. That means that most of the time there will be a bunch of triangles being drawn to the screen that I never actually intend to be seen. So the first easy win there was to enable [back-face culling](https://en.wikipedia.org/wiki/Back-face_culling). This is how I was initializing my Gfx-rs [Pipeline State Object](https://gfx-rs.github.io/2016/01/22/pso.html) before:

{% highlight rust %}
let pso = factory.create_pipeline_simple(
    Shaders::new()
        .set(GLSL::V1_50, include_str!("shaders/copypasta_150.glslv"))
        .get(glsl).unwrap().as_bytes(),
    Shaders::new()
        .set(GLSL::V1_50, include_str!("shaders/copypasta_150.glslf"))
        .get(glsl).unwrap().as_bytes(),
{% endhighlight %}

It's nice to have a simpler helper like `create_pipeline_simple`, and I'm surprised to have "outgrown" it so quickly. Fortunately the not-quite-as-simple PSO set-up isn't much more complicated:

{% highlight rust %}
let mut vs = Shaders::new();
let vs_bytes = vs
    .set(GLSL::V1_50, include_str!("shaders/copypasta_150.glslv"))
    .get(glsl).unwrap().as_bytes();
let mut ps = Shaders::new();
let ps_bytes = ps
    .set(GLSL::V1_50, include_str!("shaders/copypasta_150.glslf"))
    .get(glsl).unwrap().as_bytes();
let shader_set = factory.create_shader_set(vs_bytes, ps_bytes).unwrap();
let pso = factory.create_pipeline_state(
    &shader_set,
    Primitive::TriangleList,
    Rasterizer::new_fill().with_cull_back(),
    pipe::new()
).unwrap();
{% endhighlight %}

Notice the call to `with_cull_back` in the replacement code. Yay, no more pointless back-faces.


## Unreachable voxels

The next step was to cull _all_ the geometry for any voxels that are obviously never going to be visible. I initially started trying to write out something clever that would properly take into account voxels on the edge of chunk boundaries, and do it efficiently by making sure I always had cached copies of all the voxels from neighbouring chunks that I---no! Stop! Make it work, make it right, make it fast. In that order.

I might change my mind about the fundamentals of the way this voxmap is stored three times before this level of optimisation is relevant. So instead I dealt with the most egregious waste of triangles within the bulk of chunks with the following simple algorithm:

1. If the voxel is on the edge of a chunk, draw it.
2. Otherwise, if any of its neighboring voxels are made of air, then _maybe_ it could be seen, so draw it.
3. Otherwise, don't draw it.

This eliminated the vast majority of wasted geometry in my globe. I haven't bothered getting numbers to back this up yet, but I was able to crank the resolution of the world up a _lot_ after doing this before it started slowing down again.


-----



TODO: describe move to multi-shaped cells, with screenshots
commit d7b1414e8c9a9b4328dfddafbba330b456c2e67c
Author: Jeff Parsons <jeff@parsons.io>
Date:   Wed Nov 9 23:11:45 2016 +1100

    Draw different shaped cells
    
    Each chunk draws cells so that only the part of the hexagon
    that lies in that chunk is drawn.
    
    The next step is to store cell colors in the voxmap so we can
    get rid of those hard lines through the middle of cells.

commit eac7d4dadd686c317819b8d6769bc84f50ec18d9

commit eac7d4dadd686c317819b8d6769bc84f50ec18d9
Author: Jeff Parsons <jeff@parsons.io>
Date:   Wed Nov 9 22:25:23 2016 +1100

    Rewrite geometry generation to handle shapes
    
    Shapes other than full hexagons, that is.

commit cce6179fbd270e7ff9681b8f151cb545ed617819
Author: Jeff Parsons <jeff@parsons.io>
Date:   Wed Nov 9 20:50:13 2016 +1100

    Make cell_shape module
    
    Moved hexagon vertex offset info into this module.






TODO: mention moving to 5 root quads instead of 10

commit 84e8d6be374cb921b39ef7680dcadddb99629859
Author: Jeff Parsons <jeff@parsons.io>
Date:   Thu Nov 10 20:59:59 2016 +1100

    Move to having 5 root quads instead of 10
    
    Using 10 was a misguided. I had wanted to keep the root quads square
    for the sake of simplicity, but in retrospect that actually makes things
    more complicated.
    
    This slightly complicates the `project` method, but it means that when
    it comes time to do all calculus on positions and vectors in voxel space
    there will only be one kind of root quad (as opposed to different logic
    for northern hemisphere quads vs. southern hemisphere), and logic about
    propagating edge voxels between chunks will be simpler.
    
    This revealed a tiny bug where I had used the x-resolution and y-resolution
    the wrong way. Having a deliberately non-square world resolution will now
    very quickly reveal any similar bugs, so there's a bonus.




TODO: mention copying content over between cells.
make reference to almost accidentally starting trying to do the smart thing, rather than the thing that works. BIG NO NO. partly from not writing about what I'm doing as I do it?
commit 8a702402772623319f0ec3dbd10001ac1d4a87f4
Author: Jeff Parsons <jeff@parsons.io>
Date:   Wed Nov 16 21:49:47 2016 +1100

    Copy cells from chunks that own them
    
    Copy cells over from chunks that own cells to those that
    contain the same cells but don't own them.
    
    This resolves the last obvious visual glitch: hard lines down
    the middle of rows of cells where the chunk on each side had
    a different idea of what is in that cell. Now both chunks will
    end up with a copy of the authoritative state. Hurrah! :)




TODO: crank up the resolution and go?




TODO: what's next?





As always, the source for everything I'm talking about here is up in the [_planetkit_ repository on GitHub](https://github.com/jeffparsons/planetkit).





