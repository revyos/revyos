# yquake2

Status:
Operational, supports GLES acceleration.

References:
https://github.com/yquake2/yquake2/blob/master/doc/020_installation.md
https://github.com/yquake2/yquake2/blob/master/doc/030_configuration.md

### Steps

```
# Install dependencies
sudo apt install build-essential libgl1-mesa-dev libsdl2-dev libopenal-dev libcurl4-openssl-dev

# Download source code & compile
git clone https://github.com/yquake2/yquake2.git
mkdir build
cd build
cmake ..
make

# Run
# (Ensure the original game resource folder 'baseq2' is prepared)
cd ..
cd release
cp -r ~/baseq2 .
./quake2
```

Place the original game resource folder 'baseq2' in the same directory as the quake2 program (compatible with the Steam version). Resolution and graphics acceleration options can be changed within the in-game settings menu. By default, it's set to software rendering. For hardware rendering, set it to "OpenGL ES3" (refer to the image below).

![](images/yquake2_1.png)