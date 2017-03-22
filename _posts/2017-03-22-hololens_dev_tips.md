---
layout: default
title: HoloLens Dev tips
---


This is a HoloLens development documentation for the project.

---

## [Getting started with HoloToolkit](https://forums.hololens.com/discussion/2364/getting-started-with-holotoolkit)
Here are the steps that you currently need to take to run sample projects in HoloToolkit-Unity:
1. Clone the HoloToolkit-Unity project from GitHub to your PC.
2. Launch Unity and select 'Open'
3. Select the 'HoloToolkit-Unity' folder on your PC to open the project in Unity
4. In the Project panel, expand the HoloToolkit folder
5. Locate the feature that you are interested in (we'll use SpatialMapping as an example)
6. Expand SpatialMapping feature and then select the 'Tests' folder.
7. Double-click on one of the test scenes to open it (I would suggest trying the 'SpatialProcessing' scene, since it can run inside of Unity).
8. Some scenes (like 'SpatialProcessing') can run inside of Unity, so you just need to press the 'Play' button. All scenes will run on the HoloLens, but you need to follow the build/deploy process outlined in our docs and Academy courses.
9. If you're confused about what a test does, locate the ReadMe file associated with the feature (view it on GitHub for nice formatting, or find and double-click it under the feature-level folder in Unity). There should be instructions/explanations for each test scene.

If you want to try more end-to-end samples, then there are several available under the 'HoloToolkit-Examples' folder. To run the SpatialMappingComponent example, you would do the following:
1. Expand the 'HoloToolkit-Examples' folder.
2. Click on the 'SpatialMappingComponent' folder.
3. Double-click on the 'Example' scene to load it.
4. This sample must run on the HoloLens, so build/deploy as usual (In the Build Settings window, don't forget to press the 'AddOpenScenes' button to add the current scene to the build and delete any scene from the list that you may have added before).
5. There should be a ReadMe file associated with each example to help explain what is going on.

If you get to the point where you want to import HoloToolkit into your own Unity project, then you can do the following:
1. In the HoloToolkit-Unity project, right-click on the 'HoloToolkit' folder and select 'Export package'.
2. Wait a few seconds for the export dialog to populate...
3. You can create a custom package by unchecking any folders/scripts that you do not want to export. For example, if you don't need any of the 'Build' 'Cross Platform' 'Sharing' and 'Spatial Sound' components, then uncheck those folders. It's a good idea to uncheck all of the 'test' folders under each feature too, and make sure that the 'HoloToolkit-Examples' folder is not included so you don't get test scenes added to your project.
4. Press the 'Export' button.
5. Save the .unitypackage
6. Open your personal project in Unity or create a new project.
7. Select the 'Assets' file menu option at the top of Unity
8. Select 'Import Package' > 'Custom Package' and then find and select the .unitypackage that you saved in step 5 above. Press the 'Open' button.
9. If there are any files that you don't want to import into your project, you can uncheck them now, otherwise, press the 'Import' button.
10. The 'HoloToolkit' folder should now appear under the 'Assets' folder in the Project panel.



## Unity settings for HoloLens
### 1. Player settings
1. From the Build Settings... window, open Player Settings...
2. Select the Settings for Windows Store tab
3. Expand the Other Settings group
4. In the Rendering section, check the Virtual Reality Supported checkbox to add a new Virtual Reality Devices list and confirm "Windows Holographic" is listed as a supported device.

### 2. Build Settings
1. Select File > Build Settings...
2. Select Windows Store in the Platform list.
3. Set SDK to Universal 10
4. Set Build Type to D3D.

### 3. Performance settings
![UnityQualitySettings](/images/UnityQualitySettings.PNG)
1. Select Edit > Project Settings > Quality
2. Select the dropdown under the Windows Store logo and select Fastest. You'll know the setting is applied correctly when the box in the Windows Store column and Fastest row is green.

## Visual Studio settings
### Edit `Package.appxmanifest` in VS.
`TargetDeviceFamily Name` & `MaxVersion`.

Example:
```XML
<Dependencies>
    <TargetDeviceFamily Name="Windows.Holographic" MinVersion="10.0.10240.0" MaxVersionTested="10.0.10586.0" />
</Dependencies>
```

### VS Build settings
![build_settings](/images/VS_build_settings.PNG)


## Mirroring/Accessing HoloLens
1. [Hololens app on Windows 10](https://www.microsoft.com/en-ca/store/p/microsoft-hololens/9nblggh4qwnx)
2. From a web browser on your PC, go to https://<YOUR_HOLOLENS_IP_ADDRESS>
  * The browser will display the following message: "There’s a problem with this website’s security certificate". This happens because the certificate which is issued to the Device Portal is a test certificate. You can ignore this certificate error for now and proceed.


## "My cursor is not showing!" or "My C# scripts are not working" or VS fails to build/attach debugger.

![managers_inspector](/images/managers_inspector.PNG)

_< You MUST add-component your scripts to corresponding objects in Unity in order to let Unity use the scripts. >_

## Build fail in Unity
* Error: `missing path \UnionMetadata\Facade\Windows.winmd`
* [Install Windows 10 SDK](http://answers.unity3d.com/questions/1116443/windows-sdk-installed-but-get-error.html)

## Deploy fail in VS
* Error: `Metadata file '.dll' could not be found`
* [Solution](http://stackoverflow.com/a/17723774):
  1. Right click on the solution and click Properties.
  2. Click Configuration on the left.
  3. Make sure the check box under "Build" for the project it can't find is checked. If it is already checked, uncheck, hit apply and check the boxes again.
