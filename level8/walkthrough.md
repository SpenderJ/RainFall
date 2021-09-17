On trouve un executable:
```
level8@RainFall:~$ ls
level8
level8@RainFall:~$ ./level8
(nil), (nil)
jj
(nil), (nil)
nil
(nil), (nil)
(nil), (nil)
(nil), (nil)
^C
```

On desassemble le main afin de comprendre ce qu'il s'y passe:

```
objdump -M intel -d level8
 8048575:	8b 0d b0 9a 04 08    	mov    ecx,DWORD PTR ds:0x8049ab0
 804857b:	8b 15 ac 9a 04 08    	mov    edx,DWORD PTR ds:0x8049aac
 8048581:	b8 10 88 04 08       	mov    eax,0x8048810
 8048586:	89 4c 24 08          	mov    DWORD PTR [esp+0x8],ecx
 804858a:	89 54 24 04          	mov    DWORD PTR [esp+0x4],edx
 804858e:	89 04 24             	mov    DWORD PTR [esp],eax
 8048591:	e8 7a fe ff ff       	call   8048410 <printf@plt>
```

Interessons nous aux 2 premieres lignes il s'agit de la definition de 2 variabnles globales:
```
gdb ./level8
break fgets # on met un breakpoint sur le premier fgets
r
x/20w 0x08049aac # regardons la premiere variable
x/20w 0x08049ab0 # et la deuxieme
```

On voit qu'il s'agit de 2 variables globales appelées auth et service.
On voit par la suite que auth se fait malloc et que la data après auth est enregistrée dans un malloc(4) avec un strcpy
Pareil pour service mais avec un strdup(8).

```
./level8
(nil), (nil)
auth a
0x804a008, (nil)
service a
0x804a008, 0x804a018
```

Maintenant malheuresement l'etape que tu redoutes arrive :).
Finit de trouver des bugs dans du source code, il va falloir lire de la doc et traduire en C le code :)
VOus retrouverez le pseudo code en C dans le fichier source
