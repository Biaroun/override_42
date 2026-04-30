## Level 06

### Analyse

Le programme génère un serial unique basé sur le login. La fonction `auth` utilise `ptrace` pour détecter la présence d'un debugger — il faut contourner cette protection avant de pouvoir lire la valeur calculée.

On pose deux breakpoints : un sur l'appel `ptrace` pour bypasser la détection, un sur la comparaison finale pour lire le serial attendu :

```bash
(gdb) b *0x080487ba
(gdb) b *0x08048866
(gdb) r
-> Enter Login: AAAABBBB
-> Enter Serial: 1234
```

Au premier breakpoint (`ptrace`), on force le retour à 0 pour bypasser la détection du debugger :

```bash
Breakpoint 1, 0x080487ba in auth ()
(gdb) set $eax = 0
(gdb) c
```

Au second breakpoint (comparaison finale), on récupère la valeur hexadécimale du serial calculé stockée sur la pile à l'adresse `-0x10(%ebp)` :

```bash
Breakpoint 2, 0x08048866 in auth ()
(gdb) x/wx $ebp-0x10
0xffffd6a8:	0x005f1132
```

### Exploitation

Conversion et validation de l'authentification avec le login de test et le serial extrait :


```bash
$ python -c 'print 0x005f1132'
6230322

$ ./level06
-> Enter Login: AAAABBBB
-> Enter Serial: 6230322
Authenticated!
$ cat /home/users/level07/.pass
```
