## Level 06

### Analyse

Le programme génère un serial unique basé sur le login. L'analyse GDB permet d'extraire la valeur attendue directement en mémoire avant la comparaison.

On pose un breakpoint juste avant l'instruction de comparaison cmp dans la fonction auth pour lire la valeur du hash calculé:
```bash
(gdb) b *0x080487ba
(gdb) r
-> Enter Login: AAAABBBB
-> Enter Serial: 420
```
On récupère la valeur hexadécimale stockée sur la pile à l'adresse -0x10(%ebp).
```bash
(gdb) x/wx $ebp-0x10
0xffffd5f8:	0x005f1132
```

### Exploitation

Validation de l'authentification avec le login de test et le serial extrait :

```bash
$ python -c 'print 0x005f1132'
6230322

$ ./level06
-> Enter Login: AAAABBBB
-> Enter Serial: 6230322
Authenticated!
$ cat /home/users/level07/.pass
```
