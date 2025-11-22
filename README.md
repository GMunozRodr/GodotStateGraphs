# GodotStateGraphs
 A state machine plugin for Godot.  
 All relevant classes that should be accessed by the user are documented inside the Godot Editor itself.

 Last tested version: Godot 4.5
 
# How to use

The plugin organizes itself with a clear hierarchy of components and objects that will help define the system

## StateMachineNode

The StateMachineNode is the actual node that will attach the behavior into the Scene. StateMachineNodes request a StateMachine resource that contains all the data of the State Machine. It additionally contains several flags and fields to customize the behavior of the State Machine at runtime

### Runtime Actions

The StateMachineNode is the one in charge of executing the logic of the State Machine at runtime and, as such, it contains the logic related to the different runtime actions it can perform.
- **Start**: This action will be called automatically on load if the `auto_start` flag is enabled, or on reset if the `auto_restart` flag is enabled. It can also be manually called with the `start_or_resume()` function if the State Machine is resetted. This action will start execution of the State Machine and, if appropriate, call the `_on_enter()` function of the starting node.
- **Reset**: This action will be called automatically on halt if the `auto_restart` flag is enabled. It can also be manually called with the `reset()` function. This action will stop execution of the State Machine and set the starting node as active.
- **Halt**: This action will be called automatically when the active node requests a transition through an output that does not have a valid connection to another node. It can also be manually triggered by calling `halt()`. This action will stop execution of the State Machine and prevent it from executing until `reset()` is called.
- **Pause**: This action can be triggered by calling `pause()`. It will stop execution of the State Machine. Execution can be resumed by calling `start_or_resume()`.
- **Evaluate**: This action represents the "main loop" of the state machine. It can be manually triggered by calling `evaluate()`. It will also be called at different rates depending on the mode of the statemachine:
  - Process: The evaluation action will be triggered on every frame.
  - Physics Process: The evaluation action will be triggered on every physics frame.
  - Custom: The evaluation action will not be automatically triggered.

#### Evaluation

On every evaluation the State Machine will perform the following steps:
- Call `_on_check()` and retrieve the return value.
- If the return value is a valid output value and the connected node is valid, execute `_on_exit()`, set the new node to the returned node and call `_on_enter()`.
- If the return value is a valid output value but no valid node is connected to that output, the State Machine halts.
- If the return value is not a valid value or `frame_on_transition` is true, call `_on_eval()`.

## StateMachine

The StateMachine resource contains all the data related to a specific State Machine. Whenever a StateMachineNode containing a valid StateMachine resource is selected in the scene, the Graph UI will appear in the editor to allow the user to edit the State Machines. The only exposed variable of the StateMachine is the array of StateResources. Every StateResource defined in this array will (as long as it is a valid StateResource) create a new State option when generating nodes in the State Machine

## StateResource

State resources are simple resource objects that contain the data of a State in the State Machine. They have three main exposed variables: The name of the State, the color of the connections, and a reference to the Script representing the State.

## State

States define the different types of nodes the State Machine can have. Every state is defined by a script of the following structure:
![image](https://github.com/AsperTheDog/GodotStateGraphs/assets/45227294/8544aa4b-2a0c-468b-867b-8126a6d96007)

The ExitEvents enum **must** be defined, it **must not** be empty and it **must not** define custom values for the defined entries.

Functions `_on_enter()`, `_on_exit()` and `_on_eval()` can be optionally overridden, but `_on_check()` **must** be overridden for the State Script to be considered valid.

When the `_on_check()` function returns, the value will trigger one of three behaviors:
 - When the value returned is valid (it is a value of the ExitEvents enum) and the associated node output is connected to a valid node, the State Machine will transition to said node
 - When the value returned is valid but there is no defined connection from the associated node output, the State Machine will halt
 - When the value returned is not valid, the State Machine will call `_on_eval()` and wait for the next evaluation

## Node

Nodes are the instances of a specific State type. These are created and organized through the Graph UI and internally saved and handled by the StateMachine resource.
The structure and components of a node are determined by the State that defines it. Following the example provided in the State section:
![image](https://github.com/AsperTheDog/GodotStateGraphs/assets/45227294/d53e7d3a-c5d9-493e-ac23-3a762416477e)

## START node

This is a special node that will be always present automatically in every State Machine. It cannot be deleted or duplicated. It only contains one single output that can determine where the State Machine starts by connecting it to a node. If the START node is not connected to any other node, the State Machine will immediately halt on Start.  
![image](https://github.com/AsperTheDog/GodotStateGraphs/assets/45227294/d3495d6f-e066-40a3-a514-0d2e82b25a08)

## JUMP node

This special node lets contains two elements: An output and a String field called "keyword". When a JUMP node is created, it can be connected to any other node to define JUMP points. These points can be used to force the state machine to transition to the target node regardless of the current active node. To trigger a Jump, the function `jump_to()` must be called. This function will receive a keyword String and, optionally, two booleans to determine if the functions `_on_enter()` and `_on_exit()` should be called in the transition process.  
![image](https://github.com/AsperTheDog/GodotStateGraphs/assets/45227294/475d58ea-a02e-4313-8106-06e6b4a9ee40)

# Screenshots

<img width="1424" height="807" alt="screenshot1" src="https://github.com/user-attachments/assets/712fb54e-0a80-442e-939f-1b286fb36e5b" />
<img width="2560" height="1344" alt="screenshot2" src="https://github.com/user-attachments/assets/f8c4ed86-1f7b-4ecf-a156-429eb5f7755c" />
<img width="2553" height="1345" alt="screenshot3" src="https://github.com/user-attachments/assets/cf12b55a-bb47-46fc-b169-d43df06606de" />
