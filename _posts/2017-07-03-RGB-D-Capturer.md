---
layout: default
title: Part 2. Building RGB-D Capturer
---

## Goal for part 2.
Capture both live RGB and Depth buffer simultaneously.

## Introduction
HoloLens doesn't provide raw data. It might be so that the raw data is preprocessed in HPU and CPU never gets to access it. So we will try to hacky way. We will get z-buffer from spatial mapping mesh objects. This is highly insufficient data and sounds un-right but that is what is assigned to be done.

I've been working on this since July 1st. There has been many problems:

* shader
* Unity

## Logs

### July 19th Wed.
#### 11:12 Update SpatialMapping from [Microsoft repo](https://github.com/Microsoft/HoloToolkit-Unity/commit/9ede36646cd5953485b1bc38bc1afc91f30c506c).

#### 13:50 [Create helper script for spatialmapping](https://github.com/YoungxHelsinki/HoloRGB-DCapturer/commit/3d88a1153e92c551d89e654fc2ab31545610d833).

 git filter-branch --force --index-filter \
'git rm -r --cached --ignore-unmatch Apps/' \
--prune-empty --tag-name-filter cat -- --all

#### 14:42 [Set tagalong depthbuffer display for debug purpose](https://github.com/jonathangranskog/HoloLensDepthPass/commit/d291126791e085f56bc0fb985088f4d826a043c4)
I found my working version. This version has:

* Main Camera
	* 2nd camera(for depth)
	* Display quad for debug
* SpatialMapping prefab

and it does:

* Creates spatial mapping meshes and applies defaultDiffuse material.
* Main cam excludes spatial mapping meshes from its culling mask.
* Saves both RGB & Depth.

### July 20th Thu.

#### 10:27 Found what caused error.
I was working on the [working version I found](https://github.com/jonathangranskog/HoloLensDepthPass/commit/d291126791e085f56bc0fb985088f4d826a043c4).

These are git diff outputs.
First, BlitDepth.cs, responsible for depth buffer capture.
```
diff --git a/Assets/Scripts/BlitDepth.cs b/Assets/Scripts/BlitDepth.cs
index 9fccec8..8abd969 100644
--- a/Assets/Scripts/BlitDepth.cs
+++ b/Assets/Scripts/BlitDepth.cs
@@ -19,7 +19,6 @@ public class BlitDepth : MonoBehaviour
     private Camera cam;
     private bool isTestMode = true;
     private int i = 1;
-    private DisplayAspectRatioControl darc;

     private Material _material;
     private Material material
@@ -41,7 +40,6 @@ public class BlitDepth : MonoBehaviour
         //cam = GameObject.FindWithTag("DepthCamera").GetComponent<Camera>();
         //tex = new RenderTexture(Camera.main.pixelWidth, Camera.main.pixelHeight, 32);
         tex = new RenderTexture(Camera.main.pixelWidth, Camera.main.pixelHeight, 32);
-        darc = quad.GetComponent<DisplayAspectRatioControl>();
         // Usually cameras render directly to screen, but for some effects it is useful
         // to make a camera render into a texture. This is done by creating a RenderTexture
         // object and setting it as targetTexture on the camera. The camera will then render
@@ -70,7 +68,10 @@ public class BlitDepth : MonoBehaviour

         } else
         {
-            //darc = DisplayAspectRatioControl();
+            DisplayAspectRatioControl darc;
+            darc = quad.GetComponent<DisplayAspectRatioControl>();
+            Texture2D t = new Texture2D(tex.width, tex.height, TextureFormat.ARGB32, false);
+            darc.SetDisplay(t);
         }

     }
@@ -181,7 +182,6 @@ public class BlitDepth : MonoBehaviour
             t.Apply();    // Unnecessary?
             if (isTestMode)
             {
-                darc.SetDisplay(t);
                 //quad.GetComponent<MeshRenderer>().sharedMaterial.mainTexture = t;
             } else
             {
```

Second, properties of Unity 2nd Camera gameobject and the "tex" which is attached to BlitDepth.cs.
```
diff --git a/Assets/test.unity b/Assets/test.unity
index 7fb2527..60c638f 100644
--- a/Assets/test.unity
+++ b/Assets/test.unity
@@ -473,6 +473,28 @@ Transform:
   m_Father: {fileID: 0}
   m_RootOrder: 4
   m_LocalEulerAnglesHint: {x: 0, y: 0, z: 0}
+--- !u!84 &1201656647
+RenderTexture:
+  m_ObjectHideFlags: 0
+  m_PrefabParentObject: {fileID: 0}
+  m_PrefabInternal: {fileID: 0}
+  m_Name:
+  m_ImageContentsHash:
+    serializedVersion: 2
+    Hash: 00000000000000000000000000000000
+  m_Width: 679
+  m_Height: 555
+  m_AntiAliasing: 1
+  m_DepthFormat: 2
+  m_ColorFormat: 0
+  m_MipMap: 0
+  m_GenerateMips: 1
+  m_SRGB: 0
+  m_TextureSettings:
+    m_FilterMode: 1
+    m_Aniso: 0
+    m_MipBias: 0
+    m_WrapMode: 1
 --- !u!1001 &1468380071
 Prefab:
   m_ObjectHideFlags: 0
@@ -927,7 +949,7 @@ Camera:
   m_TargetTexture: {fileID: 0}
   m_TargetDisplay: 0
   m_TargetEye: 3
-  m_HDR: 1
+  m_HDR: 0
   m_OcclusionCulling: 1
   m_StereoConvergence: 10
   m_StereoSeparation: 0.022
@@ -945,30 +967,8 @@ MonoBehaviour:
   m_EditorClassIdentifier:
   depthLevel: 0.5
   quad: {fileID: 318272723}
-  tex: {fileID: 2007647041}
+  tex: {fileID: 1201656647}
   shader: {fileID: 4800000, guid: 47eef91b366cadc4eb5407aaa15ca13f, type: 3}
---- !u!84 &2007647041
-RenderTexture:
-  m_ObjectHideFlags: 0
-  m_PrefabParentObject: {fileID: 0}
-  m_PrefabInternal: {fileID: 0}
-  m_Name:
-  m_ImageContentsHash:
-    serializedVersion: 2
-    Hash: 00000000000000000000000000000000
-  m_Width: 1207
-  m_Height: 560
-  m_AntiAliasing: 1
-  m_DepthFormat: 2
-  m_ColorFormat: 0
-  m_MipMap: 0
-  m_GenerateMips: 1
-  m_SRGB: 0
-  m_TextureSettings:
-    m_FilterMode: 1
-    m_Aniso: 0
-    m_MipBias: 0
-    m_WrapMode: 1
 --- !u!1 &2126488711
 GameObject:
   m_ObjectHideFlags: 0
```

Instead of trying to fix a new architecture based on old working architecture, I will try to improve the old working architecture as the new architecture.

#### [Set display size at start not in every update](https://github.com/jonathangranskog/HoloLensDepthPass/commit/125af642f35fb6a5910165140d6874e42df80736)
I found one important thing. Unity HoloLens emulator is broken. Also, VS 2015 build is unstable. Test always on-device and multiple times.

#### [Add KeywordRecognizer for voice command](https://github.com/jonathangranskog/HoloLensDepthPass/commit/ee8b3160c30d1cc7a96ac0a7988ecbed664ba3f1)
Added voice command feature in order to change mesh material and culling mask. It turns out that you can get depth buffer out of wireframe as well.
However, the depth buffer is rendered so slowly that it lags a lot. 	


#### 16:11 HORRIBLE VS or Unity errors
```
Severity	Code	Description	Project	File	Line	Suppression State
Error	CS0006	Metadata file 'C:\Users\YOUNG\CODE\HoloLensDepthPass\Apps\GeneratedProjects\UWP\Assembly-CSharp\bin\x86\Release\Assembly-CSharp.dll' could not be found	UnityVersionTest	C:\Users\YOUNG\CODE\HoloLensDepthPass\Apps\UnityVersionTest\CSC	1	Active
```
```
Severity	Code	Description	Project	File	Line	Suppression State
Error		DEP2100 : Cannot copy the file "C:\Users\YOUNG\CODE\HoloLensDepthPass\App\UnityVersionTest\bin\x86\Release\resources.pri" to the target machine "127.0.0.1". 
The requested operation cannot be performed on a file with a user-mapped section open. (Exception from HRESULT: 0x800704C8)	UnityVersionTest			
```
They were horrible because it was impossible to reason the errors. Deleting `.sln` file solved the issues.


#### 16:33 Unity asset corruption.

Commit: 8d8f0b7b97e4ad0d07255c6b1d57704f4c04d366
```
--- !u!20 &1668346400
Camera:
  m_ObjectHideFlags: 0
  m_PrefabParentObject: {fileID: 0}
  m_PrefabInternal: {fileID: 0}
  m_GameObject: {fileID: 1668346395}
  m_Enabled: 1
  serializedVersion: 2
  m_ClearFlags: 2
  m_BackGroundColor: {r: 0, g: 0, b: 0, a: 0}
  m_NormalizedViewPortRect:
    serializedVersion: 2
    x: 0
    y: 0
    width: 1
    height: 1
  near clip plane: 0.3
  far clip plane: 10
  field of view: 60
  orthographic: 0
  orthographic size: 5
  m_Depth: 1
  m_CullingMask:
    serializedVersion: 2
    m_Bits: 4294967039
  m_RenderingPath: -1
  m_TargetTexture: {fileID: 0}
  m_TargetDisplay: 0
  m_TargetEye: 3
  m_HDR: 1
  m_OcclusionCulling: 1
  m_StereoConvergence: 10
  m_StereoSeparation: 0.022
  m_StereoMirrorMode: 0
--- !u!28 &1855520762
Texture2D:
  m_ObjectHideFlags: 0
  m_PrefabParentObject: {fileID: 0}
  m_PrefabInternal: {fileID: 0}
  m_Name:
  m_ImageContentsHash:
    serializedVersion: 2
    Hash: 00000000000000000000000000000000
  serializedVersion: 2
  m_Width: 256
  m_Height: 256
  m_CompleteImageSize: 262144
  m_TextureFormat: 5
  m_MipCount: 1
  m_IsReadable: 1
  m_AlphaIsTransparency: 0
  m_ImageCount: 1
  m_TextureDimension: 2
  m_TextureSettings:
    m_FilterMode: 1
    m_Aniso: 1
    m_MipBias: 0
    m_WrapMode: 0
  m_LightmapFormat: 0
  m_ColorSpace: 1
  image data: 262144
  _typelessdata: fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
```
I checked out all the commits in my git history and apparently it has been corrupted since the beginning. 

#### 17:00 Wait it's not a corruption!
According to an [Uniti Forum](https://forum.unity3d.com/threads/typelessdata-in-asset-file.277441/), that weird long data represent [Special array for large byte arrays](https://github.com/ata4/disunity/wiki/Serialized-file-format). [Check your asset serialization mode](https://docs.unity3d.com/Manual/class-EditorManager.html).


### July 28th 

**Problem**: depth image doesn't seem to represent the actual image well.

**Why?**
1. Spatial Mapping module is not doing its job.
2. Depth image generation is computationally intensive. Map and Capture in series not simultaneously.

#### 19:45 [Allow Surface update by the system](https://github.com/jonathangranskog/HoloLensDepthPass/commit/2072f1a8e90827539e93ba9f78a4608753a76cf5)
It turns out that SpatialMapping module I was using was not a clean version from MS but a hacky version I used for another project.
**Result**: Updates surface by the system.
**Problem**: The depth image doesn't match meshes. They seem to have lost an anchor.
**Try**: Add pause command that pauses spatial mapping observer and depth image generator.

#### 20:45 [Add pause/resume command for mapping & capturing](https://github.com/jonathangranskog/HoloLensDepthPass/commit/ca2f873af5aaa28f93c81662af899e6efb6844f9)
**Result**: The depth image clearly doesn't match meshes.
**Problem**: I have no idea why this is happening...
**Try**: Capture only on voice command.

#### 21:45 [Capture buffer&image on voice command](https://github.com/jonathangranskog/HoloLensDepthPass/commit/e936cad11a4f65a11d9e1ceccaba824d4657600e)
**Result**: The depth image clearly doesn't match meshes. I get images both buffer and RGB. The z-buffer camera seems to move in a funny way. See images at https://drive.google.com/drive/folders/0B0Fa41V9uAI3R2g5d2lBUjhqSzg?usp=sharing
**Problem**: I have no idea why this is happening...
**Try**: ??










