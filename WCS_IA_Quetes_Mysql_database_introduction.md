# INTRODUCTION AUX BASES DE DONNÉES RELATIONNELLES

## Objectifs

- Comprendre ce qu'est une base de données
- Écrire tes premières requêtes SQL
- Créer ta première base de données
- Importer des données

## CHALLENGE

 ### Créer une base de donnée

Créer une base de données `wild_db_quest` et place toi dessus.

![](https://imgur.com/uwvYGsw.png)

### Créer une table

```SQL
CREATE TABLE wizard (

  id INT NOT NULL AUTO_INCREMENT,

  firstname VARCHAR(100) NOT NULL,

  lastname VARCHAR(100) NOT NULL,

  birthday DATE DEFAULT NULL,

  birth_place VARCHAR(255) DEFAULT NULL,

  biography TEXT DEFAULT NULL,

  PRIMARY KEY (id)

);

```

![](https://imgur.com/a5d5Iz7.png)

### Modifier une table

Modifie la table wizard en ajoutant un champ is_muggle de type booléen, obligatoire, qui permettra d’indiquer si le sorcier est ou non un moldu.

Si tu ne sais pas ce qu’est un moldu, c’est que tu en es sans doute un toi-même !

![](https://imgur.com/JGEeuFr.png)
![](https://imgur.com/bjMHaGs.png)


### Créer une table(bis)

Créer une table (bis)

Crée une table `school`, contenant les champs :

- id (clé primaire, entier auto-incrémenté, ne pouvant pas être NULL)
- name (chaîne de caractères de longueur 100, obligatoire = ne pouvant pas être NULL)
- capacity (entier, non obligatoire)
- country (chaîne de caractères de longueur 255, obligatoire)

![](https://imgur.com/tEBPb5C.png)


