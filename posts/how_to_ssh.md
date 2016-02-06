---
template: post
title: How to SSH
sitename: Blog Blair
---

####Objectives
1. Use that Raspberry Pi (RPi) you have collecting dust!
2. Learn how to use SSH on OSX
3. Explore some basic networking

Note: The Raspberry Pi Foundation has done a great job
<a href="https://www.raspberrypi.org/documentation/remote-access/">documenting</a> how to login remotely to your little princess...but for the sake of being thorough

####SSH connection on your local network

1. Go ahead and turn you RPi on. If your RPi is up and running it should have an IP address by this point (if connected to the internet). We will need  in order to connect over the LAN to your RPi.
  - We can check the IP by logging in using a monitor/keyboard or by checking the router connecting the RPi to the internet
  - If you would like to do this via the command line inside your RPi:
<img width="600px" src="../images/find_local_ip.png"></img>

  - If you would like to do this "headless" you should log into your router using admin priviledges (if you have never done this a quick google search ld get you up to speed) and check for devices connected to the router. Some familiar faces shoud be present - one of which should be your RPi.

2. With your RPi address in hand you can locally connect to the server using SSH:
```
$ ssh pi@0.0.0.0
```
"0.0.0.0" representing your RPi IP address
This will prompt you for your password the default password for your RPi is 'raspberry'</ul>
Once logged in this should bring you to the RPi terminal!

Alright you are some hot stuff logging in remotely to your RPi... Now how would you do that when you are not in the LAN?
####SSH connection from anywhere
The RPi Foundation suggests you use this: <a href="https://www.raspberrypi.org/documentation/remote-access/access-over-Internet/internetaccess.md">weaved</a>

I have nothing against weaved but one of our objectives is to experience some basic networking so this tutorial will do it the manual way

1. Bring up your admin page for your local router again and you should see an option for "Port Forwarding" select this option. Check out this great <a href="http://superuser.com/questions/284051/what-is-port-forwarding-and-what-is-it-used-for  ">post</a> if you just said "Wait...what is 'Port Forwarding'?"

2. Once on the Port Forwarding page we want to add a new item with a type TCP/UDP and enable it for port forwarding
  - The RPi and other computers listen on port 22 for incoming SSH requests. So should use port 22 in the port category of our newly created item
  - Then enter in the RPi IP address we found earlier and start the port forwarding

3. Now you should be able to connect to your RPi from anywhere! Enter in your IP:
  ```
  $ ssh pi@0.0.0.0
  ```
  "0.0.0.0" representing your router's IP address

You can now test this out from your LAN

####Static IP address

-  Your router typically assigns IP addresses dynamically to new devices that are found on the network. Unfortunately this will change your Raspberry Pi's IP address from time to time. Although this is much better than the alternative which is manually adding new guests to your network every time a friend comes over and wants some wifi.

- So if we expect our RPi to be at the same address when ever we SSH we may need to do a bit more work

  - Depending on your router it could be as simple as locating the device on your admin page and enabling a static IP address

  - In a situation where this is not an option several different sites outline how the process is done [here is one example](http://elinux.org/RPi_Setting_up_a_static_IP_in_Debian)
