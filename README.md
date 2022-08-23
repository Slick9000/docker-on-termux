# alpine-on-termux
setup alpine on termux with qemu, as well as other things.

## ☕ donations

while completely optional, it would be highly appreciated if you could donate if you find this tutorial helpful! it can be of any amount.

[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://paypal.me/irbee246)

<br/>

## what is qemu?

if you are unaware as to what qemu is, qemu, is a free and open-source emulator, that is able to emulate machine processors with full operating system emulation, similar to other virtual machine applications such as virtualbox, allowing you to run one operating system on another. however qemu works natively as a package in termux.

<br/>

## why use qemu/a virtual linux environment on termux?

in my use case, i use qemu to run docker, which usually cannot run natively on termux due to limitations in the android kernel.

it also allows me to run programs which typically require a rooted device without having to root my phone (i prefer not to root my android device)

<br/>

## why alpine?

simply put, i tried ubuntu, but it either didn't want to boot at all, or it takes so long to boot in qemu i decided it wasn't the distro i wanted.

after some searching for other distros i decided to try alpine. alpine has a distribution called virt, which is designed specifically for virtual machines such as qemu and works very well. i was able to install and setup in less than 10 minutes. 

ubuntu is also around 2-3gb in size, while alpine's is much smaller (the iso for alpine virt is ~39mb, and when setup it's very small in size as well). while for the modern user this may not be problematic, it's still nice to prevent bloating your device storage.

<br/>

## instructions to install

after installing termux, run:

```
pkg update -y && pkg upgrade -y
```

this will make sure all termux packages are up to date.

```
pkg install x11-repo
```

x11-repo contains the package for qemu and it's tools.

```
pkg install qemu-system-x86_64 -y && pkg install qemu-utils
```

yay! you have qemu installed! now, to create a boot image for qemu to be able to read.

```
qemu-img create -f qcow2 hdd.img 16G
``` 

16G can be altered to any storage size you want, in gigabytes.

now, you need to actually download the file for alpine linux!

```
wget https://dl-cdn.alpinelinux.org/alpine/v3.16/releases/x86_64/alpine-virt-3.16.2-x86_64.iso
```

now, to actually run alpine linux in qemu! run:

```
qemu-system-x86_64 -cdrom alpine-virt-3.12.3-x86_64.iso -m 2048 -hda hdd.img -nographic
```

note: if you prefer to use vnc (like me, run:

```
qemu-system-x86_64 -cdrom alpine-virt-3.12.3-x86_64.iso -m 2048 -hda hdd.img -vnc 127.0.0.1:1
```

<br/>

**to explain these commands**:

-cdrom points to the iso of alpine

-m sets the amount of memory/ram alpine is allowed to use (for me, 2gb is plenty enough.)

-hda points to the boot image needed for qemu

-vnc creates creates a local vnc server for qemu to run through by the local ip and port of the device

once you're all booted up, it will now ask for a localhost login password. the pass is: root

```
setup-alpine
``` 

to perform the regular alpine setup. (personally, i usually set the username and password both as root, set the hdd.img as the sys storage partition, and leave everything else as default).

once setup, it will ask you to restart it to finish it's installation.

```
poweroff
```

to relaunch, run:

```
qemu-system-x86_64 -hda hdd.img -m 2048 -nographic
```

again, if you prefer to use vnc, run:

```
qemu-system-x86_64 -hda hdd.img -m 2048 -vnc 127.0.0.1:1
```

these will be the commands used for every boot from now on.

<br/>

unlock the full functionality of the package manager in alpine

to install some programs such as docker, you will need to unlock the community package in the apk package manager.

to do so, you'll need to edit the packages file using a text editor. i'll be using vim for this tutorial. install vim by running:

```
apk add --update vim
```

then:

```
vim /etc/apk/repositories
```

once in here, you must remove the # (comment character) from the second line, which says:
#http://mirror.reeningne.net/alpine/v3.16/community

this is a simple walkthrough as to how to edit the file with vim (for those familiar with vim, you can skip this part):

>  press i to enter insert mode

>  delete the comment, making it read:

http://mirror.reeningne.net/alpine/v3.16/community

>  press esc to exit insert mode

>  exit by pressing : (colon), typing wq and pressing enter.

now, you have unlocked the community packages not included in the main alpine installation such as docker.

<br/>

## setup docker on alpine.

now, you may think it would be as simple as just:

```
apk add --update docker
```

and then just:

>  docker run {package}

...right? well... no.

if you try to do this, you will receive the error:

Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

now the first thing you're probably thinking is: what the hell does that mean?

essentially, the docker package is a service, one which must run in the background at all times.

(similar to programs such as your system clock or weather programs).

for whatever reason, it is not automatically set as a service on install.

so you simply have to set it as a default service! you can do so by performing:

```
-update add docker default 
```

this will make the docker service run automatically on all future startups of alpine.

however to begin it without restarting, run:

```
rc-service docker start
```

and now you can use docker!

to see all existing docker containers, run:

```
docker ps -a
```

to stop a docker container, run:

```
docker stop {docker container id}
```

```
docker rm {docker container id}
```

<br/>

## setup a terminal multiplexor(tmux) in alpine

in the case that you would like to run multiple services at once in alpine, such as a discord bot, a discord server, and a plex server, it is recommended you get tmux.

to install, run:

```
apk add --update tmux
```

from there to create a window, you can do:

```
tmux
```

now everything you do in this environment is set to this window. do:

```
Ctrl-B + D 
```

to leave the window

to create a new window, simply do:

```
tmux
```

to reattach to a window, do:

```
tmux attach -t {session number}
```

to view all running windows, do:

```
tmux ls
```

to close a window, do:

```
Ctrl-B + X (while attached)
```

or run the command

```
tmux kill-session -t {session number}
```

to kill all sessions besides the specified session, do:

```
tmux kill-session -a -t {session number}
```

very useful if you open a lot of sessions at once.

<br/>

## setup curl in alpine

probably the easiest setup in this tutorial, simply run:

```
apk add --update curl
```

<br/>

## setup python in alpine

now everybody knows that having python in your back pocket is always useful, though it's not as simple as pkg install python is in termux. here are the commands needed to install it on alpine:

```
apk add --update --no-cache python3
```

```
ln -sf python3 /usr/bin/python
```

```
python3 -m ensurepip
```

```
pip3 install --no-cache --upgrade pip setuptools
```

<br/>

these are all of the setup tips i can provide for setting up alpine through qemu in termux, docker, tmux window manager, curl, as well as python. i use this setup everyday, and it runs flawlessly once all set up. i urge you to give this a try yourself, it's quite satisfying.

of course, it is also possible to set up a desktop environment for alpine and view it through vnc, but I won't go through how to set that up here. any youtube video can explain that. this was to assist with things that may be difficult to understand and setup in qemu and alpine for most users.

however, a list of compatible desktop environments can be found here on the [official alpine wiki](https://wiki.alpinelinux.org/wiki/Desktop_environments_and_Window_managers)!
