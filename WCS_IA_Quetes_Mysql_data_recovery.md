# RÉCUPÉRER DES INFORMATIONS AVEC `SELECT`

## Objectifs
- Connexion à la base de données, importation de données
- Récupération des données avec SELECT
- Utilisation de différentes clauses pour filtrer et trier les résultats

## CHALLENGE

### Trouve les requêtes SQL pour :

1. **Récupère tous les champs pour les sorciers nés entre 1975 et 1985:**

```SQL
SELECT * FROM wizard WHERE is_muggle = 0 AND birthday BETWEEN '1975-01-01' AND '1985-12-31';
```
![](https://imgur.com/wLO46lu.png)


2. **Le prénom uniquement des sorciers dont le prénom commence par la lettre ‘H’.**

```SQL
SELECT firstname FROM wizard WHERE is_muggle = 0 AND UPPER(firstname) LIKE 'H%';
```
![](https://imgur.com/xkkk0xJ.png)


3, **Les prénoms et noms de tous les membres de la famille ‘Potter’, classés par ordre de prénom.**

```SQL
SELECT firstname, lastname FROM wizard WHERE lastname = "potter" ORDER BY firstname ASC;
```
![](https://imgur.com/DKEVafT.png)


4. **Le prénom, nom et date de naissance du plus vieux sorcier (doit fonctionner quelque soit le contenu de la table).**

```SQL
SELECT firstname, lastname, birthday FROM wizard WHERE is_muggle = 0 ORDER BY birthday ASC LIMIT 1;
```
![](https://imgur.com/t4OrIbg.png)
