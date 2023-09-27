# Installing Golang

```bash
sudo apt install golang-go
```

Test the installation:

```bash
debian@lpi4a:~$ go version
go version go1.19.8 linux/riscv64
```

# Installing riscv64 Docker

The repository already contains the riscv64 Docker package named `docker.io`, so it can be directly installed:

```bash
sudo apt install docker.io
``` 

# Testing Docker

```bash
sudo docker pull riscv64/debian:unstable
```

# Issues

If you encounter the following error when using the command `sudo docker pull riscv64/debian:unstable`:

```bash
debian@lpi4a:~$ docker pull riscv64/debian:unstable
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/images/create?fromImage=riscv64%2Fdebian&tag=unstable": dial unix /var/run/docker.sock: connect: permission denied
```

Execute the following commands to fix it:

```bash
# 1. 
sudo chmod 666 /var/run/docker.sock

# 2.
sudo systemctl start docker

# 3.
sudo docker run hello-world
```