Boolean Operators
We will use 0 as shorthand for "false" and 1 as shorthand for "true".

AND
AND is true if both of the inputs are true. 

0 AND 0 = 0

0 AND 1 = 0

1 AND 0 = 0

1 AND 1 = 1

OR
OR is true if either input is true.

0 OR 0 = 0

0 OR 1 = 1

1 OR 0 = 1

1 OR 1 = 1

XOR
Exclusive-OR is true if only one input is true.

0 XOR 0 = 0

0 XOR 1 = 1

1 XOR 0 = 1

1 XOR 1 = 0

NOT
NOT flips the value to the opposite value

NOT 1 = 0

NOT 0 = 1

Boolean logic in C
```
Logical operators
In C "logical" boolean operations, which operate over a statement and are interpreted as true if the result is non-zero or false if the result is zero.

The logical AND operator is &&

The logical OR operator is ||

The logical NOT operator is !

Bitwise operators
In C "bitwise" boolean operations, the boolean operation is computed individually on each bit in the same index of the two operands. So if you're doing a bitwise AND of two variables, it is computed as:

output_bit[0] = input1_bit[0] AND input2_bit[0]

output_bit[1] = input1_bit[1] AND input2_bit[1]

...

output_bit[N] = input1_bit[N] AND input2_bit[N]

The bitwise AND operator is &

The bitwise OR operator is |

The bitwise XOR operator is ^

The bitwise NOT operator is ~
```c
#include <stdio.h>
#define uint64 unsigned long long

unsigned long long main ()
{
  unsigned int i = 0x50da;
  unsigned int j = 0xc0ffee;
  uint64 k = 0x7ea707a11ed;
  k ^= ~( i & j ) | 0x7ab00;
  return k;
}

```

```
140001540  55                 push    rbp {__saved_rbp}
140001541  4889e5             mov     rbp, rsp {__saved_rbp}
140001544  4883ec30           sub     rsp, 0x30
140001548  e813010000         call    __main
14000154d  c745fcda500000     mov     dword [rbp-0x4 {var_c}], 0x50da
140001554  c745f8eeffc000     mov     dword [rbp-0x8 {var_10}], 0xc0ffee
14000155b  48b8ed117a70ea07…  mov     rax, 0x7ea707a11ed
140001565  488945f0           mov     qword [rbp-0x10 {var_18}], rax  {0x7ea707a11ed}
140001569  8b45fc             mov     eax, dword [rbp-0x4]
14000156c  2345f8             and     eax, dword [rbp-0x8]
14000156f  f7d0               not     eax
140001571  0d00ab0700         or      eax, 0x7ab00
140001576  89c0               mov     eax, eax  {0xffffaf35}
140001578  483145f0           xor     qword [rbp-0x10], rax  {0x7ea8f85bed8}
14000157c  488b45f0           mov     rax, qword [rbp-0x10]  {0x7ea8f85bed8}
140001580  4883c430           add     rsp, 0x30
140001584  5d                 pop     rbp {__saved_rbp}
140001585  c3                 retn     {__return_addr}
```

we can see the new instructions AND, NOT, OR, and XOR

## AND - Bitwise AND
C bianr operator & not && that's logical AND, destination operand can be r/mx or register, source operand can be r/mx or register or immediate no source and destination as r/mXs.

```plaintext
and al, bl

       00110011b ( al - 0x33 )
AND    01010101b ( bl - 0x55 )

RESULT 00010001b ( al - 0x11 )
```

btw we calculated this by vertically comparing em, left t oright, 1 to 1 is 1 1 to 0 is 0. etc. because AND is only if both inputs are true.

**AND al, 0x42**
```plaintext

       00110011b ( al - 0x33  )
AND    010000    ( imm - 0x42 )

RESULT 00000010b ( al - 0x02  )
```

## OR - Bitwise OR
C binary operator | not || that's logical OR, destination operand can be r/mX or register, source operand can be r/mX or register or immediate, no source and desintation as r/mXs. calculation of this is if either is true.

**OR al, bl**
```plaintext

       00110011b ( al - 0x33 )
OR     01010101b ( bl - 0x55 )

RESULT 01110111b ( al - 0x77 )
```

**OR al, 0x42**
```plaintext

       00110011b ( al - 0x33  )
OR     01000010b ( imm - 0x42 ) 

RESULT 01110011b ( al - 0x73  )
```

## XOR - Bitwise Exlusive XOR
C binary operator ^, Destiantion operand can be r/mX or register soruce operand can be r/mX or register or immedaite no sourve an dedstination as  r/mXs. XOr is commonly used to zero a register, by xoring it with itself becaue its fastetr than a MOV. so frequently compiler will generate a XOR of a register with itself in order t o zer othat register

```plaintext
       00110011b ( al - 0x33 )
XOR    00110011b ( al - 0x33 )

RESULT 00000000b ( al - 0x00 )
```

all 0s becaues its only true if oe input is true.

## NOT - One's Complement Negation
C binary operator ~ not ! that's logical NOT, single source/destiatnion operand can be r/mX. NOT only flips the bits. that's all

# For LOops

C code:
```
#include <stdio.h>

int main ()
{
  int i;
  for ( i = 0; i < 10; i++ )
  {
    printf ( "i = %d\n", i );
  }
  i--;
}

```

Dissassembled:
```
140001540    int main()

140001540  55                 push    rbp {__saved_rbp}
140001541  4889e5             mov     rbp, rsp {__saved_rbp}
140001544  4883ec30           sub     rsp, 0x30
140001548  e803010000         call    __main
14000154d  c745fc00000000     mov     dword [rbp-0x4 {i}], 0x0 
140001554  eb15               jmp     0x14000156b

140001556  8b45fc             mov     eax, dword [rbp-0x4 {i}]
140001559  488d0df02a0000     lea     rcx, [rel _.rdata]  {"i = %d\n"}
140001560  89c2               mov     edx, eax
140001562  e859110000         call    printf
140001567  8345fc01           add     dword [rbp-0x4 {i}], 0x1 ; normally in Visual Studio you're going to see inc (for increment)

14000156b  837dfc09           cmp     dword [rbp-0x4 {i}], 0x9
14000156f  7ee5               jle     0x140001556

140001571  836dfc01           sub     dword [rbp-0x4 {var_c} {i}], 0x1 ; and dec (for decrement) but we're in binary ninja so
140001575  b800000000         mov     eax, 0x0 ; normally we'll also see xor eax, eax in visual studio because the programmer forgot to put return 0
14000157a  4883c430           add     rsp, 0x30
14000157e  5d                 pop     rbp {__saved_rbp}
14000157f  c3                 retn     {__return_addr}
```

### inc/dev (increment/decrement) (add/sub)
single source/destination oprand can be r/mX increase or decrease the value by 1, whn optimized compiler will tend to favor to not using inc/dev that's why we're seeing add and sub in binary ninja becuase its directed the by the intel optimization guide. SO their presence maybe be indicative of hand-written or un-optimized code. This modifies OF, SF, ZF, AF, PF, and CF flags.

```plaintext
rax 0xbe5077ed
```

**xor rax, rax**
```plaintext
rax 0x0
```

**inc rax**
```plaintext
rax 0x1:
```

another example
```plaintext
rax 0x70ad57001
```

**mov rax, 0**
```plaintext
rax 0x0
```

**dec rax**
```
rax 0xFFFFFFFF'FFFFFFFF
```

