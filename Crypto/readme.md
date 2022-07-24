**Objectif :** Dans ce challenge, on nous fournit un algorithme en programme Python qui permet d'encoder un fichier et on a le fichier encodé.
Il s'appelle en effet CONFIDENTIEL et vu l'extension, il s'agit très probablement bien d'un fichier Excel.

On
1) Il n'y a pas de fonction de déchiffrement fourni dans le code source du programme Python ; or, c'est cette fonction qui va être requise pour obtenir le fichier d'origine (CONFIDENTIEL).
Le challenge s'oriente donc vers une action de reverse engineering de la fonction de chiffrement provisionnée, pour produire par nous-même une fonction de déchiffrement valable.
2) Au premier abord, l'analyse avec l'objectif de la fonction réciproque semble compliquée.
Il apparaît une fonction aléatoire "random" qui en général n'aide pas au reverse engineering puisque par définition il est difficile d'inverser quelque chose d'aléatoire. Ensuite, 
il est fait appel à une passe-phrase inconnue. Enfin, il existe manifestement une signature faisant appel à un SHA256 quasiment impossible (ou difficile) à décrypter.
Au final, il iy a en début de programme, deux tableaux d'entiers dont l'usage n'apparaît pas comme évident.
3) Cherchons à vérifier ce qui est vraiment utilisé dans la fonction : on recherche les éléments qui ne servent à rien, donc qui n'influent pas sur le résultat. Par exemple, la fonction:
   encryptedBlock = self.encryptBlock(ctr.to_bytes(16, 'big')) affecte une variable qui n'est jamais utilisée.
De fait, la fonction "encryptBlock" ne sert à rien. Donc, par effets de dominos, la passe-phrase qui génère une clé pour la fonction inutilisée "encryptBlock" ne sert à rien non plus; ce qui 
est une très bonne nouvelle car il n'y a pas vraiment de moyen d'obtenir cette valeur par cet appel aposteriori.

4) Il demeure la fonction "random" qui elle, est bien utilisée car elle sert à effectuer un XOR sur les octets (bytes) du fichier à encoder. Si on a le contrôle du XOR, il sera facile de décoder le fichier.
Le XOR est une fonction facilement réversible puisque c'est elle-même mais il faut quand même obtenir la valeur en question.
Il semble étrange au premier abord qu'il soit déployé une fonction "random" car il faut bien que celui qui est à la l'origine de l'encryptage puisse le décrypter ultérieurement à son tour.

5) La solution est simple : la sécurité est faible car la clé est directement collée en début de fichier puisque c'est elle qui initialise la valeur "encrypted" de chiffrement du fichier.
La sécurité voulue du fichier est donc compromise ; tout simplement parce que la clé est donnée par les 16 premiers bits du fichier.
La prochaine étape est donc de simplifier le code de l'encodeur; en ne gardant que ce qui est utile et en commentant tout ce qui ne sert pas: on obtient le programme allégé "encrypt_simple1.PY".
6) Ensuite, l'étape suivante est de coder la partie "decrypt" qui va inverser du codage ((trouver la réciproque).
7) D'abord, on enlève la partie initiale "ctr" en lisant les 16 premiers bits qui correspondent à la clé aléatoire ("random"): on peut appliquer à l'identique la fonction XOR dont l'inverse est elle-même.
Du coup, on obtient un fichier décodé à un détail près qui est que le fichier d'origine a subi un padding en ajoutant par remplissage des zéros à la fin pour faire de ce tableau de bits un multiple de 16 bits.
On va juste rajouter à la fin une fonction qui rejette les zéros trouvés à la fin.
On lance le programme Python avec en entrée le fichier CONFIDENTIEL encodé plus une passe-phrase au hasard (sans intérêt) -o nom_de_fichier_de_sortie (CONFIDENTIEL.xls) puis on ouvre le fichier résultant sous Excel
au vu de l'extension : on trouve directement le flag à l'intérieur.

Le flag est HACK{ImplementationErrorBreaksCiphers}
Voir la solution.
