---
layout:     post
title:      Draw Mobius Strip With Generalized Helicoid
date:       2018-05-01
author:     Fantasy Wang
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - Mobius
    - Generalized Helicoid
    - OpenGL
---
Today, I did some research about [Generalized Helicoid](https://en.wikipedia.org/wiki/Generalized_helicoid). I will try to draw [Mobius Strip](https://en.wikipedia.org/wiki/M%C3%B6bius_strip) by this tech in **OpenGL**. 

##### The Key Theory
- Create a profile curve
- Create a sweep curve
- Sweep the profile curve along the sweep curve, the swept area is the generated surface.

##### Some Try
First, I created a rect prfile curve.
Then, I try to create the whole **Mobius Strip** sweep curve by a group of bezier curves.
![roughbeziersweep](/img/PostsImg/DrawMobius/roughbeziersweep.png)
However, the result is not satisfying.
![roughmobius](/img/PostsImg/DrawMobius/roughmobius.png)
If I need to create a smooth **Mobius Strip**, I should create a more accuracy mobius bezier curve group.

##### The Final Solution
Finally, I just created a circle profile curve by bezier curve group, and rotate the profile curve during the sweep stage by interplating. Then I got a better **Mobius Strip**.
 ![circleControlPoint](/img/PostsImg/DrawMobius/circleControlPoint.png)
![mobius1](/img/PostsImg/DrawMobius/mobius1.png)

##### More Mobius Strips
By modifying profile curve rotating interplating function, I can get more interesting **Mobius Strips**.
![mobius2](/img/PostsImg/DrawMobius/mobius2.png)
![mobius3](/img/PostsImg/DrawMobius/mobius3.png)