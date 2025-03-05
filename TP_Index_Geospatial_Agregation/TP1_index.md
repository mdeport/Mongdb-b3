# TP1

## Exercice 1.1 

1. 
```json
const genres = ["Fiction", "Science-fiction", "Philosophie", "Histoire"];
const langues = ["Français", "Anglais", "Espagnol", "Allemand"];
const editeurs = ["Gallimard", "Hachette", "Flammarion", "Actes Sud"];

for (let i = 0; i < 1000; i++) {
  const livre = {
    titre: `Livre ${i}`,
    auteur: `Auteur ${i}`,
    genre: genres[Math.floor(Math.random() * genres.length)],
    langue: langues[Math.floor(Math.random() * langues.length)],
    editeur: editeurs[Math.floor(Math.random() * editeurs.length)],
    annee_publication: Math.floor(Math.random() * 100) + 1900,
    prix: Math.floor(Math.random() * 100) + 10,
    note_moyenne: Math.random() * 5,
    description: `Description du livre ${i}`,
    date_ajout: new Date(),
    isbn: `978-3-16-148410-0${i}`
  };
  db.livres.insertOne(livre);
}
```

2. Analisez les performances des requêtes suivantes sans index en utilisant explain("executionStats"):

```json
1. db.livres.find({titre: "Livre 500"}).explain("executionStats")
```

```json
2. db.livres.find({auteur: "Auteur 500"}).explain("executionStats")
```

```json
3. db.livres.find({prix: {$gte: 10, $lte: 20}, note_moyenne: {$gte: 2}}).explain("executionStats")
```

Une recherche filtré par genre et langue avec tri par note décroisante
```json
4. db.livres.find({genre: "Fiction", langue: "Français"}).sort({note_moyenne: -1}).explain("executionStats")
```

3.

| Requête | totalDocsExamined | executionTimeMillis | Type d'étape |
|---------|-------------------------|-----------------------------|--------------------------------|
| 1 | 1006 | 0 | COLLSCAN |
| 2 | 1006 | 0 | COLLSCAN |
| 3 | 1006 | 0 | COLLSCAN |
| 4 | 1006 | 1 | COLLSCAN |



## Exercice 1.2

1. Créez des index appropriés pour optimiser chacune des requetes précentes. Réfléchissez au choix entre index simples et com^posite en fonction des critère de recherche.

```json
db.livres.createIndex({titre: 1})
db.livres.createIndex({auteur: 1})
db.livres.createIndex({prix: 1, note_moyenne: 1})
db.livres.createIndex({genre: 1, langue: 1, note_moyenne: -1})
```

2.

| Requête | totalDocsExamined | executionTimeMillis | Type d'étape |
|---------|-------------------------|-----------------------------|--------------------------------|
| 1 | 1 | 1 | FETCH |
| 2 | 1 | 1 | FETCH |
| 3 | 76 | 1 | FETCH |
| 4 | 68 | 1 | FETCH |

## Exercice 1.3

1. Créez un index de texte (text index) sur les champs titre et description des livres.

```json
db.livres.createIndex({titre: "text", description: "text"})
```

2. Testez une recherche de texte simple et analysez ses performances.

```json 
db.livres.find({$text: {$search: "Livre 500"}}).explain("executionStats")
```

Il me retourne deux stage egal a fetch avec 

3. Créez une nouvelle collection sessions_utilisateurs avec des documents contenant :
• Un ID d'utilisateur
• Une date de dernière activité
• Des données de session

```json
db.sessions_utilisateurs.insertOne({
  utilisateur_id: ObjectId('67c5cccc4aa69e56de6daafb'),
  date_derniere_activite: new Date(),
  donnees_session: {
    langue: "fr",
    theme: "clair"
  }
})
```

4. Créez un index TTL sur cette collection pour supprimer automatiquement les sessions après 30 minutes d'inactivité.

```json
db.sessions_utilisateurs.createIndex(
  { derniere_activite: 1 },
  { expireAfterSeconds: 1800 }
)
```

## Exercice 1.4

1. Créez un index qui permet d'obtenir une requête couverte (covered query). Vérifiez que la requête n'examine aucun document (totalDocsExamined = 0).

```json
db.livres.createIndex({titre: 1, auteur: 1, genre: 1})
```

```json
db.livres.find({titre: "Livre 500", auteur: "Auteur 1"}, {_id: 0, genre: 1}).explain("executionStats")
```
totalDocsExamined: 0

2. Créez un index unique sur le champ ISBN des livres et testez son comportement en essayant d'insérer un document avec un ISBN dupliqué.

```json
db.livres.createIndex({isbn: 1}, {unique: true})
```

```json
db.livres.insertOne({
  titre: "Livre 1",
  auteur: "Auteur 1",
  genre: "Fiction",
  langue: "Français",
  editeur: "Gallimard",
  annee_publication: 2000,
  prix: 50,
  note_moyenne: 4,
  description: "Description du livre 1",
  date_ajout: new Date(),
  isbn: "978-3-16-148410-00"
})
```
il me met une erreur : E11000 duplicate key error collection: bibliotheque_amazon.livres index: isbn_1 dup key: { isbn: "978-3-16-148410-00" }

3. Créez un index partiel qui n'indexe que les livres disponibles.

```json
db.livres.createIndex({titre: 1}, {partialFilterExpression: {disponible: true}})
```


4. Activez le profiler MongoDB pour identifier les requêtes lentes (prenant plus de 100ms).

```json
db.setProfilingLevel(1, { slowms: 100 })
db.getProfilingStatus()
db.system.profile.find().sort({ millis: -1 })
```

5. Identifiez et supprimez un index redondant ou peu utilisé.

```json
db.livres.dropIndex("prix_1_note_moyenne_1")
```


- Quelles améliorations de performance avez-vous observées après l'ajout d'index ?

apres l'ajout d'un index j'ai pu voir que le nombre de document examiné a diminué pour ce focaliser sur les documents qui nous intéresse.

- Quels types d'index ont été les plus efficaces pour vos requêtes ?

Les index composites ont été les plus efficaces pour mes requêtes.

- Avez-vous identifié des compromis entre performance de lecture et d'écriture ?

Oui, les index peuvent ralentir les opérations d'écriture car ils doivent être mis à jour à chaque fois qu'un document est inséré.

- Comment choisiriez-vous les index pour une application de bibliothèque en production ?

Mettre des index sur les champs qui sont le plus utilisé pour des recherches.


