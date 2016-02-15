  ---
template: post
title: How to host a web server using Caddy on a Raspberry Pi
sitename: Blog Blair
---

#### Objectives
1. Use that Raspberry Pi (RPi) you have collecting dust!
2. Install Golang on your RPi
3. Use a light weight server called [Caddy](https://caddyserver.com)
4. Explore some basic networking

#### Installing Golang

This tutorial will be given through the command line so it is suggested to run the Raspberry Pi "headless"... meaning no desktop

Now that your RPi is up and running on the internet you should connect via SSH. If you have never SSH'd into a computer before check out my short post on it [here](/how_to_ssh)

To begin installing Golang download the compiled version from [Marek Kubica](https://xivilization.net/~marek/blog/2015/05/04/go-1-dot-4-2-for-raspberry-pi/)!

````
wget https://xivilization.net/~marek/raspbian/xivilization-raspbian.gpg.key -O - | sudo apt-key add -
sudo wget https://xivilization.net/~marek/raspbian/xivilization-raspbian.list -O /etc/apt/sources.list.d/xivilization-raspbian.list

sudo aptitude update
sudo aptitude install golang
````

1. Aptitude - A package manager for linux
2. Go 1.4 compiled by our friendly neighborhood RPi [user]


You done? Awesome. This install should default to your `$HOME` directory.
First set your `$GOPATH` to where ever you decide to keep golang.
When running Go we also need to have the `go/bin` as one of our default `PATH`s so we can easily run programs and make use of Go's great tooling.
If you are curious the `$PATH` actually owns all of the binary executions that your Raspberry Pi uses. They are each separated by ':'. In the code below you can see we are just shifting our `$GOPATH/bin` to the front of the string. Almost seems a bit easier to concatenate strings in bash than in Golang...
Open your bash profile `nano ~/.bashrc` then place the following at the end of the file - use VIM or EMACs if you got skillz

  ````
  $ export GOPATH="${HOME}/go"
  $ export PATH="${GOPATH}/bin:${PATH}"
  ````

#### Installing and using Caddy

  One of the great things about Golang is even though it is a bit lower level it has many of the tools of a more modern language. One of those being a built in package installer `go get`

  ````
  $ go get github.com/mholt/caddy
  ````

  Type in `$ caddy` and your server should be up and running. Enter localhost:2015 into your browser and you should see a 404 error. This should occur if your caddy server is working

  If you were only concerned with installing caddy on to your RPi then you have completed the tutorial! Please see [here](https://caddyserver.com/docs) for the caddy docs!

  Still here? Let's look at how to configure your caddy server!
  In order to configure your caddy server you will need to use a Caddyfile
  One of the main benefits of using Caddy is the simplistic and intuitive syntax used in your Caddyfile

  Basic configuration using Caddy:

```
    1  localhost {
    2  root blog/
    3  gzip
    4  ext .html .pdf .md
    5  }
```

`localhost {` declares where you would like Caddy to run the server.
The current configuration is equivalent to this `localhost == (your local IP address):2015`. It is also important to note that Caddy runs on port 2015 by default.
`root blog/` sets the root of the site to a directory
Example file structure below:

<img src="/images/Caddy_file_struct.png" height="300px"/>
  <!-- ![Caddy file structure](/images/Caddy_file_struct.png) -->

`gzip` allows for your web server to compress the contents of its responses to clients allowing for smaller files to be sent. This speeds up load times making your website more responsive.
`ext .html .pdf .md` this will allow your web server to respond to requests without the listed extensions

  Once a Caddyfile is created we can get to work on our website. Adding an index.html file would make sense given Caddy automatically looks for this as your '/' path
    - If you add "Hello world!" to your index.html file you should see this at localhost:2015 when you run `caddy` within the file directory that has your Caddyfile

  If you have made it this far nice job setting up a server on your RPi! Most will probably say "This is great but I want this to host my website...like on the internet not just in my Mom's basement". OK maybe only I said that but lets go over how to get this thing online.

#### Connecting the server to the internet

First we need to set up port forwarding and a static IP address. If you are a bit light on your know how of port forwarding or static IP addresses take a look at my previous blog [post](how_to_ssh)
Make sure your Raspberry Pi has a static IP address
Go to your port forwarding page on your router's admin interface and create a new port forwarding line item
Have your router forward all requests on port 80 to your Raspberry Pi web server at its own port 80 (port 443 if it is https://)
HTTP requests come over port 80 and is generally accepted as the port used for serving web requests
With client requests coming in over the internet we need to make sure our Caddy server is listening on port 80. If you recall from earlier our server is running on port 2015.


Due to the simple syntax of our Caddyfile it is a quick fix:

```
    1  :80 {
    2  root blog/
    3  gzip
    4  ext .html .pdf .md
    5  }
```
The small change to line 1 should do the trick...at least for our Caddyfile

Unfortunately many machines will attempt to protect lower port numbers that have assigned responsibilities - port 80 is one of these protected ports

We can override this using the libcap package - this will allow you to set priviledges for specific binaries. Install libcap using the script below:
```
      sudo apt-get -f install libcap2-bin
```
  Now that this package is installed we can adjust the permissions of the binary
  Navigate to the binary file directory `go/bin/` then run the following code

```
      setcap cap_net_bind_service=+ep ./caddy
```

  Now with the permissions set go ahead and run `caddy` from the directory with your Caddyfile


```
      Activating privacy features... done.
      :80
```

  Your Caddy server should be up and running! Type in your router's IP address in your browser and it should redirect you to your "Hello world".

<!--
go through how to set up custom caddy (modifying/rebuilding the binary)
- git pull
- markdown
- templates -->
