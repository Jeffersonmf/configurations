# Base System to XOrg

***DO NOT USE THESE NOTES BLINDLY.***  
***SOME CONFIGURANTIONS ARE PERSONAL AND PROBABLY OUTDATED.***

#### Add new user

It is good practice to use a normal user and elevate to root only when necessary.  
There are a few ways to do it, but the most common ones are the following.

<pre>
useradd -m -G wheel -s /bin/bash filipe 
</pre>

This command will also automatically create a group called `filipe` with the same GID as the UID of the user `filipe`
and makes this the default group for `filipe` on login. Making each user have their own group (with group name same as 
user name and GID same as UID) is the preferred way to add users. 

You could also make the default group something else, e.g. users. 

<pre>
useradd -m -g users -G wheel -s /bin/bash filipe
</pre>

However, using a single default group (users in the example above) is not recommended for multi-user systems. 
The reason is that typically, the method for facilitating shared write access for specific groups of users is 
setting user umask value to 002, which means that the default group will by default always have write access 
to any file you create. 

Change your finger information and password  
<pre>
chfn filipe  
passwd filipe
</pre>

#### Give sudo permissons to new user

Trick visudo to open with nano:
<pre>
EDITOR=nano visudo
</pre>

Add user to the section “User privilege specification”:
<pre>
Filipe ALL=(ALL) ALL
</pre>

Logout from root and enter the new account:
<pre>
logout
</pre>

Once you login with the new user, you can checkout the groups information with the command:
<pre>
id
</pre>

#### Activate multilib repo

Remove comments from [multilib]:
<pre>
nano /etc/pacman.conf  
pacman -Syy
</pre>

#### Install Packer

First install the components needed to use `wget` and `git`, then get the tarball and extract.  
Finally, build the package with makepkg and install it using pacman.

<pre>
pacman -S wget git jshon expac  
wget https://aur.archlinux.org/packages/pa/packer/packer.tar.gz  
tar zxvf packer.tar.gz  
cd packer && makepkg  
pacman -U packer (press tab)
</pre>

Once installed, cleanup:

<pre>
rm -R packer  
rm packer.tar.gz
</pre>

<sub><sup>
References:
http://www.cyberciti.biz/faq/unpack-tgz-linux-command-line/
</sup></sub>

#### ALSA

ALSA is a set of build-in GNU/Linux kernel modules. Therefore, manual installation is not necessary. 
But the channels are muted by default, so we install `alsa-utils` that contains alsamixer to unmute the audio.
The `alsa-utils` package also comes with systemd unit configuration files alsa-restore.service and alsa-store.service by default. These are automatically installed and activated during installation. Therefore, there is no further action needed. Though, you can check their status using systemctl

<pre>
pacman -S alsa-utils
run alsamixer
Press H to unmute. Press F1 for help.
run speaker-test
</pre>

If you need high quality resampling install the `alsa-plugins` package to enable upmixing/downmixing and other advanced features. When software mixing is enabled, ALSA is forced to resample everything to the same frequency (48 kHz by default when supported). To do so, install the plugins for high quality resampling:
<pre>
pacman -S alsa-plugins
</pre>

To test stereo audio (for more channels change the -c parameter) use:
<pre>
speaker-test -c 2
</pre>

#### PulseAudio

Now we need a sound server, it serves as a proxy to sound applications using existing kernel sound components like ALSA.
Since ALSA is included in Arch Linux by default, the most common deployment scenarios include PulseAudio with ALSA. 

Now lets install the pulseaudio server:
<pre>
pacman -S pulseaudio 
</pre>

Install the `pulseaudio-alsa` package. It contains the necessary /etc/asound.conf for configuring ALSA to use PulseAudio.
Also install `lib32-libpulse` and `lib32-alsa-plugins`if you run a x86_64 system and want to have sound for 32-bit multilib programs like Wine, Skype and Steam. 
<pre>
pacman -S pulseaudio-alsa lib32-libpulse lib32-alsa-plugins
</pre>

<sub><sup>
References:  
https://wiki.archlinux.org/index.php/Advanced_Linux_Sound_Architecture#High_quality_resampling  
https://wiki.archlinux.org/index.php/PulseAudio#Back-end_configuration
</sup></sub>

#### PulseAudio Audiophile

By default, PulseAudio (PA) uses very conservative settings. This will work fine for most audio media as you will most likely have 44,100Hz sample rate files. However, if you have higher sample rate recordings it is recommended that you increase the sample rate that PA uses.

<pre>
nano /etc/pulse/daemon.conf
add: default-sample-format = s32le 
add: default-sample-rate = 96000 
add: resample-method = speex-float-5 
</pre>

For the most geniune resampling at the cost of high CPU usage (even on 2011 CPUs) you can add: 

<pre>
nano /etc/pulse/daemon.conf
resample-method = src-sinc-best-quality 
</pre>

If you are having problems with the channels set by pulseaudio, you can set them manually by adding:

<pre>
nano /etc/pulse/daemon.conf
default-sample-channels = 3
default-channel-map = front-left,front-right,lfe
</pre>

<sub><sup>
References: 
http://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Audiophile/
</sup></sub>

##### XOrg

Now we need to install XOrg, a display server for the X Window System. We will also install `xorg-xinit` that will provide  xinit and startx. For more configurations see `~/.xinitrc` file is a shell script read by xinit and by its front-end startx. The xinit program starts the X Window System server and works as first client program on systems that are not using a display manager. 

<pre>
pacman -S xorg-server xorg-server-utils xorg-xinit mesa
</pre>

<sub><sup>
References:  
https://wiki.archlinux.org/index.php/xorg#Installation  
https://wiki.archlinux.org/index.php/Xinitrc
</sup></sub>

##### Graphic drivers

First, identify your card: 
<pre>
lspci | grep VGA
</pre>

If you need open-souce drivers you can search the package database for a complete list of open-source video drivers: 
<pre>
pacman -Ss xf86-video | less
</pre>

Desktop:

The graphics card is too old, and there is no support for it, so I use the open-source drivers.

<pre>
pacman -S mesa-dri xf86-video-vesa xf86-video-ati
</pre>

<sub><sup>
References: 
https://wiki.archlinux.org/index.php/ATI#Installation
https://wiki.archlinux.org/index.php/Xorg#Driver_installation
</sup></sub>

Notebook:

This notebook has two graphics cards, intel and nvidia.
In order to manage them we use `bumblebee` that is Optimus for GNU/Linux. We also install bbswitch, has the goal of power management to turn off the NVIDIA card when not used by Bumblebee. It will be detected automatically when the Bumblebee daemon starts. So, no additional configuration is necessary. 

<pre>
pacman -S mesa-dri xf86-video-intel nvidia bumblebee bbswitch
</pre>

In order to use Bumblebee, it is necessary to add your regular user to the bumblebee group and also enable bumblebeed.service

<pre>
gpasswd -a filipe bumblebee  
systemctl enable bumblebeed.service 
</pre>

<sub><sup>
References:  
https://wiki.archlinux.org/index.php/Bumblebee#Installing_Bumblebee_with_Intel.2FNVIDIA  
https://wiki.archlinux.org/index.php/Bumblebee#Start_Bumblebee
https://wiki.archlinux.org/index.php/Hybrid_graphics#ATI_Dynamic_Switchable_Graphics
</sup></sub>

##### Input drivers

Desktop and Notebook:
<pre>
pacman -S xf86-input-mouse xf86-input-keyboard
</pre>

Notebook:
<pre>
pacman -S xf86-input-synaptics 
</pre>

##### Test default enviroment

Make sure XOrg is working before we install a desktop enviroment.

<pre>
pacman -S xorg-twm xorg-xclock xterm  
startx
</pre>

<sub><sup>
References:
https://wiki.archlinux.org/index.php/Xorg#Manually
</sup></sub>

###### Install KDE5

KDE4 has been removed from the repos and is no longer suported. KDE5 is now in a stable state so this is prefered.
The past problems with the tray bar have been fixed and the applications are working as they should.

If you have KDE4 is installed, and since KDE4 and KDE5 cannot run together we need to remove it.  
Also, disable KDM from staring, KDE5 recommends SDDM instead and remove KDE4 specific packages

<pre>
pacman -Rnc kdebase-workspace 
systemctl disable kdm
sudo pacman -Rns oxygen-gtk2 oxygen-gtk3-git kde-gtk-config-kde4
</pre>

Install KDE5, system language, display manager SDDM and the oficial theme breeze:

<pre>
pacman -S plasma-meta kde-l10n-pt sddm sddm-kcm
pacman -S breeze-kde4 gtk-theme-orion
</pre>

Activate SDDM and create the default config file:

<pre>
systemctl enable sddm
sddm --example-config > /etc/sddm.conf
</pre>

Finally, apply the newer `breeze` theme to the display manager:

<pre>
sudo nano /etc/sddm.conf
</pre>

And change the theme section according to this:

<pre>
[Theme]
Current=breeze
CursorTheme=breeze_cursors
FacesDir=/usr/share/sddm/faces
ThemeDir=/usr/share/sddm/themes
</pre>

You can do it with GUI because we already installed the `sddm-kcm` package, but I recomend editing the file manually for the first use, I had some problems starting with the default theme in the past. After reboot, to use the GUI go to `Setting > Startup and Shutdown > Login Screen. (2nd tab)` and choose the `Breeze` theme.

Since KDE5 uses a new tray system, we need to change QSystemTrayIcon to StatusNotifierItems the package that does this is `sni-qt` and we need to install libindicator packages as well.

<pre>
packer -S gtk-sharp-2 libdbusmenu-gtk2 libdbusmenu-gtk3 libindicator-gtk2 libindicator-gtk3
packer -S libappindicator-gtk2
packer -S libappindicator-gtk3
packer -S sni-qt lib32-sni-qt
pacman -S kde-gtk-config
</pre>

Even if you are upgrading, configurations will be lost so here are some of my settings:

<pre>
localectl set-keymap pt-latin9
localectl set-x11-keymap pt 
Add shortcut Windows+L
Fix firefox and dolphin and terminal on bar
Hide Unmounnted Drivers
Change single to double click, in system settings
Change User Avatar 
Disable program preview 
Disable default multimédia player (bar options)
</pre>

For the photon backend, use VLC because it has the best upstream support.  
But multiple backends can be installed at once and prioritized at System Settings > Multimedia > Backend, so I install both.

<pre>
pacman -S phonon-qt5 phonon-qt5-gstreamer phonon-qt5-vlc
</pre>

<sub><sup>
References:  
https://wiki.archlinux.org/index.php/KDE
https://wiki.archlinux.org/index.php/Plasma  
https://wiki.archlinux.org/index.php/SDDM  
http://www.linuxveda.com/2015/02/27/how-to-install-kdes-plasma-5-on-arch-linux/
</sup></sub>