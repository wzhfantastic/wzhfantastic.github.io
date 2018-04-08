---
layout:     post
title:      Raytracing Practice In Unity(Part One)
subtitle:   Summary
date:       2018-04-08
author:     Fantasy Wang
header-img: img/home-bg-art.jpg.jpg
catalog: false
tags:
    - Unity
    - Rendering
    - Raytracing
    - Global Illumination
---
In this week, I researched [SmallPt](http://www.kevinbeason.com/smallpt/). It's a kind of Global Illumination by raytracing. So I use [Zavis's implementation](https://www.shadertoy.com/view/4sfGDB) as a reference and implement a Unity version of **SmallPt**. 
Here is my rendering result in one frame:
![RayTracing Result](/img/PostsImg/RayTracing/RayTracing.png)

##### Main Structure 
- On c# side, I manage all sphere objects in a ComputeBuffer:
```csharp
const int SphereCount = 9;
ComputeBuffer sceneBuffer;
Material material;
float Maxistance
{
    get { return (float)1e5; }
}
void Start()
{
    material = GetComponent<MeshRenderer>().sharedMaterial;
    InitSceneBuffer();
}
void InitSceneBuffer()
{
    if (sceneBuffer != null)
        sceneBuffer.Release();
    sceneBuffer = new ComputeBuffer(SphereCount, 4 + 12 + 12 + 12 + 4);
    Sphere[] spheres = new Sphere[SphereCount];
    spheres[0] = new Sphere(16.5f, new Vector3(27f, 16.5f, 57f), Vector3.zero, new Vector3(1,1,1), 1);
    spheres[1] = new Sphere(16.5f, new Vector3(73f, 16.5f, 78f), Vector3.zero, new Vector3(0.7f, 1f, 0.9f), 2);
    spheres[2] = new Sphere(600f , new Vector3(50f, 681.33f, 81.6f), new Vector3(12,12,12), Vector3.zero, 0);

    spheres[3] = new Sphere(Maxistance, new Vector3(-Maxistance+1, 40.8f, 81.6f), Vector3.zero, new Vector3(0.75f, 0.25f, 0.25f), 0);
    spheres[4] = new Sphere(Maxistance, new Vector3(-Maxistance + 99, 40.8f, 81.6f), Vector3.zero, new Vector3(0.25f, 0.25f, 0.75f), 0);
    spheres[5] = new Sphere(Maxistance, new Vector3(50, 40.8f, -Maxistance), Vector3.zero, new Vector3(0f, 0f, 0f), 0);
    spheres[6] = new Sphere(Maxistance, new Vector3(50, 40.8f, Maxistance + 170), Vector3.zero, new Vector3(0.75f, 0.75f, 0.75f), 0);
    spheres[7] = new Sphere(Maxistance, new Vector3(50, -Maxistance-9, 81.6f), Vector3.zero, new Vector3(0.75f, 0.75f, 0.75f), 0);
    spheres[8] = new Sphere(Maxistance, new Vector3(50, Maxistance + 81.6f, 81.6f), Vector3.zero, new Vector3(0.75f, 0.75f, 0.75f), 0);

    sceneBuffer.SetData(spheres);
    material.SetBuffer("spheres", sceneBuffer);
    material.SetInt("_ObjCount", SphereCount);
}
void OnDisable()
{
    if (sceneBuffer != null)
        sceneBuffer.Release();
    sceneBuffer = null;
}
```
- On shader side, I just do raytracing like [Zavie](https://www.shadertoy.com/view/4sfGDB) did in shadertoy.
##### Next
I will talk about some detail key technique points in following parts of my post series.