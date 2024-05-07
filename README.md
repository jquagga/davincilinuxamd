# Running Davinci Resolve with AMD's rocm using RockyLinux and distrobox

So these are my notes on running Blackmagic Design's excellent Resolve editor (18.6.4 at the time) on Linux. There are presently two sets of issues which make getting a working editor running so I'll note them quickly and then show what I did to resolve them.

### Resolve crashes on Linux on AMD rocm

With newer versions of rocm, Resolve crashes on startup. Before more recent updates to rocm, I was able to run Resolve on Fedora 38/39 directly. However an update to the AMD graphics driver or kernel leads to the program just exiting. This seems to happen on other distributions (Arch) as well as I tried them on the road to my present solution below.

### Resolve on Linux has no Timeline!

This is different than the usual "not supported codec so the file doesn't show on the timeline" issue. In this case, the timeline doesn't work at all. Starting with Resolve 18.5+ (18.1 does seem to work) the usual trick with deleting glib, gmodule, etc from the resolve distribution to utilize the distro provided versions stops working. Well it works in that Resolve starts but then it doesn't have a timeline and isn't really usable. This was the challenge I had with [davincibox](https://github.com/zelikos/davincibox). I could install on the Fedora base but then couldn't actually use the program.

## Run Resolve in a RockyLinux Distrobox with an older version of rocm

The header spoils the solution, but I found that if you use RockyLinux 8, install the required support packages and an older rocm (I'm using 5.4.5 to match approximately what is in Fedora 37/38) you can use Davinci without any modifications. You _do not_ need to remove glib, etc. This is the equivalent of using the BlackMagic provided ISO (except that only has nvidia drivers and no AMD drivers). You could also use this step on Rocky to install the older AMD rocm and in theory that would work as well. I'm running Rocky in a distrobox on Fedora 39 and so far so good. It's worth noting if you do use Rocky directly, the epel packages for AMD in RockyLinux aren't fully complete - you have add the AMD repository below to get them to install. This was my major hangup - when I did this the directions had my install rocm 6.0 and Resolve doesn't like anything that new (it crashes on gpudetect). But if you install something a bit more vintage, we are good!

### Actual directions

This is basically a step by step getting an image named rocky up and running. If you get this up and distrobox runs, I wouldn't update it. Leave it and if a new davinci comes out or you want to try a new distrobox, create a new distrobox and leave this one as-is to keep working. I would imagine that updates to the host amdgpu driver might break it, but hopefully not.  Good luck out there!

```bash
distrobox create --name rocky --image quay.io/toolbx-images/rockylinux-toolbox:8
distrobox enter rocky
sudo dnf install alsa-lib apr apr-util fontconfig freetype libglvnd libglvnd-egl libglvnd-glx libglvnd-opengl libgomp librsvg2 libXcursor libXfixes libXi libXinerama libxkbcommon-x11 libXrandr libXrender libXtst libXxf86vm mesa-libGLU mtdev pulseaudio-libs xcb-util xcb-util-image xcb-util-keysyms xcb-util-renderutil xcb-util-wm alsa-plugins-pulseaudio xcb-util-cursor
unzip DaVinci_Resolve_19.0b2_Linux.zip
./DaVinci_Resolve_19.0b2_Linux.run --appimage-extract
sudo QT_QPA_PLATFORM=minimal squashfs-root/AppRun -i -a -y


sudo tee --append /etc/yum.repos.d/rocm.repo <<EOF
[ROCm]
name=ROCm
baseurl=https://repo.radeon.com/rocm/rhel8/5.4.5/main/
enabled=1
priority=50
gpgcheck=1
gpgkey=https://repo.radeon.com/rocm/rocm.gpg.key
EOF

sudo yum clean all

sudo dnf install rocm-opencl rocm-opencl-sdk
/opt/resolve/bin/resolv
```
