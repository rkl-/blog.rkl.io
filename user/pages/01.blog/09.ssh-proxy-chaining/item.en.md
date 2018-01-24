---
title: 'SSH proxy chaining (tunnel through firewalls)'
media_order: chain-690088_1920.jpg
published: true
date: '24-01-2018 11:41'
taxonomy:
    category:
        - blog
    tag:
        - security
        - ssh
        - proxy
        - firewall
        - tunnel
---

This compact article digs into daily practice if you work with linux based
server systems.  Sometimes it is really important that special systems
have no public SSH access, they only exists inside your applications
network or your office or home LAN.

But, what is if you need access such a hosts shell from a public network
or from the internet itself?  Well, you could start to configure tricky
and ugly iptables, or you simply use the native SSH proxy chains.

What are proxy chains? Well, this is quite simple. It's a way through
one or n hosts until the host you wanna reach. In practice that means,
you simple ssh to your destination host, which is behind a private
network and ssh is doing the proxy tunneling under the hood.

A simple command like this: `ssh my-private-host` can establish a tunnel
over one or more hosts to finally reach `my-private-host`. All what you
need is a public accessible jumphost inside your private network and a
special configured ssh config file.

The other important thing regarding to the jumphost is, that it must
have netcat to be installed. The `nc` command is used by ssh to forward
your TCP-Stream. I use for example the `netcat-openbsd` package which
under debian like systems you can simply install with `apt-get install
netcat-openbsd`.

Let's now see how we configure such a proxy chain in our `~/.ssh/config`
file. The magic keyword here is the ssh's build in `ProxyCommand`. An
example configuration looks like the following:

```
Host my-private-host
    IdentityFile ~/.ssh/id_ecdsa.pub
    User romano
    ProxyCommand ssh -q <my-public-jumphost> nc <internal-ip-of-private-host> 22
```

The only line to explain here is `ProxyCommand ssh -q <my-public-jumphost>
nc <internal-ip-of-private-host> 22`.

`<my-public-jumphost>` is the public accessible ip address or the hostname
of our jumphost which is available from the internet or any other public
network where we have access to.

`<internal-ip-of-private-host>` is the internal IP address or hostname
of our protected host. For example, this could be simply a LAN IP address.

The `22` at the end of the line is the ssh port on the internal private
host.

The ssh `-q` flag enables the ssh quiet mode, which means that the most
warning and diagnostic messages are suppressed.

`nc` is netcat, which forwards the ssh TCP-Stream to the private host.

By setting up multiple host configurations with a ProxyCommand, it is
easy to jump over multiple host to your final destination.

Let's say we extend our configuration in this way:

```
Host my-private-host-01
    IdentityFile ~/.ssh/id_ecdsa.pub
    User romano
    ProxyCommand ssh -q <my-public-jumphost> nc <internal-ip-of-private-host-01> 22

Host my-private-host-02
    IdentityFile ~/.ssh/id_ecdsa.pub
    User tobi
    ProxyCommand ssh -q <my-private-host-01> nc <internal-ip-of-private-host-02> 22
```

If we do now a `ssh my-private-host-02`, we automatically login to
`my-private-host-01` which than proxy us to `my-private-host-02` as user
`tobi`.

This setup is usefull, if our public jumphost can not talk to
`my-private-host-02`, but `my-private-host-01` can.

Of course, in this case, `my-private-host-01` needs also the netcat package installed.

##### Use SSH agent forward
I strongly recommend, that you also place your local pubkey on all
private hosts you wanna reach and use for all connection the ssh `-A`
flag, which forwards your ssh key with the `ssh-agent` to all jumphosts
and the final host.

##### Conclusion
You may ask you, why you should not simply ssh to the jumphost and then to
the other host and so on. Well, in a quick and dirty setup, it's enough
in the most cases. But, imagine that maybe you want to copy files from
`my-private-host-02` with scp. The situation gets really tricky. With a
clean config setup as described above, you now simply can do things like
`scp <my-private-host-02>:/foo/bar /what/ever` without getting headache.

