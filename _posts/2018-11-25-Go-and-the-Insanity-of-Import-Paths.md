---
title: Go and the Insanity of Import Paths
published: true
---

Today I got back on creating my own linux distribution. I am using [gentoo](https://gentoo.org) as a base for creating a binary distribution that is built from my current gentoo configuration.

As a start I am trying to host a portage binhost that serves prebuilt gentoo packages. I was searching for a light http server that can create a directory listing (for `/usr/portage/packages`, `/etc/portage`, etc.) with an optional `.tar.gz` download link to download the whole directory. I found [jpillora/serve](https://github.com/jpillora/serve) which is an awesome project that does exactly what I need.

I found the default listing a bit confusing as the sorting of directories and files was case sensitive. I forked the project and tried to implement a commandline flag that makes sorting case insensitive.

After a couple of minutes I knew what to change in the source code and tried to adjust it to my needs. I compiled the source and ran it... But nothing changed. I added prints and commented out a whole lot of the original source... But still, the code was working as if I haven't changed anything! Then I realized that changes to the `main.go` file are reflected to the `go run .` command. This was the point where I really started to question my sanity! From my point of view the `main.go` file was the only file that could be changed in the project. Until now I have not encountered a single compiler error!!

After asking a friend of mine for help ([@jwuensche](https://github.com/jwuensche)) I realized that I did the mistake of running `go get github.com/fin-ger/serve` instead of adding a new remote to `github.com/jpillora/serve`. All import paths in the project were pointing to `github.com/jpillora/serve/...` which lead to not importing **my** code from the `github.com/fin-ger/serve/main.go` but including the packages from the original author.

After some search I realized that this is actually **common practice** in go! This is the way to import local relative sub-packages... This actually drives my fork of the project useless as you **cannot** `go get` it anymore and use it e.g. as a replacement until my pull request is accepted on upstream. I found **no** solution to this problem. It seems like this is just accepted as "common practice".

I mean... **WHAT THE FUCK??!??!????**
