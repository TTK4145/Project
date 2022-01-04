Elevator Project
================


Summary
-------
Create software for controlling `n` elevators working in parallel across `m` floors.


Main requirements
-----------------
Be reasonable: There may be semantic hoops that you can jump through to create something that is "technically correct". Do not hesitate to contact us if you feel that something is ambiguous or missing from these requirements.

### The button lights are a service guarantee
 - Once the light on a hall call button (buttons for calling an elevator to that floor; top 6 buttons on the control panel) is turned on, an elevator should arrive at that floor
 - Similarly for a cab call (for telling the elevator what floor you want to exit at; front 4 buttons on the control panel), but only the elevator at that specific workspace should take the order

### No calls are lost
 - Failure states are anything that prevents the elevator from communicating with other elevators or servicing calls
   - This includes losing network connection entirely, software that crashes, doors that won't close, and losing power - both to the elevator motor and the machine that controls the elevator
   - Network packet loss is not a failure, and can occur at any time
   - An elevator entering the network is not a failure
 - No calls should be lost in the presence of failures
   - For cab calls, handling loss of power or software crash implies that the calls are executed once service to that elevator is restored
   - The time used to handle (compensate for) these failures should be reasonable, i.e. on the order of magnitude of seconds (not minutes)
 - If the elevator is disconnected from the network, it should still serve all the currently active calls (i.e. whatever lights are showing)
   - It should also keep taking new cab calls, so that people can exit the elevator even if it is disconnected from the network
   - The elevator software should not require reinitialization (manual restart) after intermittent network or motor power loss

### The lights and buttons should function as expected
 - The hall call buttons on all workspaces should let you summon an elevator
 - Under normal circumstances, the lights on the hall buttons should show the same thing on all workspaces
   - Normal circumstances mean when there are no active failures and no packet loss
   - Under circumstances with packet loss, at least one light must work as expected
 - The cab button lights should not be shared between workspaces
 - The cab and hall button lights should turn on as soon as is reasonable after the button has been pressed
   - Not ever turning on the button lights because "no guarantee is offered" is not a valid solution
   - You are allowed to expect the user to press the button again if it does not light up
 - The cab and hall button lights should turn off when the corresponding call has been serviced

### The door should function as expected
 - The "door open" lamp should be used as a substitute for an actual door
   - The door should not be open (light switched on) while the elevator is moving
   - The duration for keeping the door open when stopping at a floor should be 3 (three) seconds
 - The obstruction switch should substitute the door obstruction sensor inside the elevator
   - The door should not close while it is obstructed
   - The obstruction can trigger (and un-trigger) at any time

### An individual elevator should behave sensibly and efficiently
 - No stopping at every floor "just to be safe"
 - The hall "call upward" and "call downward" buttons should behave differently
   - Ex: If the elevator is moving from floor 1 up to floor 4 and there is a downward call at floor 3, then the elevator should not stop on its way upward, but should return back to floor 3 on its way down

### Multiple elevators should be more efficient than one
 - The calls should be distributed across the elevators in a reasonable way
   - Ex: If all three elevators are idle and two of them are at the bottom floor, then a new call at the top floor should be handled by the closest elevator (i.e. neither of the two at the bottom).
 - You are free to choose and design your own "cost function" of some sort: Minimal movement, minimal waiting time, etc.
 - The project is not about creating the "best" or "optimal" distribution of calls. It only has to be clear that the elevators are cooperating and communicating.


Start with `1 <= n <= 3` elevators, and `m == 4` floors. Try to avoid hard-coding these values: You should be able to add a fourth elevator with no extra configuration, or change the number of floors with minimal configuration. You do, however, not need to test for `n > 3` and `m != 4`.


Permitted assumptions
---------------------

The following assumptions will always be true during testing:
 1. There is always at least one elevator that is not in a failure state
    - I.e. there is always at least one elevator that can serve calls
    - "No failure" includes the door obstruction: At least one elevator will be able to close its doors
 2. Cab call redundancy with a single elevator is not required
    - Given assumption **1**, a system containing only one elevator is assumed to be unable to fail
 3. No network partitioning: There will never be a situation where there are multiple sets of two or more elevators with no connection between them
    - Note that this needs 4 or more elevators to become applicable, which we will not test anyway


Unspecified behavior
--------------------
Some things are left intentionally unspecified. Their implementation will not be tested, and are therefore up to you.

Which calls are cleared when stopping at a floor
 - You can clear only the calls in the direction of travel, or assume that everyone enters/exits the elevator when the door opens

How the elevator behaves when it cannot connect to the network (router) during initialization
 - You can either enter a "single-elevator" mode, or refuse to start

How the hall (call up, call down) buttons work when the elevator is disconnected from the network
 - You can optionally refuse to take these new calls

What the stop button does
 - The stop button functionality (if/when implemented) is up to you

Additional resources
--------------------

Go to [the project resources repository](https://github.com/TTK4145/Project-resources) to find more resources for doing the project. This information is not required for the project, and is therefore maintained separately.

See [Testing from home](/testing_from_home.md) document on how to test with unreliable networking on a single computer.