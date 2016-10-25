---
layout:     post
title:      Basic terrain with simplex noise
date:       2016-10-19
summary:    A first attempt at making terrain for PlanetKit using the `noise` crate.
tags:       rust planetkit
published:  true
---

Last week I [introduced PlanetKit]({% post_url 2016-10-19-introducing-planetkit %})---a project aimed at creating a toolkit for making interactive virtual worlds. Here's where we got to last week:

![A globe](/images/globe-outside.png)

And here's what I originally intented to do next:

> Next up is rendering the cells as hexagonal prisms, and taking a first stab at terrain by throwing some simplex noise at a voxmap.

Well, that turned out to be half-true; I realised about five minutes in that there would be a nontrivial amount of work between where I was and the way I want to implement my hexagonal prism voxels. So I instead of either doing that work now or building a half-baked interim version that I'd just need to throw away _next_ week, I decided to make my first terrain attempt on the quads I already have.

From my earlier playing around I'm pretty happy with the [`noise` crate](https://crates.io/crates/noise) ([docs](https://docs.rs/noise/0.2.0/noise/)), so I hooked it up like so:

{% highlight rust %}
pub fn build_chunk(&mut self, origin: CellPos) {
    // Layer 4 octaves of 3-dimensional Perlin noise to make "good-enough-for-now" terrain.
    let noise = noise::Brownian3::new(noise::perlin3::<f64>, 4).wavelength(1.0);
    let mut cells: Vec<Cell> = Vec::new();
    let end_x = origin.x + self.spec.chunk_resolution;
    let end_y = origin.y + self.spec.chunk_resolution;
    for cell_y in origin.y..end_y {
        for cell_x in origin.x..end_x {
            // Calculate height for this cell from world spec.
            // To do this, project the cell onto a unit sphere
            // and sample 3D simplex noise to get a height value.
            let cell_pos = CellPos {
                root: origin.root,
                x: cell_x,
                y: cell_y,
            };
            let pt3 = self.cell_center(cell_pos);

            // Vary a little bit around 1.0.
            let delta =
                noise.apply(&self.pt, pt3.as_ref())
                * self.spec.radius
                * 0.1;
            let height = self.spec.radius + delta;
            cells.push(Cell {
                height: height,
            });
        }
    }
    self.chunks.push(Chunk {
        origin: origin,
        cells: cells,
    });
}
{% endhighlight %}

I later use that heightmap to build the planet's geometry.

You may notice I'm actually using Perlin noise here. For those unfamiliar with Perlin noise, it was popular for this sort of thing before simplex noise came along and largely superceded it. Using Perlin noise here was just a brain fart.

Anyway, this looks like...

![A lumpy globe](/images/terrain/basic-terrain.png)

Hurrah! Pretty basic, but it's clearly a lumpy globe. You could almost call it a planet. Ok, maybe a planetoid. Obviously we'll eventually want to do something far more complex than this, but even this is good enough placeholder terrain for us to move on to other things.

If you're wondering about the giant cracks, don't worry too much. They exist because I split the world into chunks, and two edges of each chunk currently don't have convenient access to the data for the next chunks over. The system I'm planning to implement for turning this all into a giant voxmap will naturally take care of this.


## Turn it up!

My next step was of course to try cranking up the resolution.

![Higher resolution broken globe](/images/terrain/high-res-terrain-missing-geometry.png)

Uh oh... what happened there?

My first knee-jerk reaction was roughly "ugh, there's some video card or OpenGL limits that I don't know about, it's silently discarding my geometry, and I'll have to batch it up or something". It really shouldn't have been my first thought, but it was. In my experience when I run into a bug like this it's almost always something embarrassingly simple. And yet I'm still far too quick to assume I've run into some complex and arcane defect or limitation.

So that wasn't particularly useful. My second thought was "ok, that's an awfully neat shape. Maybe there's a magic number in here that might clue me in to the order of magnitude we're talking about." So I tried tweaking the resolution of my globe to something less convenient than it was before (root quad side length of 64, chunk side length of 16). Let's try a root side length of 80, and a chunk side length of 20.

![A globe](/images/terrain/high-res-terrain-counting.png)

Ooookay. This is more interesting. Note the partial chunk at the tip of the arrow. This calls for a quick count. I can see...

- 2 full roots
- 8 full chunks
- 20 * 19 + 4 = 384 quads

Based on the chunk and total root side lengths of 20 and 80 respectively, that adds up to 16384, or 2^14. Aha! That's an awfully convenient number. So maybe index buffers can only be so big and they just get truncated, or---

This is when I started to cringe. 2^14 leaves a factor of 4 before busting an unsigned 16-bit integer. And each quad has 4 vertices. Which just happens to be the size used for vertex indices in the tutorial code I started with, and which I never revisited. But then why, I wondered, wasn't I getting an overflow explosion at some point? The culprit turned out to be as simple as this:

{% highlight rust %}
let first_vertex_index = vertex_data.len() as u16;
{% endhighlight %}

So I was explicitly truncating the index before I did anything with it. D'oh! Ok, so let's try bumping up the size of the type I'm using for indexing vertices...

![A globe](/images/terrain/high-res-terrain-working.png)

Hurrah! All is good in the world again.


## Can we make it a bit prettier?

We sure can. If I make all the land green, and also render an extra blue globe without any noise applied to its surface, then we can make something that looks vaguely terrestrial:

![A globe](/images/terrain/high-res-terrain-earth.png)

I haven't pushed the code for this bit anywhere, because it's just silly copy-and-paste hacks just to get a preview of where we're headed.


## What's next?

My next move will be to turn this into a proper voxmap of (mostly) hexagonal prisms, à la Minecraft. Or, you know, I might lie about that again and do something else instead.

After that, I need to figure out how to use `gfx` properly so I can dynamically create and destroy chunks of the world.


## Wrapping up

Lessons learned / re-learned:

- The `noise` crate does what it says on the tin. Nice, simple API, and seems to Just Work.
- Water makes even Perlin noise based terrain look a lot more legit.
- Always default to assuming errors are probably something silly and simple.
- Working on smallest chunk of demonstrable work is far more satisfying than trying to build up grand architectures before having anything to look at. I'm doing this for pleasure, so constant refactoring or rewriting is fine. And I'll probably build more useful things more quickly that way.

Tune in next week for more Rusty graphics fun!
