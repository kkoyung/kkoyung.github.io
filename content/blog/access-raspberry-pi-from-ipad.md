+++
title = "Access Raspberry Pi from iPad"
date = "2024-09-11T10:18:10+08:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["raspberrypi", "ipad"]
+++

I usually work on my desktop computer, so I don't have a laptop to work in a remote location. Two months ago, I had to go to Estonia to participate in [ICALP 2024](https://compose.ioc.ee/icalp2024/) to give a talk about my first published conference paper. So, I decided to set up a VPN to allow me to access my desktop computer through remotely from my iPad (10-th generation) using SSH during the trip. This allowed me to edit the slides (written in LaTeX), recompile them to PDF, and download the PDF to my iPad. This workflow worked fine, but I was not 100% satisfied with it. If anything had happened on my desktop or the network connection, it was possible that I couldn't fix it remotely. I was lucky that things went well during my trip.

Recently, I started looking for alternative solutions. I don't prefer to buy a new laptop because it would go to waste if I don't use it regularly. Instead, I want to further utilize my iPad, and I found two Linux terminal apps, [iSH](https://ish.app/) and [a-Shell](https://github.com/holzschu/a-shell). They work quite well for some simple use cases, but they are still not complete Linux systems.

I then found a [YouTube video](https://www.youtube.com/watch?v=IR6sDcKo3V8) describing how to directly access a Raspberry Pi from an iPad Pro through a single USB-C-to-USB-C cable. It works like this: First, connect the Raspberry Pi to an iPad Pro with a USB-C-to-USB-C cable. The iPad Pro powers up the Raspberry Pi, and the Raspberry Pi is configured as an Ethernet gadget, so the USB-C-to-USB-C cable between them acts like an Ethernet cable. Then, we can access the Raspberry Pi from the iPad Pro through a simple SSH connection. It means you can bring a full Linux system along with my iPad when you go out.

I was really interested in this idea, so I did some research on the Internet. Most of the videos and documents about this setup used Raspberry Pi 4. A [comment](https://www.hardill.me.uk/wordpress/2023/12/23/pi5-usb-c-gadget/#comment-30693) of a blog post mentioned that USB-C port of Raspberry Pi 5 did not work in this setting, but the issue was later resolved by a Raspberry Pi OS update. (See more in this [GitHub issue](https://github.com/raspberrypi/bookworm-feedback/issues/77).) I also found a [GitHub repository](https://github.com/verxion/RaspberryPi/) documenting a successful setup with Raspberry Pi 5. Furthermore, since the 10-generation iPad has a USB-C port, it seems possible to do this on my iPad.
There is only one way to confirm whether it works or not, so I ordered a Raspberry Pi 5.

It took over a week for my Raspberry Pi to arrive at my home. While I was waiting, I gathered some information and wrote myself a note about the setup so that I could test it out as soon as it arrived. The setup is rather simple. It is so simple that it worked really well on my first try.

Here is my setup, which is a combination of those from the references listed at the bottom of the blog post. All you need to do is to edit `/book/firmware/config.txt` and `/boot/firmware/cmdline.txt` to configure the Raspberry Pi as an Ethernet gadget, and add a file to `/etc/network/interfaces.d/` to configure the network interface.

1. Install Raspberry Pi OS on a microSD card with [Raspberry Pi Imager](https://www.raspberrypi.com/software/). Just follow the instructions from Raspberry Pi Image. Remember to provide SSH `authorized_keys` if you want to access it with public key authentication.

2. Insert the microSD card into the Raspberry Pi, boot the Raspberry Pi, and access it in your perferred way.

3. Add the following to the `/boot/firmware/config.txt`, **at the bottom, after** `[all]`.

    ```plaintext
    dtoverlay=dwc2,dr_mode=peripheral
    ```

4. Add the following to `/boot/firmware/cmdline.txt`, **after** `rootwait`.

    ```plaintext
    modules-load=dwc2,g_ether
    ```

5. Configure network as a USB Ethernet gadget. Add file `/etc/network/interfaces.d/g_ether` with the following content.

    ```plaintext
    auto usb0
    allow-hotplug usb0
    iface usb0 inet static
            address 169.254.1.1
            netmask 255.255.0.0

    auto usb0.1
    allow-hotplug usb0.1
    iface usb0.1 inet dhcp
    ```

6. (Optional) Configure a similar network for the Ethernet port. Add file `/etc/network/interfaces.d/eth0` with the following content.

    ```plaintext
    auto eth00
    allow-hotplug eth00
    iface eth00 inet static
            address 169.254.1.1
            netmask 255.255.0.0

    auto eth0.1
    allow-hotplug etc0.1
    iface usb0.1 inet dhcp
    ```

Now, when you connect the Raspberry Pi to the iPad with USB-C-to-USB-C cable to boot the Raspberry Pi, a new item called "Ethernet" should show up between "Wi-Fi" and "Bluetooth" in the "Settings" app on iPad. Then, you can access the Raspberry Pi with any SSH client app (I use [Termius](https://termius.com/)) through IP address 169.254.1.1.

## Reference

- [RaspberryPi/Pi5-ethernet-and-power-over-usbc.md at main · verxion/RaspberryPi](https://github.com/verxion/RaspberryPi/blob/main/Pi5-ethernet-and-power-over-usbc.md)
- [Network Manager fallback to zeroconf - Raspberry Pi Forums](https://forums.raspberrypi.com/viewtopic.php?p=2203146#p2203146)
- [STICKY: Bookworm, Point to Point Ethernet (inc g_ether) - Raspberry Pi Forums](https://forums.raspberrypi.com/viewtopic.php?t=364247)
