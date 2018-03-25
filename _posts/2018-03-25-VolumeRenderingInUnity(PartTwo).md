---
layout:     post
title:      Volume Rendering Practice In Unity(Part Two)
subtitle:   Deeper Research About Shading
date:       2018-03-25
author:     Fantasy Wang
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Unity
    - Rendering
    - Volume Rendering
    - Raymarching
---
In the first part of **Volume Rendering Practice In Unity**, we introduced the basic theory of **RayMarching Distance Field** and created some basic shape using that theory.
However, we only implement a simple lighting mode for shading. 
![Simple Lighting Volume](/img/PostsImg/VolumeRenderingPractice/simple-volum-lighting.png)
So today I will try some more detailed shading technique to create more realistic volume object.
### Shadow
 I find a good introduction for raymarching shadows implement : [Iñigo Quile's soft shadow](http://iquilezles.org/www/articles/rmshadows/rmshadows.htm).
 
##### The algorithm 
- Using distance field funtion(we call it **map()**) to raymarch and hit the target object.
- Start another ray from hit point, the direction is the light direction.
- Step by step, if point in this ray fits the map funtion, the origin point should be in the shadow.

##### Implementation
```c
float shadow( float3 p, float3 lightDirection, float mint, float maxt ,float k)
{
    float ret = 1;
    for(float t = mint; t < maxt; )
    {
        float h = map(p + lightDirection*t);
        if(h < 0.0001)
            return 0;

        ret = min( ret, k*h/t );
        t += h;
    }
    return ret;
}
```


As we can see, we don't need to loop by a const step, but use a map function() return value **h** as a step. Because **h** is the distance between current point and the volume surface. So we at least should march distance **h** to get into the volume.

And about the **k** value, as **Iñigo Quile** introduced:
> k parameter in the function controls how hard/soft the shadows are

##### Effect
![Simple Lighting Volume](/img/PostsImg/VolumeRenderingPractice/volume-light-shadow.png)
Cool! We can see the shadow now.

### Ambient Occlusion
The basic purpose of ambient occlusion is to recalculate ambient light by distance between object. So I try following algorithm. 
##### The algorithm 
- Using  **map()** to raymarch and hit the target object. 
- Start another ray from hit point, the direction is the normal of the hit point.
- Step by step, calc the **map()** function. If the object is not occluded by nearby objects, the return value **h** of **map()** function should be larger and larger. If **h** becomes smaller, set **h** as the occluder's distance and influence the ambient.

##### Implementation
```c
float ao(float3 p, float3 normal, float step, float maxStep)
{
    float lastH = 0;
    for(int i = 0; i < maxStep; i++)
    {
        float h = map(p);
        if(lastH != 0 && h < lastH)
            return i / maxStep;

        p += normal * step;
        lastH = h;
    }
    return 1;
}
```

And I apple this influence to final ambient:
```c
UNITY_LIGHTMODEL_AMBIENT * min(1, pow(aoRet, 0.25))
``` 
##### Effect
![Simple Lighting Volume](/img/PostsImg/VolumeRenderingPractice/volume-light-ao.png)
That looks better.

### Conclusion
I have used some simple code to make the volume more realistic, that's interesting and amazing. I believe if we spend more time on creating and shading volumetric object, we can see more great view! 

In next part, I think I should find some more pratical application for this volume rendering technique. Hope you enjoy my post!