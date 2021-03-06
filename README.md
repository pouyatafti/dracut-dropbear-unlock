# dracut-dropbear-unlock

A minimalist dracut module that allows you to remotely unlock an encrypted root partition during boot. The module script is kept simple, so you are strongly advised to read it and see if it suits your needs.

All you get when you log in is a prompt for your encryption passphrase, for instance:

    <server>$ reboot
    Connection closed by remote host.
    <local>$ ssh myserver.com # fails
    <local>$ ssh myserver.com -p 2222
    Please enter passphrase for disk <your disk> (luks-<your uuid>)! ****************
    Connection closed by remote host.
    <local>$ ssh myserver.com # now succeeds

By default, the server uses port 2222, a 4096-bit RSA host key, only allows public key authentication, and only gives an encryption key prompt instead of a shell.

## Installation

First, you need to modify your kernel commandline to set up a minimal network during boot. If you use GRUB2, you need to add this to your `GRUB_CMDLINE_LINUX` in `/etc/default/grub`. A few examples:

	# Static IP and gateway on a specific interface (use stable names like enp0s20f0, not eth0 which might get renumbered after a reboot)
	rd.neednet=1 ip=<static IP>::<static gateway>:<netmask>::<interface name>:off:8.8.8.8:8.8.4.4

	# or just autoconfigure with DHCP on all interfaces
	rd.neednet=1 ip=dhcp

Finally install the dracut module, install your new GRUB2 settings and rebuild your initramfs:

	git clone https://github.com/mtthbfft/dracut-dropbear-unlock
	sudo mv dracut-dropbear-unlock /usr/lib/dracut/modules.d/42dropbear-unlock
	echo add_dracutmodules+="dropbear-unlock" > /etc/dracut.conf.d/dropbear-unlock.conf
	grub2-mkconfig -o /boot/grub2/grub.cfg
	dracut -f

Just a side note if you're running CentOS (other distros might have an equivalent service): you might need to `systemctl mask rhel-import-state` to prevent it from overwriting `/etc/resolv.conf` and your `/etc/sysconfig/network-scripts/` with your kernel cmdline settings at each boot (you might want more complex settings during normal runtime like IPv6, etc.)

## Contributing

Send a pull request. For comments/ideas, DM me on twitter (@mtth_bfft)
