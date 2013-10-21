SSH VPN
=======

A script to make it easier to use OpenSSH "VPN"s.  Background in my [blog post](http://blog.justinsb.com/blog/2013/10/21/the-best-vpn-is-the-one-you-use/)

OpenSSH supports tunnels, over which you can route some or all of your network traffic.  Because it's over SSH, the traffic is encrypted.  Because it uses SSH, setup/key management is much easier.  If you have a server, this is a very easy way to get a quick VPN-like experience.  Using OpenSSH directly can be a little painful.  This script makes it easier.

This script should work with Linux servers and Linux clients.  OS X clients are possible, but are not yet implemented; it will definitely require the Tun/Tap driver to be installed.  Windows support is theoretically possible, but there's no default SSH client on Windows (?).

Normally tun/tap requires root on the box; instead we create the tun/tap devices in advance, using sudo.  So you need sudo on the server, not root.  You need sudo on the client also.

There's not a lot to this; we just find an unused tunnel device on the server and on the client, assign IP addresses using a private IP range, and then allow NAT.

It's written in Python, because Python is a bit easier (for me) than alternatives that run on most operating systems.  Feel free to reimplement in your own language of choice - this is intended mostly as a proof of concept!

Running it
==========

You'll need a Linux server, to which you can ssh without typing in your password.  (If you don't know how to do this, check out ssh-copy-id)

Create a configuration file on your local machine:

```
SERVER=<MYSERVERIP>
mkdir -p ~/.ssh/vpn/
echo > ~/.ssh/vpn/myvpn <<EOF
server=${SERVER}
EOF
```

Replace <MYSERVERID> with the IP address of your server.  Note that you want to use the IP address of the server (not a DNS name).  This is because we have to change the routing tables so all traffic goes over the tunnel, except for the traffic to the VPN server itself!

You must be able to sudo on the server, without a password.

You'll probably want to create a sudo file on the server: ```/etc/sudoers.d/<yourusername>``` which looks like this:

```
<yourusername> ALL= (ALL) NOPASSWD: ALL
```

Also on the server, you'll want to make sure your /etc/ssh/sshd_config has ```PermitTunnel yes```.  You need to ```/etc/init.d/ssh reload``` after you change this.

To check, you should now be able to do this without typing in a password: ```ssh ${SERVER} sudo date```

Finally, run the script itself (on your client):

```
./ssh-vpn up myvpn
```

I suggest then visiting a site like [WhatIsMyIp](http://www.whatismyip.com).  It shows your location, which should be the location of your server.

When you're done with the VPN:

```
./ssh-vpn down myvpn
```

```myvpn``` is just an identifier.  You should be able to have multiple VPN configurations.  It's not likely you'll want to.  Running multiple VPN tunnels at once will probably break and is also very unlikely to be useful.

Troubleshooting
===============

If the VPN doesn't work, first check if you can ```ping 8.8.8.8```.

If you can ping 8.8.8.8, but you don't have e.g. web access, then check that your DNS servers are set correctly.  I recommend using 8.8.8.8 and 8.8.4.4

Feel free to open an issue if it doesn't work.  Please paste the output from ```./ssh-vpn up myvpn```

Security
========

__Cons__

This approach is unsuitable for running with untrusted users (as you have to give them sudo permission).

The VPN is not anonymous, because it does nothing to encrypt the outgoing traffic from your server.  The NSA or MPAA or whoever it is can easily see this traffic, and they can ask your server provider who you are.  This is not useful for doing anything illegal.

__Pros__

Traffic to your server is encrypted.  If you're on shared wifi (coffee shop, airplane) this will stop others snooping your traffic.

This approach can support IPv6, so if you have an IPv6 server you can get IPv6 no matter where you are.  The code isn't there yet, but I've put in a placeholder for where it goes.

It is incredibly easy to set up, and it is as secure as SSH.

If you run your own server, then if someone wants your key, they have to ask you.  But again, remember that it's trivial to look at the unencrypted outgoing traffic.  And a determined attacker can probably get the key another way.
