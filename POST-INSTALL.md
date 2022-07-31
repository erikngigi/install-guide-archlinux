## POST INSTALLATION

In post installation we will be using a lot of `sudo`. I'm not responsible if you broke your newly installed system.

### Connect to the internet

We will be downloading stuff so we need an internet connection! So set-up your connection first!

### Check for Updates

It's recommended to check for updates first before installing anything so:

```
pacman -Syu
```

### Display Server and Protocol

We need to install a display server, a protocol or both. Normally, your desktop environment or window manager of choice will automatically install these as a dependency. But for this guide's sake we will install `X` server:

```
pacman -S xorg-server xorg-xrdb xorg-xinit xorg-xrandr xorg-xev xorg-xdpyinfo xorg-xprop
```

If you're planning to use a window manager like `awesome`, `bspwm` or `i3`, you should install X. While if you're planning to use `sway`, then wayland it is. If `GNOME`, you can install both. Again, your environment of choice will automatically install these as its dependencies.

#### Video Drivers

After installing the graphical server, we need to install the video drivers. I'm only using an integrated `AMD` graphics card.

For AMD graphics. Read the arch wiki on [ATI](https://wiki.archlinux.org/title/ATI) before procedding any futher.

``````
pacman -S xf86-video-ati vulkan-icd-loader libvdpau-va-gl  
``````
For Intel Graphics. Read the arch wiki on [Intel](https://wiki.archlinux.org/title/Intel_graphics)

```
pacman -S xf86-video-intel vulkan-intel vulkan-icd-loader libva-intel-driver
```

Add your (kernel) graphics driver to your initramfs. For example, if using `ati` add `radeon`: 

```
vim /etc/mkinitcpio.conf
```

Then add `radeon` to the `MODULES`:

```
MODULES=(radeon ...)
```

Generate the mkinitcpio file:

``````
mkinitcpio -p linux-lts
``````

#### Audio Drivers

```
pacman -S alsa-utils pulseaudio-alsa pulseaudio-bluetooth pulseaudio pavucontrol
```

#### File System Tools

File system tools

```
pacman -S unrar unzip p7zip unarchiver gvfs-mtp libmtp ntfs-3g \
android-udev mtpfs xdg-user-dirs
```

`xdg-user-dirs` is a tool to help manage "well known" user directories like the desktop folder and the music folder. This will be automatically run on your next log in. Though you can manually generate XDG user directories by:

```
xdg-user-dirs-update
```


#### Git

If you didn't include `git` on `pacstrap` earlier, it's time to install it now. This tool will come in handy later:

```
pacman -S git
```

#### AUR Helper

The "later" is now, old man. We will now install an AUR helper, `yay`.

Clone `yay-bin` from the AUR using `git`.

```
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin/
makepkg -sri
```

#### Missing Kernel Modules

If you noticed, there's a warning message while running `mkinitcpio -p linux`, fix this by installing these firmwares:

```
yay -S wd719x-firmware aic94xx-firmware --removemake --noconfirm
mkinitcpio -p linux-lts 
```

#### Desktop Environment and Window Manager

Install your preferred desktop environment or window manager. 

I'm an `alacritty` and `bspwm` guy.In this guide, I'll include a guide to set-up both `alacritty` and `bspwm`.

+ Alacritty

   - I always prefer to setup alacritty together with alacritty-themes. This enables me to change up alacritty's themes from time to time

```
yay -S alacritty alacritty-themes
```

+ Binary Space Partitioning Window Manager(bspwm)

   - I install bspwm together with sxhkd that serves as the X hot key daemon. 

      ```
      sudo pacman -S bspwm sxhkd
      ```

   - Picom as the compositor:

      ```
      sudo pacman -S picom
      ```

   - Rofi as the application launcher:

      ```
      sudo pacman -S rofi
      ```

#### Terminal Emulator

After installing an environment, we need a terminal emulator. Every linux user's first partner. Note that we're still on the `TTY`.

+ Install your preferred terminal emulator. 

   I'm an `alacritty` guy. In this guide, I'll include a guide to set-up `alacritty`

+ Alacritty

   - I always prefer to setup alacritty together with alacritty-themes. This enables me to change up alacritty's themes from time to time:

      ``````
      yay -S alacritty alacritty-themes
      ``````

#### GTK

GTK, or the GIMP Toolkit, is a multi-platform toolkit for creating graphical user interfaces.

If not yet installed:

```
pacman -S gtk3
```

Install GTK engines

```
pacman -S gtk-engine-murrine gtk-engines gnome-theme-extra
```

#### File Managers

KDE's `dolphin` is the best file manager in the Linux world, imho. So I always use this no matter what environment I'm using. We will also install `nnn`, a CLI-based file manager.

+ ensure you read the [nnn](https://wiki.archlinux.org/title/Nnn) wiki. 

```
pacman -S dolphin dolphin-plugins kde-cli-tools ranger
```

To generate thumbnails, I'll also install these:

+ kdegraphics-thumbnailers: Image files and PDFs
+ kimageformats: Gimp `.xcf` files
+ qt5-imageformats : `.webp`, `.tiff`, `.tga`, `.jp2` files
+ kdesdk-thumbnailers: Plugins for the thumbnailing system
+ ffmpegthumbs: Video files (based on ffmpeg)
+ raw-thumbnailer: `.raw` files
+ taglib : Audio files

```
pacman -S kdegraphics-thumbnailers kimageformats qt5-imageformats kdesdk-thumbnailers \
ffmpegthumbs raw-thumbnailer taglib
```

Enable preview showing of required file type in *Settings > Configure Dolphin... > General > Previews*.

There's a lot more thumbnail generators that can be found from the AUR (like a generator to create a thumbnail for `APK` files), but I don't really use them.

#### CLI-based Text Editor

'Nvchad' is my prefered choice for an IDE. Read the [NVchad](https://nvchad.github.io/) documentation before installation. 

Ensure you have installed `neovim` before you can install Nvchad. 

``````
sudo pacman -S neovim
``````

#### GUI-based Text Editors

`vim` is my text editor of choice, but sublime Text 3 is my go-to GUI text editor as it's lighter than the ~~bloated~~ chromium-based counterparts like `atom` and `vscode`.

```
yay -S sublime-text-dev
```

Note that Sublime is not "free" and needs a license.

#### Web browsers

`Brave-browser` is my trusted web browser.

```
yay -S brave-bin
```

#### Reboot then Login

The system's fully functional! You can now login to you system with all the configuration we've done so far. 

```
reboot
```

## Extras

#### Power Management for Laptops

TLP brings you the benefits of advanced power management for Linux without the need to understand every technical detail. **This is for laptops.**

Install and enable it now:

```
pacman -S tlp
systemctl enable --now tlp.service
```

Install `upower`, `acpid` and `acpi_call`:

```
pacman -S acpid acpi_call upower
```

Enable acpid

```
systemctl enable acpid.service
```
#### Firewall

We'll use `Uncomplicated Firewall` or `ufw` for short.

1. Install the `ufw` package. Start and enable `ufw.service` to make it available at boot. Note that this will not work if `iptables.service` is also enabled (and same for its ipv6 counterpart). 

	```
	pacman -S ufw
	```

2. Configuration

	Here's some basic configuration. A very simplistic configuration which will deny all by default, allow any protocol from inside a 192.168.0.1-192.168.0.255 LAN, and allow incoming Deluge and rate limited SSH traffic from anywhere: 

	```
	ufw default deny
	ufw allow from 192.168.0.0/24
	ufw allow Transmission
	ufw limit ssh
	```

3. The next line is only needed once the first time you install the package: 

	```
	ufw enable
	systemctl enable --now ufw.service
	```

Adding other applications. The PKG comes with some defaults based on the default ports of many common daemons and programs. Inspect the options by looking in the `/etc/ufw/applications.d` directory or by listing them in the program itself: 

```
ufw app list
```

#### Fonts

Improve fonts.

Install these fonts. Inter will be my system font no matter what the environment.

```
sudo pacman -S ttf-dejavu ttf-liberation noto-fonts noto-fonts-emoji inter-font ttf-roboto
```

Additional fonts to support Asian characters

```
sudo pacman -S noto-fonts-cjk noto-fonts-extra
```

Enable font presets by creating symbolic links:

```
ln -s /etc/fonts/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d
ln -s /etc/fonts/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d
ln -s /etc/fonts/conf.avail/11-lcdfilter-default.conf /etc/fonts/conf.d
```

The above will disable embedded bitmap for all fonts, enable sub-pixel RGB rendering, and enable the LCD filter which is designed to reduce colour fringing when subpixel rendering is used.

For font consistency, all applications should be set to use the `serif`, `sans-serif`, and `monospace` aliases, which are mapped to particular fonts by fontconfig.

Create `/etc/fonts/local.conf`, then add:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
	<match>
		<edit mode="prepend" name="family">
			<string>Inter</string>
		</edit>
	</match>
	<match target="pattern">
		<test qual="any" name="family">
			<string>serif</string>
		</test>
		<edit name="family" mode="assign" binding="same">
			<string>Noto Serif</string>
                </edit>
	</match>
	<match target="pattern">
		<test qual="any" name="family">
			<string>sans-serif</string>
		</test>
		<edit name="family" mode="assign" binding="same">
			<string>Noto Sans</string>
		</edit>
	</match>
	<match target="pattern">
		<test qual="any" name="family">
			<string>monospace</string>
		</test>
		<edit name="family" mode="assign" binding="same">
			<string>Noto Mono</string>
		</edit>
	</match>
</fontconfig>
```

Update and set your font of choice on settings.

---
### Artcile 1 [[Installation]](./README.md)
### Article 3 [[Extras]](./EXTRAS.md)
### Article 4 [[Web Development Setup]](./WEBDEVELOPMENT.md)
