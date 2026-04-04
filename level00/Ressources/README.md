## Level 00

L'analyse du binaire via GDB nous a permis de comprendre la logique du programme. Une instruction cmp compare l'input à la valeur hexadécimale 0x149c, qui correspond au nombre 5276 en décimal

### Analyse


```bash
$ gdb ./level00
(gdb) disas main
...
0x080484e7 <+83>:    cmp    $0x149c,%eax
...
```


### Exploitation

On entre 5276 au prompt Password pour valider la comparaison avec 0x149c, ce qui déclenche un shell avec les privilèges de l'utilisateur suivant et permet de lire le fichier /home/users/level01/.pass

```bash
$ ./level00
***********************************
*            -Level00 -           *
***********************************
Password:5276

Authenticated!
cat /home/users/level01/.pass
```
