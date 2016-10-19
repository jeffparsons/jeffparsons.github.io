---
layout:     post
title:      The idea in detail
date:       2016-10-19
summary:    Something something go slow to go fast.
tags:       rust planetkit
published:  false
---


## Ugh, get to the point. What's this _idea_?

Oh, right. That. The idea I mentioned earlier is actually a mishmash of several different ideas that don't seem to be entirely incompatible. I haven't yet figured out how to combine them into a coherent plan or philosophy, but in the spirit of releasing early, I'll put some bullet points out there for comment:

- Draw heavy inspiration from the way Mozilla runs its communities, and try to apply it to a game technology project. Obviously much of the process and structure wouldn't be appropriate early on, but it could be layered on over time if there was enough interest in the project. Rust's [code of conduct](//www.rust-lang.org/en-US/conduct.html) in particular appeals to me, as does their collaborative approach to planning and reviewing changes. Encourage people to file issues for everything: bugs, feature requests, questions about why the project exists, etc.

- Release early (done), release often. Make commits as small as they can reasonably be. Always ask whether a given unit of work could be broken down into smaller units that could be _discussed_, built, and delivered separately. The goal here is to maximise interaction and transfer of knowledge, and drastically lower the barrier to entry for contributors by carving off small, well-defined tasks for other people to do wherever possible.

- Make documentation as important as running code. If I can't take an arbitrary struct or function and understand where it fits into the puzzle---at very least a link off to some other page/file that will provide some context---then that's a bug. The goal here is to minimise the barrier to entry, and to use the code as a teaching tool.

- Embrace the idea that creative projects like games can benefit from having a benevolent dictator (or very small, like-minded group of creative directors), and avoid trying to make art by committee. Instead of trying to rally people around a single game that must please everyone, focus on pushing shareable tech down as far as it can go, and encourage everyone to have their _own_ game project if they disagree with the direction of someone else's.

    In concrete terms this would mean prefering to contribute to some _other_ project's code if that would make the work useful to more people (e.g. improve someone else's ECS crate rather than building our own), and building as much as possible into reusable modules that could be mixed and matched for very different kinds of games.

    Maybe you always wanted to tinker with a game basically like Minecraft but on proper planets, perhaps with travel between them. Maybe you want almost exactly the same thing, but you want to focus on short team deathmatch-style games instead of persistent worlds. Maybe you just wanted to re-make Block Dude (e.g. [this one](//azich.org/blockdude) or a hundred other clones) but in 3D. Maybe you want to build a crazy peer-to-peer universe-scale MMO.

    All of these are really different games, but they could share a lot of tech, just like the hundreds of different free mods out there for the Quake series of game engines. They would all inevitably end up with their own bespoke and complex needs, but with a commitment to extracting the reusable bits for other people to play with, we could keep the simple case of each need simple enough that someone could walk up and start hacking on a _new_ game, visualisation, or simulation project with the same tech in around 100 lines of code, and then gradually specialise as the need arises.


## Wrapping up

This initial post has ended up rather nebulous. Subsequent posts will focus on specific details, and will include code, diagrams, and compiler output. I promise.
