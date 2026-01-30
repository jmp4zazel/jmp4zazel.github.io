
```c
int func()
{
    int i = 0x5ca1ab1e;
    return i;
}

int main()
{
    return func();
}
```

```asm
func:
140001540  mov     eax, 0x5ca1ab1e
140001545  retn     {__return_addr}

main:
140002910  sub     rsp, 0x28
140002914  call    __main
140002919  mov     eax, 0x5ca1ab1e
14000291e  add     rsp, 0x28
140002922  retn     {__return_addr}
```



