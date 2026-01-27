```asm
main:
140002910  sub     rsp, 0x28
140002914  call    __main
140002919  mov     eax, 0xf00d
14000291e  add     rsp, 0x28
140002922  retn     {__return_addr}
```
