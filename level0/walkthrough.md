```
objdump -M intel -d level0 > /tmp/test.asm
```

Ouvrons le fichier dans vim puis cherchont le term "main" afin d'acceder à la fonction principale.
Nous remarquons une utilisation d'atoi:

```
8048ed4:       e8 37 08 00 00          call   8049710 <atoi>
8048ed9:       3d a7 01 00 00          cmp    eax,0x1a7
8048ede:       75 78                   jne    8048f58 <main+0x98>
```

0x1a7 est ègal à 423 en décimal, nous allons donc essayer de lancer l'executable avec cette valeur en argument:

```
./level0423
cat /home/user/level1/.pass
1fe8a524fa4bec01ca4ea2a869af2a02260d4a7d5fe7e7c24d8617e6dca12d3a
su level1
[flag]
```

Nous avons donc réussi !
