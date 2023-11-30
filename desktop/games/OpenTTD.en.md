# OpenTTD

Status:
Runs successfully, software rendering for graphics.

References:
https://github.com/OpenTTD/OpenTTD/blob/master/COMPILING.md
https://salsa.debian.org/openttd-team/openttd/-/blob/master/debian/control

### Steps

```
# Install dependencies
sudo apt install libsdl2-dev zlib1g-dev libpng-dev libfreetype-dev libfontconfig-dev libicu-dev liblzo2-dev liblzma-dev libfluidsynth-dev libopengl-dev grfcodec openttd-opengfx cmake

# Download source & compile
git clone https://github.com/OpenTTD/OpenTTD.git
cd OpenTTD
mkdir build
cd build
cmake ..
make

# Run
./openttd
```

Graphic and audio resource files can be downloaded through in-game features. Chinese language support is available.

![](images/openttd_1.png)