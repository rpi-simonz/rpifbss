# rpifbss - Raspberry Pi Framebuffer Slideshow

This program provides a simple framebuffer slideshow from a directory of files.

It uses the software 'fbi' for that.
So only still images are possible, no videos.

PDF files are displayed as well.
Even short text files (with extension .txt) are converted to images and shown.

The directory containing the files to be displayed can be defined by the first
commandline argument. Otherwise the current working directory is used.

The slideshow can be controlled interactively with fbi's key shortcuts.

  * 'h' lists all of them on the framebuffer display, see below
  * 'q' stops the current slideshow, and then

    * reads in possible new or modified files
    * converts unseen text files
    * and restarts the slideshow with the new contents

```
keyboard commands
~~~~~~~~~~~~~~~~~
  cursor keys    - scroll image
  PgUp, k        - previous image
  PgDn, SPACE, j - next image
  <i>g           - jump to image #i

  a              - autozoom image
  +/-            - zoom in/out
  <i>s           - set zoom to <i>%

  ESC, q         - quit
  v              - toggle statusline
  h              - show this help text
  i              - show EXIF info
  p              - pause slideshow

available if started with --edit switch,
rotation works for jpeg images only:
  D, Shift+d     - delete image       *** really, without confirmation dialog! ***
  r              - rotate 90 degrees clockwise
  l              - rotate 90 degrees counter-clockwise
  x              - mirror image vertically (top / bottom)
  y              - mirror image horizontally (left to right)
```

During re-reading and possibly converting new files, what can need several seconds,
there is a maintenance message displayed.
This message is stored in a text file in the automatically created subdirectory 'maintenance/'
and can be adapted to individual needs.

Ctrl-C ends this program and therfore the slideshow completely.

The configuration is done in the script *rpifbss* itself, in a 'Configuration section' at the beginning.

To have a comfortable environment it is recommended to start *rpifbss* in a *tmux* or *screen* session.
Then the running slideshow is always accessible, even remotely via an SSH connection.


## Requirements and preparations

### Raspberry Pi OS

Tested with Bookworm (Debian 12) 64-Bit Desktop. TODO: Test lite version.

Should work on older releases too, but that has not been tested yet.


### *fbi* with current version 2.14 or newer

The really old version 2.09 included in Raspberry Pi OS doesn't work well.
So we need to compile fbi ourselves from source...

#### Compile fbi from source

Download *fbida-master.zip* from https://github.com/kraxel/fbida

Then - in a terminal - change into the folder where the downloaded file is, and:

```
unzip fbida-master.zip
cd fbida-master

# Install dependencies
sudo apt install -y meson
sudo apt install -y cmake
sudo apt install -y libpoppler-glib-dev
sudo apt install -y libdrm-dev
sudo apt install -y libexif-dev
sudo apt install -y libtiff-dev
sudo apt install -y libudev-dev
sudo apt install -y libinput-dev
sudo apt install -y libtsm-dev
sudo apt install -y libsystemd-dev
sudo apt install -y libgif-dev
sudo apt install -y libxkbcommon-dev

make

# For a short test
build-meson-${HOSTNAME}/fbi -d /dev/fb0  ~/some_image.jpg
```

Unfortunately the provided Makefile has a missing target entry...

Edit it with

```
nano Makefile
```

and add these two lines at the bottom (important: That's a leading TAB in the second line!):

```
install:
	$(MESON) install -C $(BDIR)
```

Following the fbida documentation won't work on standard Raspberry Pi OS
because the user *root* isn't usable there by default.

Instead of

```
su -c "make install"
```

better use this:

```
sudo make install
```

In case the original old *fbi* is already/still installed it is recommended
to uninstall it:

```
sudo apt purge fbi
```

Or at least we should clean up the current shell's cache by:

```
hash -r fbi
```


### Some of ImageMagick's policy restrictions need to be adapted

```
sudo nano /etc/ImageMagick-6/policy.xml
```

The following two entries need to be commented out as shown:

```
    <!--  <policy domain="path" rights="none" pattern="@*"/>  -->
    <!--  <policy domain="coder" rights="none" pattern="PDF" />  -->
```

Additionally show some more configured limits for ImageMagick:

```
identify -list resource
```


### The system should be using the multi-user target, not the graphical one

List available targets:
```
systemctl list-units --type target --all
```

Stop graphical desktop:

```
# temporarily:
sudo systemctl isolate multi-user

# permanently (recommended):
sudo systemctl set-default multi-user
sudo systemctl isolate multi-user
```

TODO: Check 'lite' systems and update this readme if needed.


### Stop the blinking cursor usually visible on the frame buffer display

```
sudo -s
echo 0 > /sys/class/graphics/fbcon/cursor_blink
exit
```

