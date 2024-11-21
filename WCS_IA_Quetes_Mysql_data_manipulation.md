# INTRODUCTION AUX BASES DE DONNÉES RELATIONNELLES

## Objectifs

- Insérer du contenu dans une table
- Modifier du contenu
- Supprimer du contenu

## CHALLENGE

Effectue les requêtes suivantes :


1. **Insère dans la table school les données suivantes :**

```SQL
INSERT INTO school
  (name, country, capacity) 
VALUES
  ('Beauxbatons Academy of Magic', 'France', 550), 
  ('Castelobruxo', 'Brazil', 380), 
  ('Durmstrang Institute', 'Norway', 570),
  ('Hogwarts School of Witchcraft and Wizardry', 'United Kingdom', 450),
  ('Ilvermorny School of Witchcraft and Wizardry', 'USA', 300),
  ('Koldovstoretz', 'Russia', 125),
  ('Mahoutokoro School of Magic', 'Japan', 800),
  ('Uagadou School of Magic', 'Uganda', 350);
```
![](https://imgur.com/dtzzXw2.png)

2. **“Durmstrang Institute” est en réalité en Suède (Sweden), modifie son pays.**
```SQL
UPDATE school
SET country = "Sweden" 
WHERE name = "Durmstrang Institute";
```
![](https://imgur.com/vESd83w.png)


3. **“Koldovstoretz” passe à une capacité de 150.**
```SQL
UPDATE school 
SET capacity = 150 
WHERE name = "Koldovstoretz";
```
![](https://imgur.com/EgZ253G.png


4. **Supprime en une seule requête toutes les écoles comportant “Magic” dans leur nom (il y en a 3). Tu peux t’aider du mot clé LIKE.**
```SQL
DELETE FROM school
WHERE name LIKE "%Magic%";
```
![](https://imgur.com/dytrZpO.png)



5. **Affiche via une requête SELECT toutes les données de la table school.**
```SQL
SELECT * FROM school;
```
![](https://imgur.com/HR84xOC.png
