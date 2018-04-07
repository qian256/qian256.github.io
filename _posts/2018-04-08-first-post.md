---
layout: post
title: First post
description: Welcome post
tags: welcome w2
---

I would welcome you all to my blog hosted on Github.


The above is a screenshot of a Unity transformation. The x, y, z axis directions are visualized with different colors, and are following left-land rules. In addition, rotation along axis is applied clockwise, unlike counter-clockwise in right-hand coordinate systems. If you are accustomed to right-hand coordinate systems as I do, please pay more attention when working with raw 4x4 matrices.
Coordinate System of ARToolKit

ARToolKit uses right-hand coordinate systems. Since OpenGL is right-handed as well, it is very comfortable to visualize the tracking result of ARToolKit by OpenGL.

In the official Unity package of ARToolKit, arunity, there exists a utility function

public static Matrix4x4 LHMatrixFromRHMatrix(Matrix4x4 rhm)

that converts the tracking result of ARToolKit to Unity environment.
Coordinate System of HoloLensARToolKit

It took me a while to figure out the complicated conversions happened inside ARToolKit, inside Unity, and in-between. Actually in HoloLensARToolKit v0.1, the coordinate system is not very clear nor consistent between different kinds of markers. In the current version v0.2, the design of coordinate system is improved, and a special Unity scene is designed to help visualize the coordinate system associated with the marker.

In this project, the tracking result is converted into left-handed coordinate system, with y-axis flipped. This is performed in the function ARUWPUtils.ConvertARUWPFloatArrayToMatrix4x4(). When the tracking result is applied to a Visualization Target of the current marker, there is no need to worry about any conversion.

The following examples show the coordinate system associated with each kind of marker.
Single Pattern Marker

Hiro marker and Kanji marker are examples 

Thanks for reading!
