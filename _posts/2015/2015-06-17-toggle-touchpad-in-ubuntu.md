---
layout: post
title: Toggle touch pad in Ubuntu
is_recommended: false
category: Linux
tags: ["ubuntu", "linux", "touchpad", "tutorial"]
---

In short, the hot key `Fn + F9` (or `F8`, whatever) to toggle enabling touch pad (track pad) does not always work due to Linux's poor driver support (It's the manufacturer's fault, not Canonical's).

I kinda came up with the following workaround.

# The basic

Basically, an unsupported touch pad will be emulated as a `psmouse` device. The following command will disable it.
```
sudo modprobe -r psmouse
```
To enable it, run
```
sudo modprobe psmouse
```
To check if the touch pad is enabled or not, check if `psmouse` is listed in the result of the following command
```
lsmod
```
# A little more advance

So I decided to write a bash script to toggle enabling touch pad, instead of typing different commands for that
```sh
#!/bin/bash
if [ "`lsmod | grep -o psmouse`" ];
then
    sudo modprobe -r psmouse;
else
    sudo modprobe psmouse;
fi
```
I put the script in `~/Programs/scripts/toggle-trackpad.sh`

Now, all I have to do is to run that script whenever I want to do the toggle. That doesn't really make my life easier. Then I came up with these ideas.

1. Using alias

    So, I put this in the alias section of my `~/.bashrc` file (I don't use a separate `~/.bash_aliases` file, since it's not really necessary)
    ```sh
    alias toggle-tp="if [ \"\`lsmod | grep -o psmouse\`\" ]; then sudo modprobe -r psmouse; else sudo modprobe psmouse; fi"
    ```
    Ok, that looks nicer. Hummm, but then I will have to open a terminal every time I want to toggle the touch pad. Not catastrophically annoying, but I want things better.

2. Using `Alt + F2` and `gksudo`

    Firstly, I install `gksu`. `modprobe` requires sudo access to work. Without a shell, `gksudo` is required.
    ```
    sudo apt-get install gksu -y
    ```
    Then, I change the alias to
    ```
    alias toggle-tp="if [ \"\`lsmod | grep -o psmouse\`\" ]; then gksudo 'modprobe -r psmouse'; else gksudo 'modprobe psmouse'; fi"
    ```
    After re-login, I can just press `Alt + F2`, then type `toggle-cp` to toggle. That's a bit better, but still, I just want a hot key.

3. Custom shortcut

    So finally, I decide to edit the script in to use `gksudo`
    ```sh
    #!/bin/bash
    if [ "`lsmod | grep -o psmouse`" ];
    then
        gksudo 'modprobe -r psmouse';
    else
        gksudo 'modprobe psmouse';
    fi
    ```
    Then config a new keyboard shortcut (I'm using `Ctrl + Super + T`)
    ```
    Name: Toggle Keyboard
    Command: /path/to/toggle-trackpad.sh
    ```
# Result

Now I can use a hot key to toggle enabling the touch pad. It's still a little annoying that every time I want to toggle the touch pad, I would have to enter my password. Fortunately, I don't have to do that too often. I use a mouse. It's also a little disappointing that I can't override the default `Fn + F9` on my machine, but that's minor. I'm happy enough.
