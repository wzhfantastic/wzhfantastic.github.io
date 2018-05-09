---
layout:     post
title:      Physics Simulation Based On Runge–Kutta methods
date:       2018-05-09
author:     Fantasy Wang
header-img: img/tag-bg-o.jpg
catalog: false
tags:
    - Physics Simulation
    - Runge–Kutta
    - OpenGL
---
Today, I implemented a simple cloth physics system. It's just a particle group connected by springs. The position and speed of mesh vertex are calculated by fourth order [Runge–Kutta methods](https://en.wikipedia.org/wiki/Runge%E2%80%93Kutta_methods). 

Here is the result.
![result](/img/PostsImg/PhysicsSimulation/physicssimulation.gif)
