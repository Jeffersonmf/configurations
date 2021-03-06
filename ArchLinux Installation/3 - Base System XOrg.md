# Base System XOrg

***DO NOT USE THESE NOTES BLINDLY.***  
***SOME CONFIGURATIONS ARE PERSONAL AND PROBABLY OUTDATED.***

This is a follow-up of the [Base System Configuration](https://github.com/Tenza/configurations/blob/master/ArchLinux%20Installation/2%20-%20Base%20System%20Configuration.md) guide.  
The configurations and installations set in here suppose that ArchLinux is properly installed and configured on the system.

#### Add new user

It is a good practice to use a normal user and elevate to root only when necessary. This should be done for a few reasons.

1. Keep ownership of personal files separate from the root user.  
2. Some applications prevent themselves from running as root.  
3. Prevent applications to have full access to the computer.  
4. Protect the system against user mistakes.  

<pre>
useradd -m -G wheel -s /bin/bash filipe 
</pre>

| Switch | Description | 
| --- | --- | 
| -m | Creates the user home directory as `/home/filipe`. Within the home directory, a non-root user can read, write and execute. | 
| -G | Introduces a list of supplementary groups which the user will also be a member of. The default is for the user to belong only to the initial group, defined by the `-g` switch. In this case, the user is going to be an administrator of the system, so he should be a member of the `wheel` group. | 
| -s | Defines the path and file name of the user's default login shell. After the boot process is complete, the default login shell is the one specified here. The bash shell is used in here, but others are available.  |
| -g | Defines the user's initial group. If omitted, the behavior of `useradd` will depend on the `USERGROUPS_ENAB` variable contained in `/etc/login.defs`. The default behavior is to create a group with the same name as the username, and a GID equal to UID. Making each user have their own group is the preferred way to add users. If specified, the group name must already exist. For example `useradd -m -g users -G wheel -s /bin/bash filipe` makes the default group `users`. However, using a single default group is not recommended for multi-user systems. Because typically the method for facilitating shared write access for specific groups of users is setting user umask value to 002, which means that the default group will by default always have write access to any file created by the used. |

<sub><sup>
References:  
https://wiki.archlinux.org/index.php/Users_and_groups#User_management  
https://wiki.archlinux.org/index.php/general_recommendations#Users_and_groups
</sup></sub>

##### (Optional) (Recommended) Change user details

The local user information is stored as plain-text in `/etc/passwd` file. Each of its lines represents a user account, and has seven fields delimited by colons. The `GECOS` field is optional, and is used for informational purposes only. The `chfn` command is used to manage this finger information.

Although it is not required to protect the newly user with a password, it is recommend to do so.

<pre>
chfn filipe  
passwd filipe
</pre>

#### Privilege escalation

There are two main ways to escalate the privileges of the newly created user. The `su` (substitute user) command allows the user to assume the identity of another user on the system (usually root) from an existing login. Whereas the `sudo` (substitute user do) command grants temporary privilege escalation for a specific command. 

From a security perspective, it is arguably better to set up and use `sudo` instead of `su`. The `sudo` system will prompt for the users own password (or no password at all) rather than the target user (the user account you are attempting to use). This way there is no need to share passwords between users, and if it is ever needed to stop a user from having root access (or access to any other account), there is no need to change the root password. It is only needed to revoke that user's `sudo` access.

##### (1) (Recommended) Sudo

To grant `sudo` access to the new user, the `sudo` configuration file `/etc/sudoers` has to be edited with the `visudo` command. The `visudo` command locks the sudoers file, saves edits to a temporary file, and checks that file's grammar before copying it to `/etc/sudoers`. Any error in this file can render the `sudo` command unusable, so always edit it with `visudo` to prevent errors. Also, the `visudo` opens by default with the `vi` editor, but it also honors use of the `VISUAL` and `EDITOR` environment variables. So in order to open `visudo` with `nano`, set the `EDITOR` before calling `visudo`.

<pre>
EDITOR=nano visudo
</pre>

To allow the user to gain full root privileges when `sudo` precedes a command, add user to the section `User privilege specification`.

<pre>
filipe ALL=(ALL) ALL
</pre>

Alternatively, allow the `wheel` group to gain full root privileges. Keep in mind that the most customized option should go at the end of the file, as the later lines overrides the previous ones. In particular, the previous command where the `filipe` user was set, should be after the `%wheel` line if the user is in this group.

<pre>
%wheel ALL=(ALL) ALL
</pre>

<sub><sup>
References:
https://wiki.archlinux.org/index.php/sudo
</sup></sub>

##### (2) (Not Recommended) Su

Sometimes can be advantageous for a system administrator to use the shell account of an ordinary user rather than its own. In particular, occasionally the most efficient way to solve a user's problem is to log into that user's account in order to reproduce or debug the problem.

However, in many situations it is not desirable, or it can even be dangerous, for the root user to be operating from an ordinary user's shell account and with that account's environmental variables rather than from its own. While inadvertently using an ordinary user's shell account, root could install a program or make other changes to the system that would not have the same result as if they were made while using the root account. For instance, a program could be installed that could give the ordinary user power to accidentally damage the system or gain unauthorized access to certain data.

Thus, it is advisable that administrative users, as well as any other users that are authorized to use `su`, acquire the habit of always following the su command with a space and then a hyphen, the hyphen has two effects.

1. Switches from the current directory to the home directory of the new user (e.g., to /root in the case of the root user) by logging in as that user.  
2. Changes the environmental variables to those of the new user as dictated by their ~/.bashrc. That is, if the first argument to su is a hyphen, the current directory and environment will be changed to what would be expected if the new user had actually logged on to a new session (rather than just taking over an existing session).

<pre>
su - root
</pre>

<sub><sup>
References:
https://wiki.archlinux.org/index.php/Su
</sup></sub>

#### Login on the new user account

Once the new account has been properly created and configured, `logout` from the root account and check the new user/groups information with the `id` command to confirm that everything is as it should be. From this point forward all commands will be done using the new account, using `sudo`.

<pre>
logout
(login)
id
</pre>

#### AUR Helper

There are a few helpers scripts to automatize the task of installing software from the AUR repository. I've personally tried `packer`, `yaourt` and `pacaur`, they are all pretty decent and this basically comes down to personal choice. [Here is a table](https://wiki.archlinux.org/index.php/AUR_helpers#Comparison_table) that can help with that decision. At the moment I'm using `pacaur` and the following instructions are for this helper, but they should be similar to any other.

Without any helper script, the manual building process to download and install software from the AUR has to be used because these helper scripts are in the AUR itself. Before anything, make sure that the package `base-devel` is installed, it is required in order to manually build the packages. Then, start of by downloading the helper script, and the dependencies that are also in the AUR, unpack the tarballs, build the packages and finally install the dependencies and the helper script.

<pre>
mkdir temp && cd temp
curl - O https://aur.archlinux.org/cgit/aur.git/snapshot/cower.tar.gz
curl - O https://aur.archlinux.org/cgit/aur.git/snapshot/pacaur.tar.gz
tar zxvf cower.tar.gz
tar zxvf pacaur.tar.gz
cd cower && makepkg -sri
cd pacaur && makepkg -sri
rm -R temp
</pre>

| Switch | Description | 
| --- | --- | 
| tar -z | Filter the archive through gzip | 
| tar -x | Extract files from an archive | 
| tar -v | Verbosely list files processed |
| tar -f | The following argument is a f̱ilename |

| Switch | Description | 
| --- | --- | 
| makepkg -s | Installs missing dependencies using pacman | 
| makepkg -s | Upon successful build, removes any dependencies installed by `makepkg` | 
| makepkg -i | Install or upgrade the package after a successful build using pacman |

> When I tried to install `cower` it failed in the verification of the package signature. In order to solve the problem I added the key from the developer of `cower` with `gpg --recv-keys --keyserver hkp://pgp.mit.edu 1EB2638FF56C0C53`.

<sub><sup>
References:  
https://wiki.archlinux.org/index.php/AUR_helpers  
http://www.cyberciti.biz/faq/unpack-tgz-linux-command-line/
</sup></sub>

#### Audio Drivers

##### ALSA

ALSA is a set of build-in GNU/Linux kernel modules. Therefore, manual installation is not necessary. Channels are muted by default, and in order to unmute the audio `alsamixer`, a application within `alsa-utils` can be installed. The `alsa-utils` package also comes with systemd unit configuration files `alsa-restore.service` and `alsa-state.service` by default. These are automatically installed and activated during installation, no further action needed. They can be checked with `systemctl status alsa-restore.service` and `systemctl status alsa-state.service`.

<pre>
pacman -S alsa-utils
alsamixer
Press F1 for help.
reboot
speaker-test -c 2
</pre>

When software mixing is enabled, ALSA is forced to resample everything to the same frequency (48 kHz by default when supported). By default, it will try to use the speexrate converter to do so, and fallback to low-quality linear interpolation if it is not available.
So, if for some reason the audio quality is poor, install the `alsa-plugins` package to enable upmixing/downmixing and other advanced features.

<pre>
pacman -S alsa-plugins
speaker-test -c 2
</pre>

| Switch | Description | 
| --- | --- | 
| -c X | Number of channels |

<sub><sup>
References: 
https://wiki.archlinux.org/index.php/Advanced_Linux_Sound_Architecture
</sup></sub>

##### PulseAudio

PulseAudio is a sound server, it serves as a proxy to sound applications using existing kernel sound components like ALSA.
Since ALSA is included in ArchLinux by default, the most common deployment scenarios include PulseAudio with ALSA. Some confusion can be made between ALSA and PulseAudio. ALSA includes both Linux kernel component with sound card drivers, and a userspace component, `libalsa`. PulseAudio builds only on the kernel component, but offers compatibility with `libalsa` through `pulseaudio-alsa`.

<pre>
pacman -S pulseaudio 
</pre>

Also install the `pulseaudio-alsa` package, it contains the necessary `/etc/asound.conf` for configuring ALSA to use PulseAudio.
The packages `lib32-libpulse` and `lib32-alsa-plugins` are also needed in x86_64 systems that want to have sound for 32-bit multilib programs like Wine, Skype or Steam. 

<pre>
pacman -S pulseaudio-alsa lib32-libpulse lib32-alsa-plugins
</pre>

By default, PulseAudio is configured to automatically detect all sound cards and manage them. It takes control of all detected ALSA devices and redirect all audio streams to itself, making the PulseAudio daemon the central configuration point. The daemon should work mostly out of the box.

<sub><sup>
References:  
https://wiki.archlinux.org/index.php/Advanced_Linux_Sound_Architecture  
https://wiki.archlinux.org/index.php/PulseAudio#Back-end_configuration
</sup></sub>

#### Graphics Drivers

When selecting graphic drivers, the choice comes down to proprietary vs open-source drivers. 
For ATI, there is `catalyst` with proprietary drivers and `xf86-video-ati` for open-source drivers. 
For NVIDIA, there is `nvidia` with proprietary drivers and `xf86-video-nouveau` for open-source drivers.
For Generic drivers, there is `xf86-video-vesa`.

Start by knowing the hardware in order to choose the driver appropriate driver.

<pre>
lspci | grep VGA
</pre>

> All of the proprietary drivers are generally bad on legacy hardware. On newer hardware they might be better, but not without problems. ATI Catalyst proprietary drivers have a very poor support, quality and speed of development. Catalyst drivers have also been dropped from the oficial Arch repository. NVIDIA proprietary drivers at the moment suffer possible screen-tearing issues, and when it comes to hybrid systems, they do not offer support for dynamic switching like they do on Windows.

##### Desktop

The desktop has an old graphics card (ATI Radeon HD 4000 series), there are proprietary legacy drivers (`catalyst-total-hd234k` in AUR) and open-source drivers available. 

When choosing what to use, I would choose Catalyst drivers for newer graphics cards because they perform better in both 2D and 3D rendering, also having better power management when compared with the open source drivers. But for older cards, I prefer the open-source drivers, they work generally very well.

<pre>
pacman -S mesa mesa-libgl lib32-mesa-libgl xf86-video-ati
</pre>

<sub><sup>
References:  
https://wiki.archlinux.org/index.php/ATI#Installation  
https://wiki.archlinux.org/index.php/Xorg#Driver_installation
</sup></sub>

##### Notebook

The notebook has a hybrid system with two graphics cards, Intel and NVIDIA. The software that manages the GPU's is called NVIDIA Optimus, in Windows it works by dynamically switching between the GPU's, based on a whitelist provided by the driver of NVIDIA. In Linux, there is no actual official NVIDIA support for the Optimus technology, or at least not completely. 

This subject is complex, but basically there are two software’s that attempt to deal with the lack of official support, one is called [PRIME](https://wiki.archlinux.org/index.php/PRIME), the other one is called [Bumblebee](https://wiki.archlinux.org/index.php/Bumblebee). I will leave here a few bullet points to illustrate the scenario.

1. PRIME requires a `xrandr` command to switch GPU's and than a logout to effectively change the GPU.
2. Bumblebee requires a `optirun` or `primusrun` command to execute a specific application with the dedicated GPU.
3. PRIME integrates better with open-source drivers that have poor performance compared with the proprietary drivers.
4. Bumblebee integrates better with proprietary drivers, but is reported to have poor performance compared with PRIME.
5. Bumblebee can be integrated with [Primus](https://github.com/amonakov/primus) (not PRIME) for better performance.
6. Bumblebee can be integrated with [bbswitch](https://wiki.archlinux.org/index.php/Bumblebee#Power_management) to have better power management.
8. There are [scripts](https://devtalk.nvidia.com/default/topic/705993/easy-switch-between-bumblebee-and-nvidia-prime/) for Ubuntu to integrate both solutions.
9. There is also [nvidia-xrun](https://aur.archlinux.org/packages/nvidia-xrun/) that tries to ditch Bumblebee to gain a bit of performance.
10. Read more [here](http://www.pcworld.com/article/2944964/how-to-use-nvidia-optimus-to-switch-active-gpus-and-save-power-on-linux-laptops.html), [here](http://www.thelinuxrain.com/articles/the-state-of-nvidia-optimus-on-linux), [here](http://www.webupd8.org/2012/11/primus-better-performance-and-less.html) and [here](https://support.steampowered.com/kb_article.php?ref=6316-GJKC-7437) 

After consideration, I want the system to run with Bumblebee, bbswitch and Primus. 

<pre>
pacman -S mesa mesa-libgl lib32-mesa-libgl xf86-video-intel 
pacman -S nvidia 
pacman -S bumblebee lib32-virtualgl lib32-nvidia-utils 
pacman -S bbswitch primus lib32-primus
pacman -S mesa-demos
</pre>

1. Dont install `nvidia-libgl` or `lib32-nvidia-libgl`, there are compatibility problems with bumblebee, instead use `mesa-libgl` and `lib32-mesa-libgl`. 
2. Another interesting package to notice is `vulkan-intel`, unfortunately my notebook has a Intel Core i7 3612QM, with Intel HD Graphics 4000 so Vulkan is not yet supported.
3. XOrg-server will probably be installed, given the dependencies of the packages. If so, it will ask to choose a input driver, select the newer `libinput` driver. 

In order to use Bumblebee, it is necessary to add your regular user to the bumblebee group and also enable bumblebeed.service.

<pre>
gpasswd -a filipe bumblebee  
systemctl enable bumblebeed.service 
reboot
systemctl status bumblebeed.service 
</pre>

This installation will only be properly tested once XOrg is installed and a screen initialized. The following commands can be used later to verify the Bumblebee and Primus installation.

<pre>
optirun glxgears -info
optirun glxspheres64
optirun glxspheres32
optirun -b primus glxgears
optirun -b primus glxspheres32
optirun -b primus glxspheres64
vblank_mode=0 primusrun glxgears
vblank_mode=0 primusrun glxspheres32
vblank_mode=0 primusrun glxspheres64
</pre>

<sub><sup>
References:  
https://wiki.archlinux.org/index.php/Bumblebee  
https://wiki.archlinux.org/index.php/NVIDIA  
https://wiki.archlinux.org/index.php/Intel_graphics  
https://wiki.archlinux.org/index.php/NVIDIA_Optimus  
</sup></sub>

##### Input drivers

The `libinput` a newer driver that can now replace `xf86-input-*` drivers. The `xf86-input-libinput` package, is a thin wrapper around `libinput` that allows it to be used for input devices in X. The default configuration file is installed in `/usr/share/X11/xorg.conf.d/60-libinput.conf` and no extra configuration is necessary for it to autodetect keyboards, touchpads, trackpointers and supported touchscreens. Install the package, and then call `libinput-list-devices`, it will output the devices on the system and their respective features supported by `libinput`. 

<pre>
pacman -S xf86-input-libinput
reboot
libinput-list-devices
</pre>

<sub><sup>
References: 
https://wiki.archlinux.org/index.php/Libinput
</sup></sub>

##### XOrg

Xorg is a display server for the X Window System. In order to start X, `xorg-xinit` will also be installed. The `~/.xinitrc` file is a shell script read by `xinit` and by its front-end `startx`. It is mainly used to execute desktop environments, window managers and other programs when starting the X server.

<pre>
pacman -S xorg-server xorg-server-utils xorg-xinit
</pre>

To make sure X is working, the following packages should be installed and X initialized. Install the three packages, before initializing X, otherwise some errors will be displayed. Once X is started, and the windows manually closed, the console should print `X Server has terminated successfully (0)`.

<pre>
pacman -S xorg-twm xorg-xclock xterm  
startx
</pre>

<sub><sup>
References:  
https://wiki.archlinux.org/index.php/xorg  
https://wiki.archlinux.org/index.php/Xinit  
https://wiki.archlinux.org/index.php/Xorg#Manually
</sup></sub>
