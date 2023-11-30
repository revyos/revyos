# yquake2

状态：
可以运行，支持GLES加速

资料：
https://github.com/yquake2/yquake2/blob/master/doc/020_installation.md
https://github.com/yquake2/yquake2/blob/master/doc/030_configuration.md

### 步骤
```
# 安装依赖
sudo apt install build-essential libgl1-mesa-dev libsdl2-dev libopenal-dev libcurl4-openssl-dev

# 下载代码&编译
git clone https://github.com/yquake2/yquake2.git
mkdir build
cd build
cmake ..
make

# 运行
#（需要准备好游戏原始资源文件夹baseq2）
cd ..
cd release
cp -r ~/baseq2 .
./quake2
```

需要将原始游戏资源文件夹baseq2放到和quake2程序同一个目录中（Steam版可用），分辨率和图形加速选项在游戏内设置菜单修改，不修改默认是软渲染，硬渲染设置请改为“OpenGL ES3”（参见下图）
![](images/yquake2_1.png)
