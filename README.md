# Flexible Grid Controller <-> Turn Based Strategy Framework integration

## Table of Contents
1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Integration Guide](#integration-guide)
4. [Contact and Support](#contact-and-support)

## Introduction
This repository demonstrates the integration between the two Unity Asset Store projects: the Flexible Grid Controller and the Turn Based Strategy Framework. The Framework is a flexible tool for developing turn-based strategy games in Unity and is available in the [Unity Asset Store](http://u3d.as/mfd). The Flexible Grid Controller facilitates integrating various input methods, such as keyboard or gamepad into any grid-based project. It is a great match for the Turn Based Strategy Framework, but it is not limited to it.

## Getting Started
### Prerequisites
- Unity version 2021.3.11f1 or higher
- Turn Based Strategy Framework version 3.0.2 or higher
- Flexible Grid Controller version 1.0.1 or higher

### Installing Turn Based Strategy Framework
1. If you haven't already, purchase the Turn Based Strategy Framework from the [Unity Asset Store](http://u3d.as/mfd).
2. Import it into your Unity project following the Asset Store's instructions.

### Installing Flexible Grid Controller
1. If you haven't already, purchase the Flexible Grid Controller from the [Unity Asset Store](https://u3d.as/3dcm).
2. Import it into your Unity project following the Asset Store's instructions.

### Installing the package
1. Download the package from the 'Releases' tab
2. Import the package into your project

### Package Structure
The package content is organized as follows:
```
├── FGC_TBSF_Integration
│   ├── Resources
│   ├── Scenes
│   └── Scripts
```
The `Scenes` folder includes a playable demo, showcasing a level from the Turn Based Strategy Framework with gamepad and keyboard controls integrated with the help of Flexible Grid Controller. The `Scripts` folder contains all custom scripts that the package requires. Finally, the `Resources` folder holds some sprites used in the UI.

## Integration Guide
The baseline that gets keyboard controls integrated into it is the Turn Based Strategy Framework Example1 demo scene. The scene consists of a hexagonal grid and some units for the player to control. Please check out the [Flexible Grid Controller docs](https://github.com/mzetkowski/flexible-grid-controller-docs) first to understand the integration process better, as I only explain the choices specific to this project here.

The key decision in integrating Flexible Grid Controller into any project is selecting the right `Interactable` interface to implement. Two factors come into play here, the grid layout and the actions that should be enabled on the tiles. In case of the Example1 demo scene, the grid is two dimensional and the tiles support selection and clicking. The `IBasicInteractable2D` interface fits these requirements.

The interface is implemented as a separate component that will be added to the tile prefab. The implemented methods, `OnClick`, `OnSelected` and `OnDeselected` simply forward the actions to methods already existing on the underlying Cell. The `GetGridIndex` references the Cell's OffsetCoords field to determine its position within the grid.

```
public class CellInteractable : MonoBehaviour, IBasicInteractable2D
{
    private Cell _cellReference;

    public UnityEvent<IBasicInteractable2D, BasicControllable2D> Selected { get; set; } = new UnityEvent<IBasicInteractable2D, BasicControllable2D>();
    public UnityEvent<IBasicInteractable2D, BasicControllable2D> Deselected { get; set; } = new UnityEvent<IBasicInteractable2D, BasicControllable2D>();
    public UnityEvent<IBasicInteractable2D, BasicControllable2D> Clicked { get; set; } = new UnityEvent<IBasicInteractable2D, BasicControllable2D>();

    private void Awake()
    {
        _cellReference = GetComponent<Cell>(); 
    }

    public GridIndex2D GetGridIndex(BasicControllable2D controllable)
    {
        // Grid Index is given by referencing underlying cell's Offset Coords
        return new GridIndex2D((int)_cellReference.OffsetCoord.x, (int)_cellReference.OffsetCoord.y);
    }

    public void OnClicked(BasicControllable2D controllable)
    {
        if(_cellReference.CurrentUnits.Count > 0) 
        {
            _cellReference.CurrentUnits[0].OnMouseDown();
        }
        else
        {
            _cellReference.OnMouseDown();
        }
    }

    public void OnDeselected(BasicControllable2D controllable)
    {
        _cellReference.OnMouseExit();
        if (_cellReference.CurrentUnits.Count > 0)
        {
            _cellReference.CurrentUnits[0].OnMouseExit();
        }
    }

    public void OnSelected(BasicControllable2D controllable)
    {
        _cellReference.OnMouseEnter();
        if (_cellReference.CurrentUnits.Count > 0)
        {
            _cellReference.CurrentUnits[0].OnMouseEnter();
        }
    }
}
```
The choice of the `IBasicInteractable2D` interface determines subsequent choices: `GridControllable2D` and subsequently the `BasicGridController2D`. The `BasicInputProvider2D` complements the setup for input handling. Scene configuration of these components finalizes the integration process.

## Contact and Support
If you have any questions, feedback, or need assistance with the Flexible Grid Controller, feel free to reach out. You can contact me directly via email at crookedhead@outlook.com for specific queries or suggestions. Additionally, for broader community support and discussions, join the [Flexible Grid Controller Discord server](https://discord.gg/h5x8MAFrCF).
