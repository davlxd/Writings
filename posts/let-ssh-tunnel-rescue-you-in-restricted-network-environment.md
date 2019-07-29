## Let SSH Tunnel Rescue You in Restricted Network Environment
2016-08-27

Networking policies in traditional companies are very strict due to its traditional nature. This may mean a lot from administrative or audit point of view, but as a developer, it makes me very difficult to apply DevOps practices like build CD pipeline, or at least deploy software to multiple environments automatically. And I seriously doubt how much it contributes to information security comparing how much trouble it makes.

One of many tough scenarios we've faced is this: we have 3 servers, `workstation` is a Windows server, which can ssh to Linux server `A` and `B`, but `A` and `B` are network isolated from each other, we need `A` to access the HTTP server on `B`.

The optimal way to solve these kinds of problems is trying to solve at higher levels, like convince policy makers to lower their strictness as it hurts productivity, sadly it's always almost impossible, then we have to ask technology for help. Since the whole stack above OS is built on software so basically technology can do almost anything. But after all it’s local optimization, a twisted use case of technology.


Before I illustrate the solution by using SSH tunnel, let’s recap some basic concepts.


## Concepts about SSH Tunnel

SSH tunnel a.k.a. SSH port forwarding - as the name suggests - provides a way to forward connections to a local port, through SSH connection, to another server on behalf of SSH server, or the other way around, based on an established SSH connection. There are 3 kinds of SSH port forwarding: Local port forwarding, Remote port forwarding, and Dynamic port forwarding.


Let’s say Alice usually uses `laptop` to ssh to her  `ec2` on AWS like this: `[alice@laptop ~] $ ssh -p 22 alice@ec2` (with security group already configured to allow this connection).

**Local port forwarding** makes TCP connections to a specific port on `laptop` get forwarded to another server by `ec2`, the magic option is `-L`. Alice can run `[alice@laptop ~] $ ssh -L 3000:server:4000 -p 22 alice@ec2` on her laptop, during the SSH session, accessing port 3000 on `laptop` is kind of the same as `ec2` accessing port 4000 on `server`. 

For instance, there is a another Linux server `ec2_secret` to which `laptop` cannot ssh, but `ec2` can. After running `[alice@laptop ~] $ ssh -L 2200:ec2_secret:22 alice@ec2`, during the session, Alice can directly ssh to `ec2_secret` by running `[alice@laptop ~] $ ssh alice@localhost -p 2200`, instead of ssh to `ec2` then ssh to `ec2_secret` manually. This is `SSH Relay`.


**Remote port forwarding** is kind of in the opposite direction, with magic option `-R`. After running `[alice@laptop ~] $ ssh -R 3000:server:4000 -p 22 alice@ec2`, during the ssh session, accessing port 3000 on `ec2` is the same as `laptop` accessing port 4000 on `server`.

For both `Local port forwarding` and `Remote port forwarding`, `server` can be same as a forwarding machine (`ec2` for `Local port forwarding`, `laptop` for `Remote port forwarding`), which is especially useful for `Remote port forwarding`. Consider this scenario: Alice usually leaves her laptop at home with a Wi-Fi network connection to a wireless router, when she goes to work, every now and then she wants to access her laptop by the PC in the company, what she gonna do? The pure command-line/SSH solution is very straightforward with the help of `Remote port forwarding`: Before she leaves home in the morning, she opens a `Remote port forwarding` ssh connection from her `laptop` to `ec2` like this `[alice@laptop ~] $ ssh -R 2200:localhost:22 -p 22 alice@ec2`, thus ssh connection to port 2200 on `ec2` itself will be forwarded to port 22 on `laptop` itself. When she arrives at company, she can ssh to `ec2` from her `PC` at first, then ssh to her `laptop` by `[alice@ec2 ~] $ ssh -p 2200 alice@localhost`.

**Dynamic port forwarding** opens a SOCKS proxy port on top of an established SSH connection, we people in China use this to bypass GFW very often. Let’s assume Alice gets very lucky and takes a business trip to China, after she lands she finds herself won’t be able to access Google, Facebook, Twitter etc. If she can still access `ec2`, by running `[alice@laptop ~] $ ssh -D 1080 -C -p 22 alice@ec2` in terminal where `-D` stands for **Dynamic port forwarding** and `-C` stands for compress payload data, opens a SOCKS proxy on port 1080 on her laptop. The world will come back to live again after she sets SOCKS proxy to `127.0.0.1:1080` for the browser.


## Connecting the Dots

After a full understanding of `Local port forwarding` and `Remote port forwarding`, the tough scenario at the beginning of this post can be easily handled by using `Local port forwarding` and `Remote port forwarding` together:
[alice@workstation ~] $ ssh -R 2000:localhost:3000 alice@A
[alice@workstation ~] $ ssh -L 3000:localhost:4000 alice@B

thus, server `A` can access HTTP service of port 4000 on server `B` by access http://localhost:2000. 

Furthermore, if HTTP port 2000 on server `A` needs to accept HTTP connection from other servers in the same network, just change `GatewayPorts` to `yes` in `/etc/ssh/sshd_config` on server `A`. 


In a word, if one machine can ssh to another machine, these 2 machines then closely associated with each other: they can exchange files by `scp`, one can provision another by `Ansible`, and by `SSH Tunnel` one can access another's resource on its behalf.

---

### Refers
- https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding






