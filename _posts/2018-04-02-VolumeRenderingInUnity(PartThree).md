---
layout:     post
title:      Volume Rendering Practice In Unity(Part Three)
subtitle:   Volume Terrain
date:       2018-04-02
author:     Fantasy Wang
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - Unity
    - Rendering
    - Volume Rendering
    - Raymarching
---
In this final part of volome rendering practice, I will try to render a simple terrain with raymarching.
### Getting Ready
As we talked in previous two posts, we can use **SDF** to describe a shape. 
Now think abount a terrain, it's a ground with a height. So we can use a **SDF** to descripe the height of terrain in each position.
In this post, I will use a height map to save this **SDF** function in a texture.
![height map](/img/PostsImg/VolumeRenderingPractice/height.jpg)
 
### Mapping HeightMap 
I will render the terrain in XZ plane
> -100 < x < 100 && -100 < z < 100

Now I can map a world position to height map uv

```c
float2 GetUV(float3 position)
{
    float u = (position.x + 100) / 200;
    float v = (position.z + 100) / 200;
    return float2(u, v);
}
```
Then I can get a world position's height
```c
float Height(float3 position)
{
    float4 color = tex2Dlod (_HeightTex, float4(GetUV(position), 0, 0));
    return length(color) * 20;
}
```

Finally I get the **SDF** function
```c
float map(float3 p)
{
    return p.y - Height(p);
}
```

### Effect
Now we get this result
![height map](/img/PostsImg/VolumeRenderingPractice/terrain.png)
### Texturing
I attach a texture to this terrain. It seems cool.
![height map](/img/PostsImg/VolumeRenderingPractice/terrainWithTexture.png)

I can even use this texture as a height map
![pikachu map](/img/PostsImg/VolumeRenderingPractice/terrainPikachuHeightMap.png)
It looks strange. Howerver, it tells us that we can create more interesting effects as long as we spend more time on height map polishing. 
### Conclusion
It's the ending of series of remarching practice. I think it's a good start to learn skills about graphics. I hope I can find more fundamental and interesting topics about computer graphics.

*If you have anything recommand, you can intro it to me. I'm always willing to learn new things.* 
*Fantasy Wang*
*2018-04-02*