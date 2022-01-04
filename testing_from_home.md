Testing from home
=================

Setting up networks on a single computer
----------------------------------------

The UDP-broadcast-only network modules (Go, D, Rust) are designed to be able to be used on a single machine, with multiple programs listening to the same port. Erlang and Elixir users using nodes should have no problems either.

You are allowed to manually input whatever you need for network configuration, such as manually naming the peers' names, ports, or whatever else in order to avoid needing to write general peer discovery.

Testing network problems
------------------------

### Disconnects

Testing resilience to disconnects is hard when there is no network cable to unplug. Instead, we have to use whatever operating system features we have to emulate network problems. Different systems have different limitations: 

 - Linux lets you block outgoing communication for programs run as part of a group, but not incoming - which is like disconnecting one half of the network cable.
 - Windows let's you block all network communication (in and out) for a single program (effectively disconnecting that program), but only for network activity that isn't on localhost or loopback - which is what all our network communication will be.
 
This means that creating test scenarios for disconnecting one of three elevators is not easily possible using just the system features. However, we can block *all* communication that isn't to and from the simulators, which lets us test what happens when all the elevators are (simultaneously) disconnected. Because of this, we recommend **only using two elevators** when testing on one machine, as testing with three won't test anything different than testing with just two. But if you have access to multiple machines (real or virtual), this becomes a lot easier to test.

Blocking everything is probably a little aggressive, as you might want to use a web browser, screen share, ssh, or anything else that uses the internet while testing for disconnects (or packet loss). Instead, you might want to block just the communication between *your* programs. To do this, you need to know what ports and protocols your program is using for communication. For those of you who have programmed your own ports manually this shouldn't be a problem to find (typically: the ports you pass in to the network module you are handed, or all the ports passed to any network call that looks something like "bind"), but some of you use other infrastructure where this is more... hidden (Elixir/Erlang, or anything else that "just works").

 - Linux: use a program called `iptables`
    ``` sh
    sudo iptables -A INPUT -p PROTOCOL --dport PORT1 -m statistic --mode random --probability 0.2 -j DROP
    sudo iptables -A INPUT -p PROTOCOL --dport PORT2 -m statistic --mode random --probability 0.2 -j DROP
    # +more for other ports you use
    ```
    where PROTOCOL is either `tcp` or `udp`, and PORTn is whatever ports you bind to. You will need one line for each port you use, if you use multiple.
    To test disconnects, just up the `--probability` to `1`.
    Use `sudo iptables -F` to flush the filter chain when you are done.

 - Windows: use a program called [clumsy](http://jagt.github.io/clumsy/), and set the rule to 
    ```
    udp and (udp.DstPort == PORT1 or udp.DstPort == PORT2 [or ...])
    ```
    where the PORTn's are whatever ports you bind to, then enable the `Drop` function for both inbound and outbound with a chance of 20% to test packet loss, or 100% to test disconnects.
 - Mac (*Temporary solution*): Use [Network Link Conditioner](https://developer.apple.com/download/more/?q=Additional%20Tools). (Select your version of Xcode, then select the `.dmg` download, and install. Network Link Conditioner is found in the Hardware folder. See [this blog post for more detailed install/usage instructions](https://nshipster.com/network-link-conditioner/).)  
    Make your own profile from the "Manage Profiles..." menu, then set the *Downlink Packets Dropped* and *Uplink Packets Dropped* to 20%.  
    *If you cannot find Network Link Conditioner in Settings the next time you want to use it, search for it in spotlight (`cmd + space`).* 
 - Mac: (*The real solution - but incomplete*): Network Link Conditioner affects *all* traffic, including traffic to the simulator. This means testing disconnects (100% loss) would also disconnect the simulator. The real solution is to use `dummynet` and `packetfilter`, something like this..?  
    ``` sh
    sudo pfctl -E  
    sudo dnctl pipe 1 config plr 0.2
    echo "dummynet out proto PROTOCOL from PORTn to any pipe 1" | sudo pfctl -f -
    echo "dummynet in proto PROTOCOL from any to PORTn pipe 1" | sudo pfctl -f -
    ```
    where PROTOCOL is either `tcp` or `udp`, and PORTn is whatever ports you bind to. You will need one line for each port you use, if you use multiple.  
    Use `sudo pfctl -d` to disable the packet filtering.  
    *Questions: Do we have to configure the pipe (`dnctl`) each time? Do we have to configure the rules (`dummynet`) each time? If not, how do we clear the rules?*
   

### Blocking everything

In case you want to block *everything*, change the rule to let the simulator(s) through, then drop everything else. 

 - Linux / `iptables`
    ```
    sudo iptables -A INPUT -p tcp --dport 15657 -j ACCEPT
    sudo iptables -A INPUT -p tcp --sport 15657 -j ACCEPT
    # +more for other simulator ports you use
    sudo iptables -A INPUT -j DROP
    ```  
   (And `sudo iptables -F` to flush - as before)
 - Windows: Use a program called [clumsy](http://jagt.github.io/clumsy/), and set the rule to  
    ```
    udp or (tcp.DstPort != sim1 and tcp.SrcPort != sim1 and tcp.DstPort != sim2 and tcp.SrcPort != sim2)
    ```  
    where `sim1` and `sim2` are the ports used by the simulators, then enable the `Drop` function with 20% or 100% probability.

If your program needs to perform some internal communication using sockets, then you will have to add exceptions to the rules for packet loss and disconnect testing. Ask for help if you need it.
 
### If you insist on testing three elevators on one machine

Here are two possible ways of doing it:

 - Add disconnect (and packet loss, if you want) to your own networking code. Since the stop button is not in use, you could use that to trigger the network-problem effects.
 - Run one of the elevators in a virtual machine (like VirtualBox), then disconnect all networking to the virtual machine. Chances are that your virtual machine will be some other operating system than the host machine, so this is not always possible. The most common combination of a Windows host with some Linux VM running on VirtualBox seems to work nicely, with no extra configuration needed.
 

 
Elevator hardware
-----------------

One of the tests involves disconnecting the power to the elevator motor. You can simulate this by pressing the `[key_moveStop]` key in the simulator window, which by default is bound to `8` (numpad or number row, both work). If your software is constantly, consistently, or confusingly sending commands to move the elevator, you can still force the elevator to get stuck by continuously spamming the moveStop key.


