# 安装 golang

```bash
sudo apt install golang-go
```

测试安装是否成功:

```bash
debian@lpi4a:~$ go version
go version go1.19.8 linux/riscv64
```

# 安装 riscv64 docker
源中已有 riscv64 的docker安装包，名为 `docker.io`，可以直接使用:

```bash
sudo apt install docker.io
``` 

# 测试 docker

```bash
sudo docker pull riscv64/debian:unstable
```

# issues

如果使用命令`sudo docker pull riscv64/debian:unstable` 出现以下错误:

```bash
debian@lpi4a:~$ docker pull riscv64/debian:unstable
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/images/create?fromImage=riscv64%2Fdebian&tag=unstable": dial unix /var/run/docker.sock: connect: permission denied

```
需要执行以下命令进行修复:

```bash
# 1. 
sudo chmod 666 /var/run/docker.sock

# 2.
sudo systemctl start docker

# 3.
sudo docker run hello-world
```

