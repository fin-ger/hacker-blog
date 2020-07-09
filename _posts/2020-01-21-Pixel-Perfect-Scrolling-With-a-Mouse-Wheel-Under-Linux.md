---
title: Pixel-Perfect Scrolling With a Mouse Wheel Under Linux
published: true
---

Yesterday, I plugged in my Logitech mouse into my laptop and started reading a PDF file. I started scrolling the page and realized that the scrolling is not as smooth as with my touchpad. The difference is that the touchpad scrolls the PDF pixel-by-pixel, whereas my mouse scrolls the PDF with one scroll-wheel click roughly 3 lines at once.

I previously had a look into the HID++ protocol of Logitech mice to [enable the thumb button of my M720](https://github.com/fin-ger/logitech-m720-config), and did know that my mouse has a high-resolution mode for the scroll-wheel. So, technically it is possible to get high-resolution scrolling data from my mouse and translate it to smooth pixel-perfect scrolling in my PDF viewer.

I searched the internet, read systemd source code, dug into libinput documentation, and finally found a way to achieve pixel-perfect scrolling! 🎉

## Here is how to set it up!

First you have to install the `solaar` package with your Linux distribution's package manager.

 - [Debian](https://packages.debian.org/sid/solaar)
 - [Ubuntu](https://packages.ubuntu.com/eoan/solaar)
 - [Fedora](https://apps.fedoraproject.org/packages/solaar)
 - [Gentoo](https://packages.gentoo.org/packages/app-misc/solaar)
 - [Arch](https://www.archlinux.org/packages/community/any/solaar/)
 - [Mageia](http://madb.mageia.org/package/show/release/cauldron/application/0/name/solaar)
 - [OpenSUSE](http://software.opensuse.org/package/Solaar)

After installing, follow the wizard to configure your unifying receiver and mouse. Now, you can use `Solaar` to configure some options on your mouse. The `Wheel Resolution` property enables the high-resolution mode on the mouse-wheel.

![Solaar configuration window](assets/solaar-config.png)

> Unfortunately, the wheel resolution property is not working with all unifying receivers. If it is not working, you can try restarting and disconnecting and reconnecting the unifying receiver to see if it works then, but some receivers of mine were no able to set the property. The receiver with `Bootloader 34.00.B0004` and `Firmware 04.02.B0009` worked best for me.

Alternatively, you can try to set the property via the command line:

```shell
$ solaar config "M720 Triathlon" hires-smooth-resolution true
```

If the `Wheel Resolution` property could be successfully set, you will recognize way too fast scrolling behavior. This can be changed by applying some `udev` `uwdb` rules.

Create the file `/etc/udev/hwdb.d/71-logitech-m720.hwdb` as **root** and open it in your preferred editor (also as root or with sudo). Insert the following lines and save the file.

```sh
mouse:usb:v046dp405e:name:Logitech M720 Triathlon:
mouse:bluetooth:v046dpb015:name:M720 Triathlon:
    MOUSE_WHEEL_CLICK_ANGLE=1
    MOUSE_WHEEL_CLICK_COUNT=360
```

Documentation on all available options for your mouse can be found [here](https://github.com/systemd/systemd/blob/05de16766b6bae290c500857269b753df2a0c649/hwdb.d/70-mouse.hwdb). A more up-to-date version may be available [here](https://github.com/systemd/systemd/blob/master/hwdb.d/70-mouse.hwdb) (might be broken in the future).

Next, we have to apply the changes. Run the following commands:

```shell
$ sudo udevadm hwdb --update
$ libinput list-devices | grep -A 1 M720
Device:           Logitech M720 Triathlon
Kernel:           /dev/input/event15
$ sudo udevadm trigger /dev/input/event15
```

> Replace `/dev/input/event15` with the result shown by `libinput list-devices**

**Congratulations!**

You got pixel-perfect scrolling with your mouse-wheel!

<video width="1000" controls>
  <source src="assets/m720.mp4" type="video/mp4">
</video>

> A before/after comparison of the proposed configuration

## Persisting the Configuration

The `hwdb` configuration will be automatically loaded when you plug in your mouse. The `Wheel Resolution` property is sometimes restored by Solaar, and sometimes not. Try to put Solaar in the autostart of your desktop environment. It might work. Otherwise, you have to manually enable the property 😢

## Enhancements and Future Work

The scrolling can stutter or jump when scrolling really slow. Also, when the middle mouse button is clicked you may unintentionally scroll, as the sensitivity of the scroll-wheel is so high. Maybe, this can be solved by applying a filter with libinput on the incoming scroll events. I have not figured out, how to set this up, yet.
