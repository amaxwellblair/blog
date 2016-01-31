---
template: post
title: How to SSH
sitename: Blog Bear
---

    <br>
    <div class="container">
      <h3>Objectives</h3>
      <ol>
        <li>Use that Raspberry Pi (RPi) you have collecting dust!</li>
        <li>Learn how to use SSH on OSX</li>
        <li>Explore some basic networking</li>
      </ol>
      <p>
        Note: The Raspberry Pi Foundation has done a great job
        <a href="https://www.raspberrypi.org/documentation/remote-access/">documenting</a> how to login remotely to your little princess...but for the sake of being thorough
      </p>
      <h3>SSH connection on your local network</h3>
      <ol>
        <li>Go ahead and turn you RPi on. If your RPi is up and running it should have an IP address by this point (if connected to the internet). We will need this in order to connect over the LAN to your RPi.</li>
        <li>We can check the IP by logging in using a monitor/keyboard or by checking the router connecting the RPi to the internet</li>
        <li>If you would like to do this via the command line inside your RPi:</li>
        <p><img width="600px" src="../images/find_local_ip.png"></img></p>
        <li>If you would like to do this "headless" you should log into your router using admin priviledges (if you have never done this a quick google search should get you up to speed) and check for devices connected to the router. Some familiar faces shoud be present - one of which should be your RPi.</li>
        <li>With your RPi address in hand you can locally connect to the server using SSH:</li>
        <br/>
        <code>$ ssh pi@0.0.0.0</code>
        <br/><br/>
        "0.0.0.0" representing your local RPi IP address
        <br/><br/>
      </ul><li>This will prompt you for your password the default password for your RPi is 'raspberry'</li></ul>
        <li>Once logged in this should bring you to the RPi terminal!</li>
      </ol>
      <p>Alright you are some hot stuff logging in remotely to your RPi... Now how would you do that when you are not in the LAN?</p>
      <h3>SSH connection from anywhere</h3>
      <p>The RPi Foundation suggests you use this: <a href="https://www.raspberrypi.org/documentation/remote-access/access-over-Internet/internetaccess.md">weaved</a></p>

      <p>I have nothing against weaved but one of our objectives is to experience some basic networking so this tutorial will do it the manual way</p>
      <ol>
        <li>Bring up your admin page for your local router again and you should see an option for "Port Forwarding" select this option</li>
        <p>Check out this great <a href="http://superuser.com/questions/284051/what-is-port-forwarding-and-what-is-it-used-for">post</a> if you just said "Wait...what is 'Port Forwarding'?"</p>
        <li>Once on the Port Forwarding page we want to add a new item with a type TCP/UDP and enable it for port forwarding</li>
        <li>The RPi and other computers listen on port 22 for incoming SSH requests. So should use port 22 in the port category of our newly created item</li>
        <li>Then enter in the RPi IP address we found earlier and start the port fowarding</li>
      </ol>
      <p>Now you should be able to connect to your RPi from anywhere! Enter in your IP</p>
      <code>$ ssh pi@0.0.0.0</code>
      <br/><br/>
      "0.0.0.0" representing your router's IP address
      <br/>
      <p>You can now test this out from your LAN</p>
    </div>