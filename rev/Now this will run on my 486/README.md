# Now this will run on my 486? (50 Points)

## 0x0 Overview

바이너리를 확인하면 main 내부에서 sigaction() 함수를 통해 SIGILL의 signal handler를 `sub_12C8`로 새로 지정한다. SIGILL는 Illegal Instruction을 감지했을 때 발생한다.
```c
v6.sa_flags = 4
v6.sa_handler = (__sighandler_t)sub_12C8
sigaction(4, &amp;v6, 0LL)
```
이후 특별한 입력 없이 특정 메모리에 권한을 부여하고 실행시킨다.

## 0x01 Analysis

당연하게도 SIGILL 이 발생하고 때문에 signal handler로 지정한 함수가 실행될 것이다. 해당 함수는 vm처럼 현재 메모리의 값을 각 연산에 맞추어 복호화해주고 다시 sigretrun을 통해 돌아간다. 복호화된 메모리는 프로세스가 이해하는 instruction으로 바뀌게 되고 때문에 이를 해석해서 flag 값을 얻어내면 된다. strace로 해당 프로그램을 trace 하면 vm으로 복호화된 instruction들의 의해 어떻게 프로그램이 돌아가는지 확인할 수 있다.
read 이후 sigill 을 계속 발생시켜 프로그램을 복호화시킨 어셈을 확인하면서 조건을 맞추어 주면 된다.
가장 처음 조건은 0x1f 길이의 문자의 합이 0xcff가 되어야 하고 이후부터는 4글자씩 xor 연사를 해주어 플래그와 일치하는지 확인한다. 복호화된 어셈 안에는 `a ^ b = c` 에서 b, c가 존재하기 때문에 입력해야 하는 a 값을 역연산을 통해 알아낼 수 있다. 프로그램이 종료되기 전에 계속해서 메모리를 확인해 4바이트씩 플래그를 복호화해주면 문제를 풀 수 있다. 스크립트를 작성하면 편하겠지만, 0x1f 길이어서 손으로 직접 하였다.

signal handler를 디버깅하기 위해서는 gdb에서 다음 옵션을 켜주어야 handler에 bp를 걸 수 있다.

```
handle SIGILL pass
handle SIGILL nostop
```


