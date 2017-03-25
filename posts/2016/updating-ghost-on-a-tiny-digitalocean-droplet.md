I rarely run into problems running a few web apps on the smallest DO droplet, but upgrading Ghost is one of those times where I need to do a bit of work in order for everything to work.

---

DigitalOcean is awesome and I recommend it to all the developers I know. [Here's a referral link](https://m.do.co/c/9b3b5a8e977b) if you've never used it.

Upgrading a [Ghost](https://ghost.org/) installation is a relatively simple process than, when updating Node modules can quickly consume more than the 500Mb of RAM that the tiny droplet has access to, which, beyond making the process take a long time, can crash npm and abort the upgrade.

Clearly, this is not what I wanted, so I needed to increase the amount of RAM that the OS has access to. I didn't want to upgrade the droplet just to upgrade this blog, so [swapfile](https://wiki.archlinux.org/index.php/swap) to the rescue.

Sidenote: though my VPS OS is Ubuntu, the Arch wiki is an incredible resource for things beyond the Arch Linux OS.

Adding more swap space (and thus more available memory) to the OS is done through a few commands:

```bash
fallocate -l 500M /tmp/swapfile
chmod 600 /tmp/swapfile
mkswap /tmp/swapfile
swapon /tmp/swapfile
```

In order, that

1. creates (allocates space for) a new file in `/tmp/swapfile` with [fallocate](http://man7.org/linux/man-pages/man1/fallocate.1.html)
2. sets the permissions on the file to owner read/write only
3. formats the file for swap with [mkswap](http://linux.die.net/man/8/mkswap)
4. mounts the file for use by the OS with [swapon](http://man7.org/linux/man-pages/man8/swapon.8.html)

Now, the OS has access to 500Mb of RAM and 500Mb of swap for if/when it goes over the RAM. Instead of npm crashing, the OS will use the swapfile to *swap out* RAM to and from the drive. Sure, this process can be slow, but I'm just installing app prereqs and the drive is an SSD, so it's not a problem.