# Frame-it

Frame-it makes it easy to work with Ubuntu Frame on the desktop.

[![frame-it](https://snapcraft.io/frame-it/badge.svg)](https://snapcraft.io/frame-it)

## Using Ubuntu Frame "on the desktop"

Ubuntu Frame is designed for use to provide a single application GUI on Ubuntu Core, but it can be used on other systems including traditional desktops.

The following describes creating a desktop user that has access to a single application. In this example we will use Kodi, a popular media player.

The aim of this exercise is to create a desktop user that needs no sign-in and only has access to a single application.

For this we will be installing three snaps and configuring a “kodi” user account without a password.

### Install frame-it, ubuntu-frame and mir-kiosk-kodi

The snaps we will be using are:

1. `frame-it` which manages running Ubuntu Frame and the chosen application on the desktop;
1. `ubuntu-frame` which provides a Wayland shell; and,
1. `mir-kiosk-kodi` which packages Kodi for this type of use

The instructions to install these are:

    $ sudo snap install --classic frame-it
    $ sudo snap install ubuntu-frame
    $ sudo snap install mir-kiosk-kodi

On a desktop system you will need to connect the Wayland interface for `mir-kiosk-kodi`:

    $ sudo snap connect mir-kiosk-kodi:wayland

At this point you can use Frame-it to test that your application runs with Ubuntu Frame:

    $ frame-it mir-kiosk-kodi

But we want to do more, and configure a "user session" and user to frame Kodi.

### Configure a frame-it user session

We need to tell frame-it which application to use, in this case `mir-kiosk-kodi`:

    $ sudo snap set frame-it shell-app=mir-kiosk-kodi

Next we check the setup:

    $ frame-it.check
    Checking: Ubuntu Frame installation (needed for all commands)
    . OK . : Ubuntu Frame is installed
    Checking: Ubuntu Frame has login-session-control (needed to run a login shell)
    WARNING: ubuntu-frame:login-session-control NOT connected. Please run the following:
    sudo snap connect ubuntu-frame:login-session-control
    Checking: shell-app option (needed for frame-it.shell and to run a login shell)
    . OK . : shell-app is set (to 'mir-kiosk-kodi') and can be found

From this we see a WARNING that asking us to connect `ubuntu-frame:login-session-control`, so we do that now:

    $ sudo snap connect ubuntu-frame:login-session-control

We can also test the configuration for running Kodi on Ubuntu Frame with:

    $ frame-it.shell

This should start an Ubuntu Frame window and run Kodi in it. Having checked this works, close it.

At this point it is possible to log out and select “Frame-it” as the session when logging in.

But we set out to create a new “kodi” user that doesn’t need a password.

### Creating a "kodi" user

The first bit of creating a user we do by running the following commands (and pressing “enter” for the defaults):

    $ sudo adduser --disabled-password --gecos kodi kodi
    $ sudo passwd -d kodi

That creates a user that signs in without a password, but it will sign into the systems default desktop session (e.g. GNOME). Now we need to reconfigure the account to log into “frame-it”:

    $ sudo sh -c "cat > /var/lib/AccountsService/users/kodi"
    [User]
    XSession=frame-it
    Icon=/home/kodi/.face
    SystemAccount=false
    ^D

    $ sudo sh -c "chmod u=rw,g=,o= /var/lib/AccountsService/users/kodi"

And that’s it!

We now have a new “kodi” user on the computer, and when that user is selected on the logon screen you, or your family, are taken straight to Kodi. And when you exit Kodi you are logged out.

## Testing with alternative shells

When developing Ubuntu Frame it is often convenient to use alternative installations, for example (using snapd's experimental "parallel install":
```plain
snap install --beta ubuntu-frame_beta
```
To support this it is possible to configure Ubuntu Frame to use an alternative shell using a snap configuration option:
```plain
snap set frame-it shell=ubuntu-frame_beta
```
