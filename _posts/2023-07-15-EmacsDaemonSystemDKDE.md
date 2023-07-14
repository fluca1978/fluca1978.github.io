---
layout: post
title:  "Emacs as a Daemon within KDE Plasma using SystemD"
author: Luca Ferrari
tags:
- emacs
- KDE
- systemd
- linux
permalink: /:year/:month/:day/:title.html
---
A simple configuration of Emacs to run as a daemon for a specific user, and then to enable it as a client on KDE Plasma.

# Emacs as a Daemon within KDE Plasma using SystemD

Emacs has the built-in capability to run as a *daemon*, that is a server application listening for incoming connections. The usage of a daemon speeds up dramatically the application startup time, especially if Emacs is configured to be launched at the user login.
<br/>
In this article I explain how to configure SystemD to run Emacs as a daemon, therefore starting it on startup, and then how to configure KDE Plasma to run the client counterpart.

## Configure SystemD to run the Emacs daemon

In order to exploit the daemon, and therefore its *client* counterpart, it is possible to configure SystemD to start the daemon as a *per-user* service. To such an aim, place a file like the following into `~/.config/systemd/user/emacs-daemon.service`:

<br/>
<br/>
```shell
[Unit]
Description=Emacs running as a daemon
Documentation=info:emacs man:emacs(1) https://gnu.org/software/emacs/

[Service]
Type=forking
ExecStart=/opt/emacs/emacs28.2/bin/emacs --daemon
ExecStop=/opt/emacs/emacs28.2/bin/emacsclient --eval "(kill-emacs)"
Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
Restart=on-failure

[Install]
WantedBy=default.target
```
<br/>
<br/>

The important part is, clearly, the `ExecStart` line that tells Emacs to run as a daemon by means of the `--daemon` argument.
The `Environment` allows you to specify the environment variables, most notably, the SSH Agent socket, so that Emacs will be able to pick up an SSH agent if anyone is running.

<br/>
<br/>
With the above SystemD configuration file in place, it is now possible to start the service by enabling it:

<br/>
<br/>
```shell
% systemctl --user enable emacs-daemon
% systemctl --user start emacs-daemon
```
<br/>
<br/>

Please note that you need not to be running Emacs to start the daemon, therefore, gosh, you need to *leave Emacs for a while* while doing this configuration!

## Configuring KDE Plasma to run the Emacs client

It is now possible to add a new item to the KDE Plasma menu, by right clicking on the menu icon and selecting `Edit Applications`.

In the window that pops-up, add a new item with your beloved description. Make the executable to point to the right path where `emacsclient` is, and usually this is the `bin` directory of your Emacs installation. Add the `--create-frame` command line option: `emacsclient` wants a file to fireup, while with the `--create-frame` option it will start as a *fresh* Emacs window (frame, in Emacs terminology), starting at the scratch buffer or whatever is the configured first page.


<br/>
<center>
<img src="/images/posts/emacs/emacsclient1.png" />
</center>
<br/>

Now you can launch Emacs thru the application icon, and the ending result will be Emacs running as a client while connected to your running daemon.


# Conclusions

Setting up Emacs to run in daemon is simple, thanks to the ability of SystemD to run services on a per-user basis.
Configuring KDE Plasma to make Emacs running as a client is, similarly, straightforward. One drawback of the `emacsclient` approach is that the KDE Plasma task manager does not recognize well a running `emacsclient`, spawning a new icon in the bar instead of using the same icon.
