Testing from home
=================

Setting up networks on a single computer
----------------------------------------

The UDP-broadcast-only network modules (Go, D, Rust) are designed to be able to be used on a single machine, with multiple programs listening to the same port. Erlang and Elixir users using nodes should have no problems either.

You are allowed to manually input whatever you need for network configuration, such as manually naming the peers' names, ports, or whatever else in order to avoid needing to write general peer discovery.

This is not a good time to be having network problems, so if you have any problems ask for help as soon as possible.

Testing network problems
------------------------

### Disconnects

Testing resilience to disconnects is hard when there is no network cable to unplug. Instead, we have to use whatever operating system features we have to emulate network problems. DIfferent systems have different limitations: 

 - Linux lets you block outgoing communication for programs run as part of a group, but not incoming - which is like disconnecting one half of the network cable.
 - Windows let's you block all network communication (in and out) for a single program (effectively disconnecting that program), but only for network activity that isn't on localhost or loopback - which is what all our network communication will be.
 
This means that creating test scenarios for disconnecting one of three elevators is not easily possible using just the system features. However, we can block *all* communication that isn't to and from the simulators, which lets us test what happens when all the elevators are (simultaneously) disconnected. Because of this, we recommend **only using two elevators**, as testing with three won't test anything different than testing with just two.

 - Linux: use iptables as you would for packet loss, but just drop everything:  
    ```
    sudo iptables -A INPUT -p tcp --dport 15657 -j ACCEPT
    sudo iptables -A INPUT -p tcp --sport 15657 -j ACCEPT
    # +more for other simulator ports you use
    sudo iptables -A INPUT -j DROP
    ```  
   Then run `sudo iptables -F` to flush the filter chain when you are done.
 - Windows: use a program called [clumsy](http://jagt.github.io/clumsy/), and set the rule to  
    ```
    udp or (tcp.DstPort != sim1 and tcp.SrcPort != sim1 and tcp.DstPort != sim2 and tcp.SrcPort != sim2)
    ```  
    where `sim1` and `sim2` are the ports used by the simulators, then enable the `Drop` function for both inbound and outbound with a chance of 100%.

### Packet loss

For testing packet loss, do the same thing, but change the probability to 20%:

 - Linux: Change the last line to `sudo iptables -A INPUT -m statistic --mode random --probability 0.2 -j DROP`.
 - Windows: Literally just change the drop chance to 20% in the thing with the number.
 
### If you insist on testing three

While it is both not required and not recommended to test network problems with three elevators, here are two possible ways of doing it - if you insist:

 - Add disconnect (and packet loss, if you want) to your own networking code. Since the stop button and obstruction switch are not in use, you could use those to trigger the network-problem effects.
 - Run one of the elevators in a virtual machine (like VirtualBox), then disconnect all networking to the virtual machine. Chances are that your virtual machine will be some other operating system than the host machine, so this is not always possible. The most common combination of a Windows host with some Linux VM running on VirtualBox seems to work nicely, with no extra configuration needed.
 
### Local communication with sockets

If your program needs to perform some internal communication using sockets, then you will have to add exceptions to the rules for packet loss and disconnect testing. Ask for help if you need it.
 

 
Elevator hardware
-----------------

One of the tests involves disconnecting the power to the elevator motor. You can simulate this by pressing the `[key_moveStop]` key in the simulator window, which by default is bound to `8` (numpad or number row, both work). If your software is constantly, consistently, or confusingly sending commands to move the elevator, you can still force the elevator to get stuck by continuously spamming the moveStop key.

Specification clarifications
============================

As is tradition, there are some inevitable problems with the specification...

 - Cab order redundancy:
   - In order to open up more valid solutions than saving cab orders to disk, it should be possible to store the backup of cab requests on the other nodes, by using the assumption that *at least one other elevator is working normally*. However, the current specification requires that cab requests be served when the elevator is disconnected (in order to let people off), which means that no service guarantee can be delivered if the backup copy of that cab order must be stored on an unreachable node, meaning that no light should be turned on.  
     In this case, the correct solution would be to service the request without turning on the light.  
   
     Double-however, another part of the specification says that you can assume that there are no two simultaneous errors, which means that once a node has lost connection to the network (making it unable to backup the cab request), it cannot also crash (guaranteeing that it will serve the un-backed-up cab request).  
     Because of this, servicing the request *and also turning on the light* is also a valid solution.
