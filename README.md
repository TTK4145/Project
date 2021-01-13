Elevator Project
================


Summary
-------
Create software for controlling `n` elevators working in parallel across `m` floors.


Main requirements
-----------------
Be reasonable: There may be semantic hoops that you can jump through to create something that is "technically correct". Do not hesitate to contact us if you feel that something is ambiguous or missing from these requirements.

### No orders are lost
 - Once the light on a hall call button (buttons for calling an elevator to that floor; top 6 buttons on the control panel) is turned on, an elevator should arrive at that floor
 - Similarly for a cab call (for telling the elevator what floor you want to exit at; front 4 buttons on the control panel), but only the elevator at that specific workspace should take the order
 - This means handling network packet loss, losing network connection entirely, software that crashes, and losing power - both to the elevator motor and the machine that controls the elevator
   - For cab orders, handling loss of power/software crash implies that the orders are executed once service is restored
   - The time used to detect these failures should be reasonable, ie. on the order of magnitude of seconds (not minutes)
   - Network packet loss is not an error, and can occur at any time
 - If the elevator is disconnected from the network, it should still serve all the currently active orders (ie. whatever lights are showing)
   - It should also keep taking new cab calls, so that people can exit the elevator even if it is disconnected from the network
   - The elevator software should not require reinitialization (manual restart) after intermittent network or motor power loss

### Multiple elevators should be more efficient than one
 - The orders should be distributed across the elevators in a reasonable way
   - Ex: If all three elevators are idle and two of them are at the bottom floor, then a new order at the top floor should be handled by the closest elevator (ie. neither of the two at the bottom).
 - You are free to choose and design your own "cost function" of some sort: Minimal movement, minimal waiting time, etc.
 - The project is not about creating the "best" or "optimal" distribution of orders. It only has to be clear that the elevators are cooperating and communicating.
 
### An individual elevator should behave sensibly and efficiently
 - No stopping at every floor "just to be safe"
 - The hall "call upward" and "call downward" buttons should behave differently
   - Ex: If the elevator is moving from floor 1 up to floor 4 and there is a downward order at floor 3, then the elevator should not stop on its way upward, but should return back to floor 3 on its way down
 
### The lights and buttons should function as expected
 - The hall call buttons on all workspaces should let you summon an elevator
 - Under normal circumstances, the lights on the hall buttons should show the same thing on all workspaces 
   - Under circumstances with high packet loss, at least one light must work as expected
 - The cab button lights should not be shared between elevators
 - The cab and hall button lights should turn on as soon as is reasonable after the button has been pressed
   - Not ever turning on the button lights because "no guarantee is offered" is not a valid solution
   - You are allowed to expect the user to press the button again if it does not light up
 - The cab and hall button lights should turn off when the corresponding order has been serviced
 - The "door open" lamp should be used as a substitute for an actual door, and as such should not be switched on while the elevator is moving
   - The duration for keeping the door open should be in the 1-5 second range
 - The obstruction switch should substitute the door obstruction sensor inside the elevator
   - The door should not close while it is obstructed

 
Start with `1 <= n <= 3` elevators, and `m == 4` floors. Try to avoid hard-coding these values: You should be able to add a fourth elevator with no extra configuration, or change the number of floors with minimal configuration. You do, however, not need to test for `n > 3` and `m != 4`.


Unspecified behaviour
---------------------
Some things are left intentionally unspecified. Their implementation will not be tested, and are therefore up to you.

Which orders are cleared when stopping at a floor
 - You can clear only the orders in the direction of travel, or assume that everyone enters/exits the elevator when the door opens
 
How the elevator behaves when it cannot connect to the network (router) during initialization
 - You can either enter a "single-elevator" mode, or refuse to start
 
How the hall (call up, call down) buttons work when the elevator is disconnected from the network
 - You can optionally refuse to take these new orders
 
What the stop button does
   - The stop button functionality (if/when implemented) is up to you

   
Permitted assumptions
---------------------

The following assumptions will always be true during testing:
 1. At least one elevator is always working normally
 2. No multiple simultaneous errors: Only one error happens at a time, but the system must still return to a fully operational state after this error
    - Recall that network packet loss is *not* an error in this context, and must be considered regardless of any other (single) error that can occur
 3. No network partitioning: There will never be a situation where there are multiple sets of two or more elevators with no connection between them
 4. Cab call redundancy with a single elevator is not required
    - Given assumptions **1** and **2**, a system containing only one elevator is assumed to be unable to fail
   
Additional resources
--------------------

Go to [the project resources repository](https://github.com/TTK4145/Project-resources) to find more resources for doing the project. This information is not required for the project, and is therefore maintained separately.

See [Testing from home](/testing_from_home.md) document on how to test with unreliable networking on a single computer.