Ressource: http://doc.ubuntu-fr.org/sshfs

Install fuse

	sudo apt-get install fuse
	sudo adduser $USER fuse

Install mount point

	sudo mkdir /media/mountpoint
	chmod a+w /media/mountpoint

Start

	sshfs user@server:/home/user/targetdir -p 2222 /media/mountpoint