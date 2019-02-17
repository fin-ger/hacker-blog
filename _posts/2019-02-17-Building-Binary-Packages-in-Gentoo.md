---
title: Building Binary Packages in Gentoo
published: true
---

In gentoo it is possible to build binary packages from already installed packages to avoid recompiling them e.g. on another machine. Gentoo provides a package called `quickpkg` for this which can package single packages for binary distribution. It is also possible to package all installed packages with `quickpkg "*/*"`. The binary packages are then located in `/usr/portage/packages/`.

However when using `quickpkg` to package the whole system I strongly recommend to use `--include-config y` as by default `quickpkg` does not include config files in the binary packages. This will lead to erasing all configuration files from the machine that is using the binary host. This will most likely brick your system.

To use these binary packages on another machine you can simply host them on a http server and enter the server URI into the `PORTAGE_BINHOST` variable in your `make.conf`.

```bash
PORTAGE_BINHOST="https://user:password@my.own.server/"
```

I used a small go program to serve my binary packages which I bundled in a small docker container. You can use it to host your own binary packages and secure them with a http basic auth.

```
$ docker run -p 3000:3000 -v /usr/portage/packages:/content fin1ger/serve --auth user:password
```

The source code for the go program is located on [github](https://github.com/fin-ger/serve).

After all, remember always keeping a backup of your previous system as things can go wrong any time (not that *I* have bricked my system when doing this)...
