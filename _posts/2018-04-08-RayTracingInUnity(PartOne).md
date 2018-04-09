---
layout:     post
title:      Raytracing Practice In Unity(Part One)
subtitle:   Summary
date:       2018-04-08
author:     Fantasy Wang
header-img: img/home-bg-art.jpg
catalog: true
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
```c
Shader "Custom/RayTracing" {
	Properties
    {
		_ObjCount("Object Count", int) = 1
    }
    SubShader  
    {    
		Tags {"Queue" = "Transparent" "RenderType" = "Transparent"}

		Blend SrcAlpha OneMinusSrcAlpha 

		Pass  
		{  
			CGPROGRAM  
			#pragma target 5.0

			#pragma vertex vert  
			#pragma fragment frag  
  
			#include "UnityCG.cginc"  

			#define SAMPLES 6
			#define MAXDEPTH 4
			#define ENABLE_NEXT_EVENT_PREDICTION

			#define PI 3.14159265359 
			#define DIFF 0
			#define SPEC 1
			#define REFR 2

			float seed;
			float rand() 
			{ 
				seed = seed + 1;
				return frac(sin(seed)*43758.5453123); 
			}
			
			struct Ray
			{
				float3 o;
				float3 d;
			};

			struct Sphere
			{
				float r;
				float3 p;
				float3 e;   //emission
				float3 c;   //color
				int refl; //DIFF, SPEC, REFR
			};

			StructuredBuffer<Sphere> spheres;
			int _ObjCount;

			float intersect(Sphere s, Ray r) {
				float3 op = s.p - r.o;
				float t, epsilon = 1e-3, b = dot(op, r.d), det = b * b - dot(op, op) + s.r * s.r;
				if (det < 0.) return 0.; else det = sqrt(det);
				return (t = b - det) > epsilon ? t : ((t = b + det) > epsilon ? t : 0.);
			}

			int intersect(Ray r, out float t, out Sphere s, int avoid)
			{
				int id = -1;
				t = 1e5;
				s = spheres[0];

				for (int i = 0; i < _ObjCount; ++i) {
					Sphere S = spheres[i];
					float d = intersect(S, r);
					if (i!=avoid && d!=0. && d<t) { t = d; id = i; s=S; }
				}
				return id;
			}

			float3 jitter(float3 d, float phi, float sina, float cosa) {
				float3 w = normalize(d), u = normalize(cross(w.yzx, w)), v = cross(w, u);
				return (u*cos(phi) + v*sin(phi)) * sina + w * cosa;
			}

			float3 radiance(Ray r) {
				float3 acc = float3(0,0,0);
				float3 mask = float3(1,1,1);
				int id = -1;
				for (int depth = 0; depth < MAXDEPTH; ++depth) {
					float t;
					Sphere obj;
					if ((id = intersect(r, t, obj, id)) < 0) break;
					float3 x = t * r.d + r.o;
					float3 n = normalize(x - obj.p), nl = n * sign(-dot(n, r.d));

					//float3 f = obj.c;
					//float p = dot(f, float3(1.2126, 0.7152, 0.0722));
					//if (depth > DEPTH_RUSSIAN || p == 0.) if (rand() < p) f /= p; else { acc += mask * obj.e * E; break; }

					if (obj.refl == DIFF) {
						float r2 = rand();
						float3 d = jitter(nl, 2.*PI*rand(), sqrt(r2), sqrt(1. - r2));
						float3 e = float3(0, 0, 0);
			#ifdef ENABLE_NEXT_EVENT_PREDICTION
						//for (int i = 0; i < NUM_SPHERES; ++i)
						{
							// Sphere s = sphere(i);
							// if (dot(s.e, float3(1.)) == 0.) continue;

							// Normally we would loop over the light sources and
							// cast rays toward them, but since there is only one
							// light source, that is mostly occluded, here goes
							// the ad hoc optimization:
							Sphere s =  {20, float3(50, 81.6, 81.6), float3(12, 12, 12), float3(0,0,0), DIFF};
							int i = 2;

							float3 l0 = s.p - x;
							float cos_a_max = sqrt(1. - clamp(s.r * s.r / dot(l0, l0), 0., 1.));
							float cosa = lerp(cos_a_max, 1., rand());
							float3 l = jitter(l0, 2.*PI*rand(), sqrt(1. - cosa*cosa), cosa);

							Ray rTemp = {x, l};
							if (intersect(rTemp, t, s, id) == i) {
								float omega = 2. * PI * (1. - cos_a_max);
								e += (s.e * clamp(dot(l, n),0.,1.) * omega) / PI;
							}
						}
			#endif
						float E = 1.;//float(depth==0);
						acc += mask * obj.e * E + mask * obj.c * e;
						mask *= obj.c;
						Ray rTemp = {x, d};
						r = rTemp;
					} else if (obj.refl == SPEC) {
						acc += mask * obj.e;
						mask *= obj.c;
						Ray rTemp = {x, reflect(r.d, n)};
						r = rTemp;
					} else {
						float a=dot(n,r.d), ddn=abs(a);
						float nc=1., nt=1.5, nnt=lerp(nc/nt, nt/nc, float(a>0.));
						float cos2t=1.-nnt*nnt*(1.-ddn*ddn);
						Ray rTemp = {x, reflect(r.d, n)};
						r = rTemp;
						if (cos2t>0.) {
							float3 tdir = normalize(r.d*nnt + sign(a)*n*(ddn*nnt+sqrt(cos2t)));
							float R0=(nt-nc)*(nt-nc)/((nt+nc)*(nt+nc)),
								c = 1.-lerp(ddn,dot(tdir, n),float(a>0.));
							float Re=R0+(1.-R0)*c*c*c*c*c,P=.25+.5*Re,RP=Re/P,TP=(1.-Re)/(1.-P);
							if (rand()<P) { mask *= RP; }
							else { mask *= obj.c*TP; Ray rTemp = {x, tdir}; r = rTemp; }
						}
					}
				}
				return acc;
			}

			struct appdata  
			{  
				float4 vertex : POSITION;  
			};  
  
			struct v2f  
			{  
				float4 vertex : SV_POSITION;  
				float3 worldPos : TEXCOORD1;  
			};  
  
			v2f vert(appdata v)  
			{  
				v2f o;  
				o.vertex = UnityObjectToClipPos(v.vertex);  
				o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
				return o;  
			}  
			
			float4 frag(v2f input) : SV_Target  
			{   
				seed = input.worldPos.x + input.worldPos.y + _Time.x;
				float3 viewDirection = normalize(input.worldPos - _WorldSpaceCameraPos);
				Ray ray = {input.worldPos,viewDirection};
				float3 color = float3(0,0,0);
				for (int i = 0; i < SAMPLES; ++i)
				{
					color += radiance(ray);
				}

				return float4(pow (saturate(color / SAMPLES), 1/2.2),1);
			}  
			ENDCG  
		}  
    }  
	FallBack "Diffuse"
}
```

##### Next
I will talk about some detail key technique points in following parts of my post series.