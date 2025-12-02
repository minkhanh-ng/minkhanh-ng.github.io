---
title: 'Custom Python Scripts for AutoCAD Plant 3D - Case study of tubing fittings'
date: 2025-08-14
permalink: /posts/2025/08/Custom_Python_Scripts_for_AutoCAD_Plant_3D_-_Case_study_of_tubing_fittings/
tags:
  - AutoCAD Plant 3D
  - Python
  - Parametric Modeling
  - Swagelok
  - CAD
---

Let's try creating a simple male connector for AutoCAD Plant 3D (PLNT3D). First, we'll grasp a drawing from Swagelok as a reference:

![Swagelok Male Connector Reference](https://enginine.wordpress.com/wp-content/uploads/2025/11/image.png?w=966)

## 1. Coding the Geometry

We import libraries in the header, then define the part's principal dimensions which are used to drive the entire model, serving as user inputs.

```python
from varmain.primitiv import *
from varmain.custom import *
 
@activate(Group="Coupling", TooltipShort="Tube Male Connector TubexMPT", TooltipLong="Tube Male Connector TubexMPT", FirstPortEndtypes="P", LengthUnit="in",  Ports="2")
@group("MainDimensions")
@param(D=LENGTH, TooltipShort="Tube OD", TooltipLong="Tube OD")
@param(D1=LENGTH, TooltipShort="Hex head OD", TooltipLong="Fitting hex head OD")
@param(D31=LENGTH, TooltipShort="Hex OD of Body", TooltipLong="Hex OD of Body")
@param(L=LENGTH, TooltipShort="Length of Fitting", TooltipLong="Overall Length of Fitting")
@param(L31=LENGTH, TooltipShort="Tube gland", TooltipLong="Tube gland length")
@param(I1=LENGTH, TooltipShort="Tube insert", TooltipLong="Tube insert distance")
@param(OF=LENGTH0)
```

Next, we define the function signature:

```python
def maleConnector(s, D=0.25, D1=0.5625, D31=1.0625, L=1.82, L31=0.6, I1=0, K=1, OF=0, **kw):
```

### The Geometry Strategy

As AutoCAD Plant 3D only accepts a few primitives, we have to be creative in our coding.

The Hex nut shape will be treated as a cylinder being trimmed by six cubes, followed by deleting these cubes.

```python
    # hex head
    headStartangle = 0
    headBox1 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle) #cos(30deg) = 0.866
    headBox2 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle+60) 
    headBox3 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle+2*60) 
    headBox4 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle+3*60) 
    headBox5 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle+4*60) 
    headBox6 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle+5*60) 
    
    headBox1.uniteWith(headBox2)
    headBox1.uniteWith(headBox3)
    headBox1.uniteWith(headBox4)
    headBox1.uniteWith(headBox5)
    headBox1.uniteWith(headBox6)
    
    headBox2.erase()
    headBox3.erase()
    headBox4.erase()
    headBox5.erase()
    headBox6.erase()
 
    head1 = CYLINDER(s, R=D1/2, H=L32, O=0).rotateY(90)
    head1.subtractFrom(headBox1)
    headBox1.erase()
```

After creating other shapes (like the body and the second hex), we combine them into one body using `uniteWith`.

### Port Definition

We need to create the insert point and port points. As the tube will penetrate these fittings in the tubing port, we have to offset one port inwardly.

```python
    # tube insert distance
    if I1 == 0:
        I1 = L31
     
    # set port points
    s.setPoint((0.0 + I1, 0.0, 0.0), (-1.0, 0.0, 0.0), 0)
    s.setPoint((L, 0.0, 0.0), (1.0, 0.0, 0.0), 0)
```

## 2\. Testing the Script

1.  **Locate Folder:** Choose the Share Content folder by command `MODIFYSHAREDCONTENTFOLDER` or use the default location:  
    `C:\AutoCAD Plant 3D 2016 Content\CPak Common\CustomScripts`
    Save your python script file at that location.
2.  **Compile:** Run command `PLANTREGISTERCUSTOMSCRIPTS` to compile the script.
3.  **Load Adapter:** Load the PnP3dACPAdapter.arx by running `arxload "PnP3dACPAdapter"`.
4.  **Test:** Run `testacpscript "maleConnector"`.

> **Note:** If you make any modifications to the code, PLNT3D must be closed and restarted, then you must register the script again.

## 3\. Catalog Integration

After testing the script and generating the geometry, our job is to publish it into the catalog.

### Image Creation

Manually `dim` (dimension) all the necessary parts and change the CAD environment to a white background.

![CAD Dimensioning](https://enginine.wordpress.com/wp-content/uploads/2025/11/6a0167607c2431970b01bb084f4db5970d.png?w=594)

Capture the image using an app like PicPick or Cropper to create three images with different sizes:
* `maleConnector_64.png` (64x64)
* `maleConnector_200.png` (200x200)
* `maleConnector_640.png` (640x640)

![Thumbnail Examples](https://enginine.wordpress.com/wp-content/uploads/2025/11/6a0167607c2431970b01bb084f4db5970dx.png?w=313)

### Part Creation in Spec Editor

Final step is to launch Spec Editor.
1.  Open an existing CustomCatalog (this is often better than creating a whole new catalog using CatalogBuilder).
2.  Choose "Create New Component".
3.  Select our part by choosing the **Group** defined in our Python script (`Coupling`).

![Spec Editor Selection](https://enginine.wordpress.com/wp-content/uploads/2025/11/6a0167607c2431970b01bb084f4db5970d3.png?w=834)

Setting and save the catalog. Now it ready-to-use in Plant3D.

![Catalog View](https://enginine.wordpress.com/wp-content/uploads/2025/11/6a0167607c2431970b01bb084f4db5970d4.png?w=1024)

![Final Result](https://enginine.wordpress.com/wp-content/uploads/2025/11/6a0167607c2431970b01bb084f4db5970d5.png?w=1024)

-----

## Appendix: The Full Script

Here is the complete code for the Male Connector.

```python
from varmain.primitiv import *
from varmain.custom import *
 
@activate(Group="Coupling", TooltipShort="Tube Male Connector TubexMPT", TooltipLong="Tube Male Connector TubexMPT", FirstPortEndtypes="P", LengthUnit="in",  Ports="2")
@group("MainDimensions")
@param(D=LENGTH, TooltipShort="Tube OD", TooltipLong="Tube OD")
@param(D1=LENGTH, TooltipShort="Hex head OD", TooltipLong="Fitting hex head OD")
@param(D31=LENGTH, TooltipShort="Hex OD of Body", TooltipLong="Hex OD of Body")
@param(L=LENGTH, TooltipShort="Length of Fitting", TooltipLong="Overall Length of Fitting")
@param(L31=LENGTH, TooltipShort="Tube gland", TooltipLong="Tube gland length")
@param(I1=LENGTH, TooltipShort="Tube insert", TooltipLong="Tube insert distance")
@param(OF=LENGTH0)
 
#(arxload "PnP3dACPAdapter")
#(testacpscript "maleConnector")
def maleConnector(s, D=0.25, D1=0.5625, D31=1.0625, L=1.82, L31=0.6, I1=0, K=1, OF=0, **kw):
    #length
    L32 = L31*0.6 #adjusting the length of hex head
    L33 = L - L31 #adjusting the length of hex head
    L1 = 0.2 #adjusting the length of body
        
    #hex head
    headStartangle = 0
    headBox1 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle) #cos(30deg) = 0.866
    headBox2 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle+60) 
    headBox3 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle+2*60) 
    headBox4 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle+3*60) 
    headBox5 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle+4*60) 
    headBox6 = BOX(s, L=D1, W=D1, H=L32).translate((L32/2,0,1.866*D1/2)).rotateX(headStartangle+5*60) 
    
    headBox1.uniteWith(headBox2)
    headBox1.uniteWith(headBox3)
    headBox1.uniteWith(headBox4)
    headBox1.uniteWith(headBox5)
    headBox1.uniteWith(headBox6)
    
    headBox2.erase()
    headBox3.erase()
    headBox4.erase()
    headBox5.erase()
    headBox6.erase()
 
    head1 = CYLINDER(s, R=D1/2, H=L32, O=0).rotateY(90)
    head1.subtractFrom(headBox1)
    headBox1.erase()
 
    #body create
    bodyMain = CYLINDER(s, R=0.8*D1/2, H=L, O=0).rotateY(90)
 
    # hex head second
    head2Startangle = 0
    head2Box1 = BOX(s, L=D31, W=D31, H=L33*0.2).translate((L33*0.2/2+L31,0,1.866*D31/2)).rotateX(head2Startangle) #cos(30deg) = 0.866
    head2Box2 = BOX(s, L=D31, W=D31, H=L33*0.2).translate((L33*0.2/2+L31,0,1.866*D31/2)).rotateX(head2Startangle+60) 
    head2Box3 = BOX(s, L=D31, W=D31, H=L33*0.2).translate((L33*0.2/2+L31,0,1.866*D31/2)).rotateX(head2Startangle+2*60) 
    head2Box4 = BOX(s, L=D31, W=D31, H=L33*0.2).translate((L33*0.2/2+L31,0,1.866*D31/2)).rotateX(head2Startangle+3*60) 
    head2Box5 = BOX(s, L=D31, W=D31, H=L33*0.2).translate((L33*0.2/2+L31,0,1.866*D31/2)).rotateX(head2Startangle+4*60) 
    head2Box6 = BOX(s, L=D31, W=D31, H=L33*0.2).translate((L33*0.2/2+L31,0,1.866*D31/2)).rotateX(head2Startangle+5*60) 
    
    head2Box1.uniteWith(head2Box2)
    head2Box1.uniteWith(head2Box3)
    head2Box1.uniteWith(head2Box4)
    head2Box1.uniteWith(head2Box5)
    head2Box1.uniteWith(head2Box6)
    
    head2Box2.erase()
    head2Box3.erase()
    head2Box4.erase()
    head2Box5.erase()
    head2Box6.erase()
 
    head2 = CYLINDER(s, R=D31/2, H=L33*0.2, O=0).rotateY(90).translate((L31,0,0))
    head2.subtractFrom(head2Box1)
    head2Box1.erase()
 
    head2F = CONE(s, R1=D31/2*0.85, R2=D31/2*0.7, H=L33, E=0).rotateY(90).translate((L31,0,0))
    head2.uniteWith(head2F)
    head2F.erase()
 
    #joint 3 parts
    bodyMain.uniteWith(head1)
    bodyMain.uniteWith(head2)
    head1.erase()
    head2.erase()
 
    #create a hole fitting
    boreCy = CYLINDER(s, R=D/2, H=L, O=0).rotateY(90)
    bodyMain.subtractFrom(boreCy)
    boreCy.erase()
 
    #tube insert distance
    if I1 == 0:
        I1 = L31
    
    #set port points
    s.setPoint((0.0 + I1, 0.0, 0.0), (-1.0, 0.0, 0.0), 0)
    s.setPoint((L, 0.0, 0.0), (1.0, 0.0, 0.0), 0)
    
    return
```

