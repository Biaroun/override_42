## Level 03

### Analyse

Le binaire lit un entier en entrée, effectue un calcul, puis s'en sert comme clé XOR pour déchiffrer une chaîne. Si le déchiffrement correspond à la chaîne attendue, `system("/bin/sh")` est exécuté.

**Flux d'exécution**

```
main()
  └─ test(input, 0x1337d00d)
       └─ si (0x1337d00d - input) <= 21
            └─ decrypt(0x1337d00d - input)
                 └─ XOR chaîne chiffrée avec la clé
                      └─ si résultat == chaîne attendue → system("/bin/sh")
```

**Fonctions clés**

`test` : calcule `clé = 0x1337d00d - input`. Si `clé <= 21`, appelle `decrypt(clé)`.

`decrypt` : applique un XOR octet par octet entre une chaîne chiffrée stockée sur la stack et la clé reçue, puis compare avec une chaîne de référence en mémoire.

---

### Trouver la clé XOR

On connaît les deux chaînes, on peut donc retrouver la clé par XOR inverse :

```bash
gdb ./level03
break decrypt
run

# Premier octet de la chaîne chiffrée (stockée à -0x1d(%ebp))
# issue de 0x757c7d51 en little-endian → premier octet = 0x51
x/x $ebp-0x1d
# → 0x51

# Premier octet de la chaîne de référence (à 0x80489c3)
x/s 0x80489c3
# → "Cable"  →  premier octet = 'C' = 0x43
```

**Calcul de la clé :**
```
clé = octet_chiffré XOR octet_référence
clé = 0x51 XOR 0x43 = 0x12 = 18
```

---

### Calcul de l'input

Dans `test` : `clé = 0x1337d00d - input`

```
0x1337d00d = 322 424 845

322 424 845 - input = 18
input = 322 424 845 - 18 = 322 424 827
```

---

### Valeurs

| Élément                  | Valeur hex     | Valeur décimale   |
|--------------------------|----------------|-------------------|
| Constante `0x1337d00d`   | `0x1337d00d`   | `322 424 845`     |
| Octet chiffré            | `0x51`         | `81`              |
| Octet référence (`'C'`)  | `0x43`         | `67`              |
| Clé XOR                  | `0x12`         | `18`              |
| **Input à fournir**      | —              | **`322 424 827`** |

---

### Exploitation

```bash
./level03
Password: 322424827

$ whoami
level04
$ cat /home/users/level04/.pass
```