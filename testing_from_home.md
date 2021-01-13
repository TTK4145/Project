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

Testing resilience to disconnects is hard when there is no network cable to unplug. Instead, we have to use whatever operating system features we have to emulate network problems. Different systems have different limitations: 

 - Linux lets you block outgoing communication for programs run as part of a group, but not incoming - which is like disconnecting one half of the network cable.
 - Windows let's you block all network communication (in and out) for a single program (effectively disconnecting that program), but only for network activity that isn't on localhost or loopback - which is what all our network communication will be.
 
This means that creating test scenarios for disconnecting one of three elevators is not easily possible using just the system features. However, we can block *all* communication that isn't to and from the simulators, which lets us test what happens when all the elevators are (simultaneously) disconnected. Because of this, we recommend **only using two elevators**, as testing with three won't test anything different than testing with just two.

Blocking everything is probably a little aggressive, as you might want to use a web browser, screen share, ssh, or anything else that uses the internet while testing for disconnects (or packet loss). Instead, you might want to block just the communication between *your* programs. To do this, you need to know what ports and protocols your program is using for communication. For those of you who have programmed your own ports manually this shouldn't be a problem to find (typically: the ports you pass in to the network module you are handed, or all the ports passed to any network call that looks something like "bind"), but some of you use other infrastructure where this is more... hidden (Elixir/Erlang, or anything else that "just works").

 - Linux: use a program called `iptables`
    ``` sh
    sudo iptables -A INPUT -p PROTOCOL --dport PORT1 -m statistic --mode random --probability 0.2 -j DROP
    sudo iptables -A INPUT -p PROTOCOL --dport PORT2 -m statistic --mode random --probability 0.2 -j DROP
    # +more for other ports you use
    ```
    where PROTOCOL is either `tcp` or `udp`, and PORTn is whatever ports you bind to. You will need one line for each port you use, if you use multiple.
    To test disconnects, just up the `--probability` to `1`.
    Use `sudo iptables -F` to flush the filter chain when you are done

 - Windows: use a program called [clumsy](http://jagt.github.io/clumsy/), and set the rule to 
    ```
    udp and (udp.DstPort == PORT1 or udp.DstPort == PORT2 [or ...])
    ```
    where the PORTn's are whatever ports you bind to, then enable the `Drop` function for both inbound and outbound with a chance of 20% to test packet loss, or 100% to test disconnects.

### Blocking everything

In case you want to block *everything*, change the rule to let the simulator(s) through, then drop everything else. This is what the student assistants do for live testing on the lab, as it lets them avoid having to look up what ports you program uses.

 - Linux / `iptables`
    ```
    sudo iptables -A INPUT -p tcp --dport 15657 -j ACCEPT
    sudo iptables -A INPUT -p tcp --sport 15657 -j ACCEPT
    # +more for other simulator ports you use
    sudo iptables -A INPUT -j DROP
    ```  
   (And `sudo iptables -F` to flush - as before)
 - Windows: use a program called [clumsy](http://jagt.github.io/clumsy/), and set the rule to  
    ```
    udp or (tcp.DstPort != sim1 and tcp.SrcPort != sim1 and tcp.DstPort != sim2 and tcp.SrcPort != sim2)
    ```  
    where `sim1` and `sim2` are the ports used by the simulators, then enable the `Drop` function with 20% or 100% probability.

If your program needs to perform some internal communication using sockets, then you will have to add exceptions to the rules for packet loss and disconnect testing. Ask for help if you need it.
 
### If you insist on testing three

While it is both not required and not recommended to test network problems with three elevators, here are two possible ways of doing it - if you insist:

 - Add disconnect (and packet loss, if you want) to your own networking code. Since the stop button and obstruction switch are not in use, you could use those to trigger the network-problem effects.
 - Run one of the elevators in a virtual machine (like VirtualBox), then disconnect all networking to the virtual machine. Chances are that your virtual machine will be some other operating system than the host machine, so this is not always possible. The most common combination of a Windows host with some Linux VM running on VirtualBox seems to work nicely, with no extra configuration needed.
 

 
Elevator hardware
-----------------

One of the tests involves disconnecting the power to the elevator motor. You can simulate this by pressing the `[key_moveStop]` key in the simulator window, which by default is bound to `8` (numpad or number row, both work). If your software is constantly, consistently, or confusingly sending commands to move the elevator, you can still force the elevator to get stuck by continuously spamming the moveStop key.


