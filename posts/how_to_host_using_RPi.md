---
template: post
title: How to host a web server using Caddy on a Raspberry Pi
sitename: Blog Blair
---
### Hosting a webserver with Raspberry Pi

The website you are currently viewing uses the set up herein described. I wanted to share my experiences so others could host on their own website!

##### Objectives
1. Use that Raspberry Pi you have collecting dust!
2. Install Golang on your Raspberry Pi
3. Use a light weight server called [Caddy](https://caddyserver.com)
4. Explore some basic networking

#### Installing Golang

One way to connect to your Raspberry Pi's command line is to use [SSH](how_to_ssh)

We will be installing a compiled version of Golang 1.4 from [Marek Kubica](https://xivilization.net/~marek/blog/2015/05/04/go-1-dot-4-2-for-raspberry-pi/)

````
$ wget https://xivilization.net/~marek/raspbian/xivilization-raspbian.gpg.key -O - | sudo apt-key add -
$ sudo wget https://xivilization.net/~marek/raspbian/xivilization-raspbian.list -O /etc/apt/sources.list.d/xivilization-raspbian.list

$ sudo aptitude update
$ sudo aptitude install golang
````

You done? Awesome. This installation should default to your `$HOME` directory

First set your `$GOPATH` to where ever you decide to keep Golang

When running Go we also need to have the `go/bin` as one of our default `PATH`s so we can easily run programs and make use of Go's great tooling.

If you are curious the `$PATH` actually owns all of the binary executions that your Raspberry Pi uses. Each `/bin` is separated by `:`. In the code below you can see we are just unshifting our `$GOPATH/bin` to the front of the list

Open your bash profile `nano ~/.bashrc` then place the following at the end of the file - obviously use VIM or EMACs if you prefer

````
$ export GOPATH="${HOME}/go"
$ export PATH="${GOPATH}/bin:${PATH}"
````

#### Installing and configuring Caddy

  Why Caddy?

  1. Simplistic configuration syntax made for humans (me)
  2. Runs on Golang (personal interest)
  3. Built in HTTPS (which I need to deploy)

##### Install
  One of the great things about Golang is even though it is lower level it has many of the tools of a more modern language. One of those being a built in package installer `go get`

````
$ go get github.com/mholt/caddy
````

  Run `$ caddy` from the command line and your webserver should be up and running!

  Enter `localhost:2015` into your browser and you should see a `404 error`


##### Configure
  Please see [here](https://caddyserver.com/docs) for the Caddy docs


  Let's look at the basics of configuring your Caddy server. Go ahead and `$ touch Caddyfile` in your chosen app directory

  One of the main benefits of using Caddy is the simplistic and intuitive syntax used in your Caddyfile

  Basic configuration inside your Caddyfile:

```
localhost {
  root blog/
  gzip
  ext .html .pdf .md
}
```

`localhost` declares where you would like Caddy to run the server

It is important to note that Caddy runs on port `:2015` by default

`root blog/` sets the root of the web application to a directory

Example file structure below:

<img src="/images/Caddy_file_struct.png" height="300px"/>
  <!-- ![Caddy file structure](/images/Caddy_file_struct.png) -->

`gzip` allows for your web server to compress the contents of its responses to clients allowing for smaller files to be sent. This speeds up load times making your website more responsive.

`ext .html .pdf .md` this will allow your web server to respond to requests without the listed extensions

Once a Caddyfile is created we can get to work on our website. Adding an index.html file would make sense given Caddy automatically looks for this as your '/' path

```
$ touch index.html
$ echo "Hello World!" > index.html
```

If you pull up your browser at `localhost:2015` again you should see your "Hello World!"


#### Connecting the server to the internet

First we need to set up port forwarding and a static IP address. Take a look at my previous blog [post](how_to_ssh) for more detailed instructions

Have your router forward all requests on port 80 to your Raspberry Pi web server at its own port 80 (port 443 if it is HTTPS). HTTP/HTTPS requests come over port 80/443 and is generally accepted as the port used for serving web requests

With client requests coming in over the internet we need to make sure our Caddy server is listening on port 80. If you recall from earlier our server is running on port 2015.


Due to the simple syntax of our Caddyfile this is a quick fix:

```
:80 {
  root blog/
  gzip
  ext .html .pdf .md
}
```
The small change should do the trick...at least for our Caddyfile

Many machines will attempt to protect lower port numbers that have assigned responsibilities - port 80 is one of these protected ports

We can override this using the `libcap` package - this will allow you to set privileges for specific binaries.

Install `libcap` using the script below:
```
$ sudo apt-get -f install libcap2-bin
```
Now that this package is installed we can adjust the permissions of the binary. Navigate to the binary file directory `go/bin/` then run the following code:

```
$ setcap cap_net_bind_service=+ep ./caddy
```

With permissions set go ahead and run `caddy` from the directory where your Caddyfile is located

```
$ caddy
  Activating privacy features... done.
  :80
```



  Your Caddy server should be up and running! Type in your router's IP address in your browser and it should redirect you to your "Hello World!".
