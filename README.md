# Maya Script: Create Locators and Joints on Ribbon

This Python script for Autodesk Maya automates the creation of locators and joints on a selected polygonal ribbon. It calculates face centers, generates groups, and pins locators to the ribbon using UVPin — streamlining the rigging process for ribbons like eyebrows or other surface-driven setups.

## Features

- **Face Center Calculation:** Determines the geometric center of each face on the selected ribbon.
- **Locator and Joint Creation:** Generates locators and joints for each face's center.
- **Grouping Structure:** Automatically organizes the following groups:
  - `*_Loc_Ribbon`: Contains all locators
  - `*_Jnt_Skin_MSH`: Contains all joints
  - `*_Ribbon`: Parent group containing both locator and joint groups
- **UV Pinning:** Pins each locator to the corresponding face using UVPin.
- **Parent Constraints:** Creates parent constraints between each locator and joint for automatic movement syncing.

## Requirements

- Autodesk Maya 2022 or newer

## Usage

1. Select the polygonal ribbon in your Maya scene.
2. Run the script in the Maya Script Editor.
3. The script will create locators, joints, and groups, then pin the locators to the ribbon's faces.

## Example

If you run the script on a ribbon named `EyeBrow`, the following structure will be created:

- `EyeBrow_Ribbon`
  - `EyeBrow_Loc_Ribbon`
    - `EyeBrow_Loc_000`
    - `EyeBrow_Loc_001`
    - ...
  - `EyeBrow_Jnt_Skin_MSH`
    - `EyeBrow_Jnt_000`
    - `EyeBrow_Jnt_001`
    - ...

## Code

```python
import maya.cmds as cmds

def get_face_center(face):
    """ Calculate the geometric center of a face using its vertices. """
    vertex_indices = cmds.polyInfo(face, faceToVertex=True)
    
    if not vertex_indices:
        return None
    
    indices = [int(i) for i in vertex_indices[0].split(":")[1].split()]
    positions = []
    
    for index in indices:
        vertex = f"{face.split('.')[0]}.vtx[{index}]"
        pos = cmds.xform(vertex, query=True, translation=True, worldSpace=True)
        positions.append(pos)
    
    center = [sum(coords) / len(positions) for coords in zip(*positions)]
    return center

def create_locators_and_joints_on_ribbon(ribbon_name):
    sel = cmds.ls(selection=True)
    
    if not sel:
        cmds.warning("Please select a ribbon (polygon object).")
        return
    
    ribbon = sel[0]
    faces = cmds.ls(f"{ribbon}.f[*]", flatten=True)
    
    if not faces:
        cmds.warning("No faces found on the selected object.")
        return
    
    locator_group = cmds.group(empty=True, name=f"{ribbon_name}_Loc_Ribbon")
    joint_group = cmds.group(empty=True, name=f"{ribbon_name}_Jnt_Skin_MSH")
    main_group = cmds.group(locator_group, joint_group, name=f"{ribbon_name}_Ribbon")
    
    for i, face in enumerate(faces):
        center = get_face_center(face)
        
        if center:
            locator_name = f"{ribbon_name}_Loc_{i:03d}"
            joint_name = f"{ribbon_name}_Jnt_{i:03d}"
            
            locator = cmds.spaceLocator(name=locator_name)[0]
            cmds.xform(locator, translation=center, worldSpace=True)
            cmds.parent(locator, locator_group)
            
            joint = cmds.joint(name=joint_name, position=center)
            cmds.parent(joint, joint_group)
            
            cmds.select(ribbon, locator, replace=True)
            cmds.UVPin()
            
            print(f"Locator {locator_name} and joint {joint_name} created and UV pinned")
            
            cmds.parentConstraint(locator, joint, maintainOffset=True)

create_locators_and_joints_on_ribbon("EyeBrow")
```

## Contributing

Contributions are welcome! Feel free to fork the repository and submit pull requests.

## License

This project is licensed under the MIT License. See the LICENSE file for details.

