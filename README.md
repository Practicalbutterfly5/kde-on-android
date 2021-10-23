# kde-on-android
Install kde-plasma-desktop and ubuntu on android (chroot)

**REQUIRES ROOTED DEVICE**

**Requires at least 5 GB of storage, approx <1.5 GB of network data**

![photo_2021-10-23_07-35-21](https://user-images.githubusercontent.com/75225829/138538986-b659b21f-9e85-4f57-abc5-c6503ec544bb.jpg)

_Backup and rename your /sdcard/linux.img if you have one_

### Initial Setup

1. Install [Linux Deploy](https://github.com/meefik/linuxdeploy/releases/tag/2.6.0) and [Termux](https://f-droid.org/en/packages/com.termux/)
2. Open Linux Deploy.
3. Goto Linux Deploy Settings and change the following settings
   1. Distribution : `Ubuntu`
   2. User name : `username`
   3. User Password : `password`
   4. Init > Enable (tick checkbox)
   5. Init > Init Settings > Init user : `username`
   6. Mounts > Enable (tick checkbox)
   7. Mounts > Mount points > `+` icon on top right >  Source : `/sdcard` Target : `/sdcard`
   8. SSH > Enable (tick checkbox)
4. Click `⋮` icon in Linux Deploy > Install . It needs a stable Internet connection, otherwise it might fail at some step. Proceed to next step only after installation is complete.
5. Press `START` button in Linux Deploy.
6. Open Termux and type following commands. 
   1. `apt update`
   2. `apt install -y openssh e2fsprogs pulseaudio`
   3. `echo "pkill pulseaudio;pulseaudio --start --exit-idle-time=-1 && pacmd load-module module-native-protocol-tcp auth-anonymous=1" >  $PREFIX/etc/profile.d/pulseaudio.sh`
   4. `chmod +x $PREFIX/etc/profile.d/pulseaudio.sh`
   4. `$PREFIX/etc/profile.d/pulseaudio.sh`
   5. `termux-setup-storage` (Allow termux to access storage)
   6. `e2fsck -f /sdcard/linux.img` (Press enter on all prompts. Recommended to stop linux deploy before this step and start after resizing!)
   7. `resize2fs /sdcard/linux.img 5G` (This step will change size of linux.img to 5G. You can change it anytime required.)
   4. `ssh-keygen` (Press Enter on all prompts. If it asks for overwrite type 'y' and press Enter)
   5. `ssh-copy-id -i ~/.ssh/id_rsa username@localhost` (Type "yes" to accept fingerprint. On prompt for password type password you set in Linux Deploy)
   8. `ssh username@localhost`
7. Now you should be logged into ubuntu. Verify it by running `whoami` or `sudo apt install -y neofetch && neofetch` and it should display your username.
8. Run the following commands in ubuntu.
   1. `sudo passwd` (Set root user password. Enter your password 2 times. This is root user password, the password set in Linux Deploy was username user password)
   2. `sudo su -c 'grep -qxF "deb http://ports.ubuntu.com/ bionic-updates main restricted" /etc/apt/sources.list || echo "deb http://ports.ubuntu.com/ bionic-updates main restricted" >> /etc/apt/sources.list' `
   3. `sudo apt update`
   4. `sudo DEBIAN_FRONTEND='noninteractive' apt-get -y -o Dpkg::Options::='--force-confdef' -o Dpkg::Options::='--force-confold' dist-upgrade`
   6. `sudo apt install -y ubuntu-release-upgrader-core`
   7. `sudo do-release-upgrade -f DistUpgradeViewNonInteractive` (Press 'Enter' key on any prompt. Upgrades from ubuntu bionic 18.04 to focal 20.04. Takes long time)
   8. `sudo sed -i 's/Prompt=lts/Prompt=normal/g' /etc/update-manager/release-upgrades`
   9. `sudo do-release-upgrade -f DistUpgradeViewNonInteractive` (Press 'Enter' key on any prompt. Upgrades from ubuntu focal 20.04 to hirsute 21.04. Takes long time)
   10. `sudo do-release-upgrade -f DistUpgradeViewNonInteractive` (Press 'Enter' key on any prompt. Upgrades from ubuntu hirsute 21.04 to Impish Indri 21.10. Takes long time)
   11. `sudo apt install -y kde-plasma-desktop` (Press 'Enter' key on any prompt. Installs plasma kde desktop. apprx 600 Mb. download)
   12. `sudo apt remove -y tightvncserver && sudo apt install -y --install-suggests tigervnc-standalone-server dbus-x11 net-tools`
   13. `sudo service dbus start`
   14. `sudo bash -c 'echo -e "#!/bin/bash \n [[ -z "\$PULSE_SERVER" ]] && export PULSE_SERVER=127.0.0.1 \n sudo rm -rf /etc/profile.d/pulse.sh \n vncserver -kill :1 \n sudo rm /tmp/.X11-unix/X1 /tmp/.X1-lock \n dbus-launch tigervncserver -localhost no -geometry 1366x768 -xstartup /usr/bin/startplasma-x11 -listen tcp :1" > /etc/rc.local/init'`
   15. `sudo bash -c 'echo -e "#!/bin/bash \n [[ ! -z "\$XDG_CURRENT_DESKTOP" ]] &&echo "Execute this command from termux after logging in to ubuntu via ssh" && exit \n export PULSE_SERVER=\$1 \n vncserver -kill :1 \n /etc/rc.local/init">/bin/audioserver '`
   15. `sudo chmod +x /etc/rc.local/init /bin/audioserver`
   16. `vncpasswd` (Here set your VNC password to be used in vnc viewer. Enter `n` when it asks for "Would you like to enter a view-only password".)
   17. `sudo mv /etc/apt/apt.conf.d/20packagekit /etc/apt/apt.conf.d/20packagekit.disabled` (Fix the "Error connecting: Could not connect: Connection refused" bug with apt)


**Now simply restart linux container. Linux Deploy>STOP. Linux Deploy>START.**


## View on android
1. Start container from Linux Deploy>START.
1. Install the [VNC viewer](https://play.google.com/store/apps/details?id=com.realvnc.viewer.android) app.
2. Add a new computer and use `locahost:5901`as address.
3. Enter your vnc password that you had set with vncpasswd.
4. Enjoy. :-)

## View on PC
1. Start container from Linux Deploy>START.
1. Install Remmina
   - On Windows: Use any VNC viewer available for windows. For example TigerVNC or realVNC. I don't use windows so I don't know which is better.
   - On Linux PC: `sudo apt install -y --install-suggests remmina remmina-plugin-vnc`
2. Connect your android and PC to a common network. For example to a common wifi, or start mobile hotspot and connect PC to it.
3. Type `ifconfig` in Termux. You will see a screen similiar to below image. Get the `inet` of `wlan0` or `wlan1` whichever represents your common network. This is the address you will use in VNC viewer. Lets call it `internal_ip`
![photo_2021-10-23_13-25-24](https://user-images.githubusercontent.com/75225829/138548343-fcc3ba21-a366-47db-a6d0-5fed74ac013b.jpg)
4. In address of VNC viewer type internal_ip:5901. Press Enter key.
![image](https://user-images.githubusercontent.com/75225829/138548480-1f0c9592-1c6a-4df2-9ec2-1965d15383d0.png)
5. If the resolution is incorrect and video seems to be cut, refer to **Changing Display Resolution** in customisation section below.
5. Enjoy :-)
   
----------------   
   
### Customisation
This section includes some customisations to make life easier. ;-). These steps are completely voluntary, and my personal preference.
Login to ubuntu via termux or vnc viewer (and open konsole). Also restart linux container after customisation to take effect.
1. Install firefox : `sudo apt install -y firefox`
2. Install ZSH as default shell : `sudo apt install -y zsh && sudo chsh $(which zsh)`
3. Install OhMyZsh : `sudo apt install -y curl git && sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
4. Install zsh plugins : [zsh-auto-suggesitons](https://github.com/zsh-users/zsh-autosuggestions), [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) and command-not-found . Open ~/.zshrc and change the plugins line to `plugins=(git command-not-found zsh-autosuggesitions zsh-syntax-highlighting)` . Note that zsh-syntax-highlighting should be the last plugin.
5. Installing [kdeconnect - The ultimate android integration with linux](https://linuxconfig.org/connect-your-android-phone-to-linux-with-kde-connect)
6. Resizing linux.img, in case your linux runs out of storage.
   1. Make sure linux container is stopped. Linux Deploy>STOP.
   2. In termux type 
      - `e2fsck -f /sdcard/linux.img` (Press enter on all prompts.)
      - `resize2fs /sdcard/linux.img 10G` (Replace 10G with the amount of storage u want linux to have)
7. Changing Display Resolution : If you are using vnc viewer on PC then you might have to change resolution. First get the resolution of your PC display. You can google it, or in Linux pc use `xdpyinfo | grep 'dimensions:'` to find resolution.
   - Run the following command in ubuntu to change resolution. Replace 1366x768 with your resolution 
     `sudo bash -c 'echo -e "#!/bin/bash \n [[ -z "\$PULSE_SERVER" ]] && export PULSE_SERVER=127.0.0.1 \n sudo rm -rf /etc/profile.d/pulse.sh \n vncserver -kill :1 \n sudo rm /tmp/.X11-unix/X1 /tmp/.X1-lock \n dbus-launch tigervncserver -localhost no -geometry 1366x768 -xstartup /usr/bin/startplasma-x11 -listen tcp :1" > /etc/rc.local/init'`
8. Setting up audio output :
    - **Audio via android** : Make sure that termux is running in background, and audio should work out-of-the-box.
    - **Audio via Linux pc** : Currently I only know how to do this on linux pc with pulseaudio installed. Open terminal your linux pc and type :
        1. `pacmd load-module module-native-protocol-tcp auth-anonymous=1` (You can also add it to your .profile file, so it will run on startup from next boot.)
        2. `ifconfig` (and get the `inet` address. Lets call it internal_ip)
             - Login to ubuntu via termux and type `audioserver internal_ip` (Replace internal_ip with the inet address you got from linux pc. After this command vncserver will restart and you have to reconnect to vnc with audio output to your linux pc ip address.)
9. Installing kde-full : KDE just doesn't end here. KDE has its own, tons of personalised and beautiful ui applications. Install full set of kde applications by running `sudo apt install -y kde-full` (approx 200Mb).
