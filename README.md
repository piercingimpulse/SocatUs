# SocatUs

How to connect two mobile device (iOS and/or Android) via P2P in Among Us legacy.

# Brief(?) intro
[From reddit: 03/06/2021]: I've been trying to fix Among Us for legacy device.
For my test, I've used an iPad3,4 (Retina) on iOS8.4.1 and iOS10.3.3 (Coolbooter).I've tested from 2018.11.14 up to 2019.10.10. (32bit)
What I've discovered so far:
- It seems that  from 2019.11.5 it just crash if trying to play online and from 2020.2.7 it just crash at opening, at least on my iPad.
- Changing info.plist file CFBundleShortVersion does not have any effect on tricking the server; (due change in how the connection is handled)
- Changing amongus.app/Data/globalgamemanagers with Filza HexEditor at 011E0 to 2021.5.25 does not work either;
- Upgrading amongus.app/Data/Resources/unity_built_extra gives of course the pink screen of death since change the version of Unity to read the assets;
- Upgrading the assets just simply does not work;
- In every version, playing via LAN works if on the same version of the device and of course on the same network works (tested with iPad 3,4 and Android phone);
- A private server (Impostor) can't be used due support starting from version 2020.09.27.
- ImpostorConfig tweak (available on Cydia) seems kinda of working or anyway affecting AmongUs in the way it communicates to the server, therefore I feel that can be a really good starting point in developing a possible tweak that could interfere and communicate between devices rather than broadcast to LAN [ADD 16/07/2021]
It seems there is no real way to spoof the version of the AmongUs in order to use it online, but to be honest I think it's a fair point: not having the right assets and binary, how can you even think to join a server that has Polus or Mira while running 2018 version? This does not mean that I give up lol. What I've realized I need it to think out of the box.
So the idea is simple really, and take inspiration from Among Us tunnel project and info about ZeroTier, instead of join the main server, what about creating a virtual LAN to connect whoever want to play with someone using the same version? It should allow people to play using the "Local" gameplay option if using ZeroTier and the same version of Among Us across the legacy device.


# Disclaimer
First, a huge thanks to InvoxiPlayGames and his AmongUSP2P! That's what really inspired me to find the right solution!

At this stage is a very raw interpretation of the same process, but using the mobile Terminal and socat.
It comes with its limitations, the biggest of it all it's can only handle a connection between server and a single client, and seems to be a bit unstable...
but I believe this then can be largely be adapt for any type of LAN game in the future, especially the one that rely on UDP and don't have option to connect to a remote IP as default option - this is doable as long you know which port the game use to communicate and how to forward port on your router.
I feel that sharing those infos can be useful to find a more stable solution (: .
# Tool necessary
- Among Us v2018.12.24.1 (available for iOS 8.x forward) (it it's possible that works with earlier version too - I've tested a newer 2019 build and it failed)
- Your favorite Terminal (NewTerm, mTerminal, etc.)
- Socat Net (from Cydia)
- Forward port TCP/UDP 22023 and 47777
- iOS 8x or higher
# Preparation for both client and server:
1. Go to your router setting page (usually 192.168.1.0 or similar)
2. Following your router manual, forward port 47777 and 22023 where you are going to play or host Among Us. Remember: bind the correct IP, if you unsure, on your iDevice tap on Settings -> Wifi and tap on your network name and take note of the IP address (eg:192.168.1.200)
3. Open https://www.yougetsignal.com/tools/open-ports/ on your laptop for testing and taking note of your remote IP. Share your address between each others
# Server (game host)
1. Make sure you are not in the LAN room on Among Us;
2. Open the mobile terminal and create a shortcut to this commands:

> socat tcp-listen:47777,reuseaddr,keepalive,broadcast,fork udp:localhost:47777 & socat tcp-listen:22023,reuseaddr,keepalive,broadcast,fork udp:localhost:22023 & socat udp-recvfrom:47777,reuseaddr,keepalive,broadcast,fork tcp:CLIENT:REMOTE.IP:47777

# Explication
Among Us uses UDP communication to create host and connect between players while using LAN. UDP packets can be send/receive only if the devices are on the same network*. We need to use socat to listen port TCP 47777 and 22023, since TCP can be used over internet. The parameter reuseaddr allows socat to keep communicating and don't close the socket after the first response (same for keepalive and broadcast? I use them as reinforcement lol); fork tunnel the packets and the communications to a different format. This command starts three socat processes: the first two to listen TCP port and convert the packets in UDP, while the last one convert the broadcast message send by the Among Us server (the name that appear in the list of games available) over TCP.

# Client (player)
1. Make sure you are not in the LAN room on Among Us;
2. Open the mobile terminal and create a shortcut to this commands:

* socat tcp-listen:47777,reuseaddr,keepalive,broadcast,fork udp:localhost:47777 & socat udp-l:22023,reuseaddr,broadcast,fork tcp:SERVER.REMOTE.IP:22023

# Explication
For the client we need to open a TCP port 47777 and tunnel the broadcast message as UDP,; since we are tunneling packets, this it does not preserve the IP of the sender, but instead it reads the packets as being send from the same remote IP of the client; to prevent that, the second socat use udp-l to "catch" the packets and tunnel them back to the server remote ip, where they will be then converted by the socat server in UDP packets!

# Final Note
I've tested the whole procedure using two device on the same network with some tweaks to ensure I was tunneling every packets and it works, but as I don't have access two different network and forwarding port it's not possible on mobile network I can't say it works 100%; on the other hand, I had positive result in broadcasting at least the game host on the client mobile network.If anywant it's available to test this with me, just let me know (:

# *If you have iOS 9 or higher
You can use ZeroTier as more "elegant" solution that does not required port forwarding. Still need the use of Terminal and socat.
1. From a computer, go on the ZeroTier website and create a free account
2. Follow the manual on how to create a network (I suggest you stick to private server) and keep the website open for now.
3. Download ZeroTier One on both client and server devices
4. Connect the server device to the server with ZeroTier One app
5. Go back to the ZeroTier setting website and authorized the new connection and take note of each address in the Managed IP column (Note: You can give a name to the device to recognize to how belong the IP)
6. Now connect the client and repeat step 5
7. Once you are sure both devices are connect to the same VPN server, open the server (host) mobile terminal and type this command

* socat udp-recvfrom:47777,reuseaddr,keep alive,broadcast,fork udp:client.ip.from.zerotier:47777
