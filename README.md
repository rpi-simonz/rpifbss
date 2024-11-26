# rpifbss - Raspberry Pi Framebuffer Slideshow

This program provides a simple slideshow from a directory of files.
It uses the software 'fbi' for that.
So only still images are possible, no videos.

PDF files are displayed as well.
Even short text files (with extension .txt) are converted to images and shown.

The slideshow can be manipulated interactively with fbi's key shortcuts.
  * 'h' lists them on the framebuffer display
  * 'q' stops the slideshow, reads in possible new files and restarts the show

Ctrl-C ends this program.


## Requirements

### *fbi* with current version 2.14 or newer

The old one included in Raspberry Pi OS doesn't work.
So we need to compile it ourselves from source...

### Some of ImageMagick's policy.xml entries need to be adapted

```
sudo vi /etc/ImageMagick-6/policy.xml
      <!--  <policy domain="path" rights="none" pattern="@*"/>  -->
      <!--  <policy domain="coder" rights="none" pattern="PDF" />  -->

identify -list resource
```


### The system should be using the multi-user target, not graphical

```
systemctl list-units --type target --all

# Start system without graphical desktop
# temporary:
  sudo systemctl isolate multi-user
# permanent:
  sudo systemctl set-default multi-user
```

### To stop the blinking cursor on the frame buffer display

```
sudo -s
echo 0 > /sys/class/graphics/fbcon/cursor_blink
exit
```

