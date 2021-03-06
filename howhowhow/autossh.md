# Introduction

## Reminder

*autossh* is only a wrapper around ssh, handle connection restart when lost, with a monitoring port and so.. Port forwarding is a proper SSH feature.

## Ressources:

* https://wiki.archlinux.org/index.php/Secure_Shell
* http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man5/sshd_config.5?query=sshd_config
* https://gist.github.com/thomasfr/9707568
* http://blog.philippklaus.de/2013/03/start-autossh-on-system-startup-using-systemd-on-arch-linux/

## Client-side

### Installing

	sudo apt-get install autossh

or

	sudo pacman -S autossh

### Warning

Before starting autossh in a init script, you should better try it before in command line:

	$ autossh -M 0 -N -R 2222:localhost:22 -i /home/user/.ssh/id_ecdsa user@host
	The authenticity of host 'host (32.180.0.114)' can't be established.
	ECDSA key fingerprint is 09:60:c0:88:96:8b:97:e9:99:cf:b9:1f:26:08:7c:cd.
	Are you sure you want to continue connecting (yes/no)?

Say yes if it seems correct, now you can test it in an init script, go next step.

### Upstart 

It may be optional, but that's more explicit to make a symlink:

	sudo ln -s /etc/init.d/autossh /lib/init/upstart-job

Then write in `/etc/init/autossh.conf` file:

	description "autossh tunnel"
	author "Yann Le Breton"

	start on (local-filesystems and net-device-up IFACE=eth0) # assuming we have multiple interfaces
	stop on runlevel [016]

	respawn
	respawn limit 5 60

	exec autossh -M 0 -N -R 2222:localhost:22 -o "ServerAliveInterval 60" -o "ServerAliveCountMax 3" -i /home/user/.ssh/id_ecdsa user@server

Start the script:

	sudo service autossh start

If you wanna check it runs well on your client machine:

	sudo service autossh start

To debug possible errors in your upstart script:
	
	sudo tail -f /var/log/upstart/autossh.log

### Systemd

Edit a `/etc/systemd/system/autossh.service` script

	[Unit]
	Description=Keeps a tunnel to 'remote.example.com' open
	After=network.target
	 
	[Service]
	ExecStart=/usr/bin/autossh -M 0 -N -R 2222:localhost:22 -o "ServerAliveInterval 60" -o "ServerAliveCountMax 3" -i /home/user/.ssh/id_ecdsa user@server
	 
	[Install]
	WantedBy=multi-user.target

Reload the daemon list:

	systemctl daemon-reload

You then can run it by:

	systemctl start autossh

To check if it is well running:

	systemctl status autossh

To debug if you encounter some troubles:

	journalctl -f autossh

## Server-side

### Check the forwarded port

When you started your client autossh daemon, we can check on the server-side the port (2222 here) is open:

	$ sudo netstat -nltp | grep 2222
	tcp        0      0 0.0.0.0:2222            0.0.0.0:*               LISTEN      10820/sshd: user 
	tcp6       0      0 :::2222                 :::*                    LISTEN      10820/sshd: user

### Configuration

There is something to add to your `/etc/ssh/sshd_config`, please consider the documentation snippet above:

	GatewayPorts

	 Specifies whether remote hosts are allowed to connect to ports forwarded 
	 for the client.  By default, sshd(8) binds remote port forwardings to the
	 loopback address. This prevents other remote hosts from connecting to 
	 forwarded ports.  GatewayPorts can be used to specify that sshd should 
	 allow remote port forwardings to bind to non-loopback addresses, thus 
	 allowing other hosts to connect.  The argument may be “no” to force remote 
	 port forwardings to be available to the local host only, “yes” to force 
	 remote port forwardings to bind to the wildcard address, or
	 “clientspecified” to allow the client to select the address to which the 
	 forwarding is bound.  The default is “no”.

Add this to your `/etc/ssh/sshd_config`:

	GatewayPorts yes

To enable port forwarding on external IPs.
If you disabled it, you would only be allowed to perform such command:

	ssh localhost -p 2222

from your server to test the port forwarding.
