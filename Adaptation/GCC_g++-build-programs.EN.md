# Instructions for Building Programs with gcc

## Installing gcc

First, ensure that gcc is installed:

```bash
sudo apt update
sudo apt install gcc
```

## Running the Program

Below is a demonstration of how to compile and run the simplest 'hello, world' program:

Compile the following content and name it `hello.c`:

```c
#include <stdio.h>

int main()
{
        printf("hello, world\n");
        return 0;
}
```

Compile and execute:

```bash
debian@lpi4a:~/test$ gcc -g hello.c -o hello

debian@lpi4a:~/test$ ./hello
hello, world

```

# Instructions for Building Programs with g++

First, ensure that g++ is installed:

```bash
sudo apt update
sudo apt install g++
```

## Running the Program

Below is a demonstration of how to compile and run the simplest 'hello, world' program:

Compile the following content and name it `hello.cpp`:

```c++
#include <iostream>
using namespace std;

int main()
{
    cout << "Hello, World!\n";
    return 0;
}

```

Compile and execute:

```bash
debian@lpi4a:~/test$ g++ -g hello.cpp -o hello
debian@lpi4a:~/test$ ./hello
Hello, World!
```

The above is the simplest example of compiling and running programs with gcc/g++. For more complex use cases, refer to the respective system programming manual.