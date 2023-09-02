# gcc build programs 说明

## 安装gcc

首先确保安装gcc:

```bash
sudo apt update
sudo apt install gcc
```

## 运行程序

下面以最简单的 'hello,world' 程序为例演示如何编译并运行:

编译以下内容并命名为 `hello.c`:

```c
#include <stdio.h>

int main()
{
        printf("hello, world\n");
        return 0;
}
```

编译并执行:

```bash
debian@lpi4a:~/test$ gcc -g hello.c -o hello

debian@lpi4a:~/test$ ./hello
hello, world

```

# g++ build programs 说明


首先确保安装g++:

```bash
sudo apt update
sudo apt install g++
```

## 运行程序

下面以最简单的 'hello,world' 程序为例演示如何编译并运行:

编译以下内容并命名为 `hello.cpp`:

```c++
#include <iostream>
using namespace std;

int main()
{
    cout << "Hello, World!\n";
    return 0;
}

```

编译并执行：

```bash
debian@lpi4a:~/test$ g++ -g hello.cpp -o hello
debian@lpi4a:~/test$ ./hello
Hello, World!
```

以上就是gcc/g++ 编译程序并运行的最简单实例，更复杂的应用案例可以参考相应的
系统编程手册。
