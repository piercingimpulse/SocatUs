# SocatUs

How to connect two mobile device (iOS and/or Android) via P2P in Among Us legacy.

## Brief intro (?)
I've been trying to fix Among Us for legacy device.
For my test I've used:
* iPad3,4 (Retina) - iOS8.4.1 and iOS10.3.3 Coolbooter
* Among Us 2018.11.14 up to 2019.10.10. (32bit)
##### What I've discovered so far?
* It seems that  from 2019.11.5 it just crash if trying to play online and from 2020.2.7 it just crash at opening, at least on  iPad, but I feel it is a common issue on almost all 32 bit devices.
* Changing info.plist file CFBundleShortVersion does not have any effect on tricking the server; this probably due to the change in how the connection is handled in the newver version.
* Changing amongus.app/Data/globalgamemanagers with Filza HexEditor at 011E0 to 2021.5.25 does not work either as still the issue lies in the way the connection protocol is written and how the server read those connection;
* Upgrading amongus.app/Data/Resources/unity_built_extra gives of course the pink screen of death since change the version of Unity to read the assets, so therefore does not help with the connection protocol;
* Upgrading the assets from a newver version into an older just simply does not work (*duh!*);
* In every version, playing via LAN works if on the same version of the device and of course on the same network works (tested with iPad 3,4 and Android phone);
* A private server (Impostor) can't be used due support starting from version 2020.09.27.
* __*ImpostorConfig tweak (available on Cydia) seems kinda of working or anyway affecting Among Us in the way it communicates to the server, therefore I feel that can be a really good starting point in developing a possible tweak that could interfere and communicate between devices rather than broadcast to LAN*__
It seems there is no real way to spoof the version of the Among Us in order to use it online, but to be honest I think it's a fair point: how can you even think to join a match in Polus or Mira while running 2018 version? It can't be archieve when the software simply does not have the right assets and binary to play the match.
##### The solution
What I've realized I need it to think out of the box.
So the idea is simple really, and take inspiration from Among Us tunnel project and info about ZeroTier, instead of join the main server, what about creating a virtual LAN to connect whoever want to play with someone using the same version? It should allow people to play using the "Local" gameplay option if using ZeroTier and the same version of Among Us across the legacy device.
##### Disclaimer
I just want to give a huge thanks to @InvoxiPlayGames and @Randy-420 for inspiring me in pursuing the dev road (: ! 
Please have a look especially at InvoxiPlayGames [AmongUsP2P](https://github.com/InvoxiPlayGames/AmongUsP2P) - you could consider SocketUs as a raw prototype/fork of AmongUsP2P really.
_**It comes with its limitations**_ ~~and the biggest of it all it's can only handle a connection between server and a single client, and seems to be a bit unstable...~~ [17/07/2021] **I've actually managed to connect more clients! Please read the update code!**
That said, I believe this, once refined, can be largely be adapt for any type of LAN game in the future, especially the one that rely on UDP and don't have option to connect to a remote IP as default option - this is doable as long you know which port the game use to communicate and how to forward port on your router.

## How to use SocatUs

#### Tool necessary
- Among Us v2018.12.24.1 (available for iOS 8.x forward) (it it's possible that works with earlier version too - I've tested a newer 2019 build and it failed)
- Your favorite Terminal (NewTerm, mTerminal, etc.)
- Socat Net (from Cydia)
- Forward port TCP/UDP 22023 and 47777
- iOS 8x or higher
#### Preparation for both clients and server:
1. Go to your router setting page (usually 192.168.1.0 or similar)
2. Following your router manual, forward port 47777 and 22023 where you are going to play or host Among Us. Remember: bind the correct IP, if you unsure, on your iDevice tap on Settings -> Wifi and tap on your network name and take note of the IP address (eg:192.168.1.200)
3. Open https://www.yougetsignal.com/tools/open-ports/ on your laptop for testing and taking note of your remote IP. Share your address between each others
#### Server (game host)
1. Make sure you are not in the LAN room on Among Us;
2. Open the mobile terminal and create a shortcut to this commands:

`socat tcp-listen:47777,reuseaddr,keepalive,broadcast,fork udp:localhost:47777 & socat tcp-listen:22023,reuseaddr,keepalive,broadcast,fork udp:localhost:22023 & socat udp-recvfrom:47777,reuseaddr,keepalive,broadcast,fork tcp:FIRST.CLIENT:REMOTE.IP:47777`
#### Explication
Among Us uses UDP communication to create host and connect between players while using LAN. UDP packets can be send/receive only if the devices are on the same network. We need to use socat to listen port TCP 47777 and 22023, since TCP can be used over internet. The parameter reuseaddr allows socat to keep communicating and don't close the socket after the first response (same for keepalive and broadcast? I use them as reinforcement lol); fork tunnel the packets and the communications to a different format. This command starts three socat processes: the first two to listen TCP port and convert the packets in UDP, while the last one convert the broadcast message send by the Among Us server (the name that appear in the list of games available) over TCP.

#### Client (player) [UPDATED 17/07/2021]
1. Make sure you are not in the LAN room on Among Us;
2. Open the mobile terminal and create a shortcut to this commands:

`socat tcp-listen:47777,reuseaddr,keepalive,broadcast,fork udp:localhost:47777 & socat udp-l:22023,reuseaddr,broadcast,fork tcp:SERVER.REMOTE.IP:22023 & socat udp-l:22023,reuseaddr,broadcast,fork tcp:SERVER.REMOTE.IP:22023 & socat udp-recvfrom:47777,reuseaddr,broadcast,fork tcp:NEXT.CLIENT.REMOTE.IP:47777`
#### Explication
For the client we need to open a TCP port 47777 and tunnel the broadcast message as UDP; since we are tunneling packets, this it does not keep the original IP of the host, but instead it reads the packets as being send from the same remote IP of the client; to prevent that, the second socat use udp-l to "catch" the packets and tunnel them back to the server remote ip, where they will be then converted by the socat server in UDP packets! **With the third socat, we then once again send the packets receive from the server to another client.**

#### *If you have iOS 9 or higher
You can use ZeroTier as more "elegant" solution that does not required port forwarding. Still need the use of Terminal and socat.
1. From a computer, go on the ZeroTier website and create a free account
2. Follow the manual on how to create a network (I suggest you stick to private server) and keep the website open for now.
3. Download ZeroTier One on both client and server devices
4. Connect the server device to the server with ZeroTier One app
5. Go back to the ZeroTier setting website and authorized the new connection and take note of each address in the Managed IP column (Note: You can give a name to the device to recognize to how belong the IP)
6. Now connect the client and repeat step 5
7. Once you are sure both devices are connect to the same VPN server, open the server (host) mobile terminal and type this command:

`socat udp-recvfrom:47777,reuseaddr,keepalive,broadcast,fork udp:first.client.ip.from.zerotier:47777`

8. On the first client then, run this command:

`socat udp-recvfrom:47777,reuseaddr,keepalive,broadcast,fork udp:second.client.ip.from.zerotier:47777 & socat udp-l:22023,reuseaddr,broadcast,fork udp:server.ip.from.zerotier:22023`

9. Repeat step 8 for every client you want to connect, remembering that you have to change the "upd:second.client.ip.from.zerotier" with the IP of the next client.

# Final Note
Unfortunately, I coudn't do a proper test since I don't have access to another network. I've decided to forward a 3rd port on my router to tunnel back the host UDP broadcast message via TCP to test if everything works. And it does!
The major results are with ZeroTier: having host on wifi and two clients on mobile network, I've managed to both see the host and connect to it!
