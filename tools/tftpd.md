Ubuntu 20.04 TFTP Server Setup
------------------------------
Installing the tftp server is straightforward:
```bash
$ sudo apt-get install -y xinetd tftpd tftp
```
Then create something like this in `/etc/xinetd.d/tftp`:
```
service tftp
{
	protocol        = udp
	port            = 69
	socket_type     = dgram
	wait            = yes
	user            = nobody
	server          = /usr/sbin/in.tftpd
	server_args     = /tftpboot
	disable         = no
}
```
Whatever is in `server_args` needs to be created, with the ownership and
permissions set appropriately
```bash
$ sudo mkdir -v /tftpboot
$ sudo chown nobody /tftpboot
$ sudo chmod 777 /tftpboot
```
And then restart the Internet service daemon
```bash
$ sudo service xinetd restart
```

