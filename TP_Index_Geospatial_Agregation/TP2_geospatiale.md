# TP2

database livre

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

database utilisateur

```json
const prenoms = ["Jean", "Marie", "Pierre", "Sophie", "Thomas",
"Isabelle"];
const noms = ["Martin", "Bernard", "Dubois", "Thomas", "Robert",
"Richard"];
const villes = [
  { nom: "Paris", coordinates: [2.3522, 48.8566] },
  { nom: "Lyon", coordinates: [4.8357, 45.7640] },
  { nom: "Marseille", coordinates: [5.3698, 43.2965] }
];

for (let i = 0; i < 1000; i++) {
  const prenom = prenoms[Math.floor(Math.random() * prenoms.length)];
  const nom = noms[Math.floor(Math.random() * noms.length)];
  const ville = villes[Math.floor(Math.random() * villes.length)];
  const age = Math.floor(Math.random() * 100) + 18;
  const utilisateur = {
    prenom: prenom,
    nom: nom,
    email: `${prenom.toLowerCase()}.${nom.toLowerCase()}@example.com`,
    age: age,
    adresse: {
      rue: `${Math.floor(Math.random() * 100) + 1} rue de la Paix`,
      ville: ville.nom,
      code_postal: `${Math.floor(Math.random() * 10000) + 10000}`,
      localisation: {
          type: "Point",
          coordinates: ville.coordinates
      }
    },
    date_inscription: new Date()
  };
  db.utilisateurs.insertOne(utilisateur);
}
```

## Exercice 2.1

1. Modifiez le schéma de vos utilisateurs pour inclure des coordonnées géographiques dans leur adresse. Utilisez le format GeoJSON Point :


2. Créez une nouvelle collection bibliotheques avec au moins 3 bibliothèques dans différentes villes.
Chaque document doit contenir :
    • Un nom et une adresse
    • Des coordonnées GeoJSON Point pour la localisation
    • Une zone de service définie comme un polygone GeoJSON

```json
db.bibliotheques.insertMany([
  {
    nom: "Bibliothèque Nationale de France",
    adresse: "Quai François Mauriac, 75013 Paris",
    localisation: {
      type: "Point",
      coordinates: [2.3751, 48.8331]
    },
    zone_service: {
      type: "Polygon",
      coordinates: [
        [
          [2.3741, 48.8331],
          [2.3761, 48.8331],
          [2.3761, 48.8341],
          [2.3741, 48.8341],
          [2.3741, 48.8331]
        ]
      ]
    }
  },
  {
    nom: "Bibliothèque Municipale de Lyon",
    adresse: "30 Boulevard Vivier Merle, 69003 Lyon",
    localisation: {
      type: "Point",
      coordinates: [4.8554, 45.7600]
    },
    zone_service: {
      type: "Polygon",
      coordinates: [
        [
          [4.8544, 45.7600],
          [4.8564, 45.7600],
          [4.8564, 45.7610],
          [4.8544, 45.7610],
          [4.8544, 45.7600]
        ]
      ]
    }
  },
  {
    nom: "Bibliothèque de Marseille",
    adresse: "7 Rue des Beaux-Arts, 13001 Marseille",
    localisation: {
      type: "Point",
      coordinates: [5.3770, 43.2965]
    },
    zone_service: {
      type: "Polygon",
      coordinates: [
        [
          [5.3760, 43.2965],
          [5.3780, 43.2965],
          [5.3780, 43.2975],
          [5.3760, 43.2975],
          [5.3760, 43.2965]
        ]
      ]
    }
  }
]);
```
3. Créez les index géospatiaux nécessaires sur les collections utilisateurs et bibliotheques.

```json
db.utilisateurs.createIndex({ "adresse.localisation": "2dsphere" });
db.bibliotheques.createIndex({ "localisation": "2dsphere"});
```


## Exercice 2.2
1. Trouvez les 5 utilisateurs les plus proches d'un point donné (par exemple, le centre de Paris) dans un rayon de 5km.

```json
db.utilisateurs.find({
  "adresse.localisation": {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [2.3522, 48.8566]
      },
      $maxDistance: 5000
    }
  }
}).limit(5);
```
2. Trouvez les bibliothèques les plus proches d'un utilisateur spécifique

```json
db.bibliotheques.aggregate([
  {
    $geoNear: {
      near: {
        type: "Point",
        coordinates: db.utilisateurs.findOne(ObjectId('67c83686c19fb231442071e3')).adresse.localisation.coordinates
      },
      distanceField: "distance"
    }
  }
]);
```
3. Utilisez l'opérateur $geoNear dans un pipeline d'agrégation pour obtenir les bibliothèques triées par distance et calculer précisément cette distance (en km).

```json
db.bibliotheques.aggregate([
  {
    $geoNear: {
      near: {
        type: "Point",
        coordinates: db.utilisateurs.findOne(ObjectId('67c83686c19fb231442071e3')).adresse.localisation.coordinates
      },
      distanceField: "distance",
      distanceMultiplier: 0.001
    }
  }
]);
```


## Exercice 2.3

1. Utilisez $geoWithin pour trouver tous les utilisateurs à l'intérieur d'une zone définie par un polygone (par exemple, un quartier de Paris).

```json
db.utilisateurs.find({
  "adresse.localisation": {
    $geoWithin: {
      $geometry: {
        type: "Polygon",
        coordinates: [
          [
            [2.3522, 48.8566],
            [2.3522, 48.8600],
            [2.3560, 48.8600],
            [2.3560, 48.8566],
            [2.3522, 48.8566]
          ]
        ]
      }
    }
  }
});
```
2. Trouvez tous les utilisateurs qui se trouvent dans la zone de service d'une bibliothèque spécifique.

```json
db.utilisateurs.find({
  "adresse.localisation": {
    $geoWithin: {
      $centerSphere: [
        db.bibliotheques.findOne({ nom: "Bibliothèque Nationale de France" }).localisation.coordinates,
        0.5 / 6371
      ]
    }
  }
});
```
3. Créez une collection rues avec au moins une rue représentée comme un LineString GeoJSON, puis utilisez $geoIntersects pour trouver les bibliothèques dont la zone de service intersecte cette rue.

```json
db.rues.insertOne({
  nom: "Rue de la Paix",
  localisation: {
    type: "LineString",
    coordinates: [
      [2.3522, 48.8566],
      [2.3522, 48.8600]
    ]
  }
});
```

```json
db.bibliotheques.find({
  zone: {
    $geoIntersects: {
      $geometry: db.rues.findOne({ nom: "Rue de la Paix" }).localisation
    }
  }
});
```


## Exercice 2.4

1. Créez une collection livraisons pour suivre les livraisons de livres, avec :
• Des références vers les livres et les utilisateurs
• Un point de départ (bibliothèque) et un point d'arrivée (adresse utilisateur)
• Une position actuelle du livreur (Point GeoJSON)
• Un itinéraire planifié (LineString GeoJSON)

```json
db.livraisons.insertOne({
  livre: ObjectId('67c812bc496d77e175cc1280'),
  utilisateur: ObjectId('67c83686c19fb231442071e3'),
  depart: ObjectId('67c84d8cf0ecf4642768e308'),
  arrivee: db.utilisateurs.findOne(ObjectId('67c83686c19fb231442071e3')).adresse.localisation.coordinates,
  position: {
    type: "Point",
    coordinates: [2.3522, 48.8566]
  },
  itineraire: {
    type: "LineString",
    coordinates: [
      [2.3522, 48.8566],
       [2.3522, 48.8600]
    ]
  }
  });
```
2. Implémentez une fonction pour mettre à jour la position d'une livraison.

```json
db.livraisons.updateOne(
  { _id: ObjectId('67c86847f0ecf4642768e30b') },
  { $set: { position: { type: "Point", coordinates: [2.3522, 48.8600] } } }
);
```
3. Créez une requête pour trouver toutes les livraisons en cours dans un rayon de 1km autour d'un point donné.

```json
db.livraisons.createIndex({ "position": "2dsphere" });
```

```json
db.livraisons.find({
  position: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [2.3522, 48.8566]
      },
      $maxDistance: 1000
    }
  }
});
```


- Comment les fonctionnalités géospatiales peuvent-elles améliorer un service de bibliothèque ?

    elles penvent amélun service de bibliothèque en permettant de savoir ou sont chaque livre et de les livrer plus rapidement.

- Quels défis avez-vous rencontrés lors de l'implémentation des requêtes géospatiales ?

    les défis rencontrés sont la compréhension des requêtes géospatiales et la mise en place des index géospatiaux.

- Comment intégreriez-vous ces fonctionnalités dans une application web ou mobile?

    en utilisant des services de cartographie comme google maps pour afficher les données géospatiales et en utilisant des requêtes géospatiales pour récupérer les données nécessaires.