# 🍽️ TALN avec spaCy — Analyse linguistique d'avis de restaurants

**TP5 — POS tagging, entités nommées et analyse en dépendances** sur un corpus d'avis de restaurants marocains rédigés en français.
Le projet utilise **spaCy** et le modèle `fr_core_news_sm` pour explorer ce que le traitement automatique du langage sait (et ne sait pas) faire sur des textes courts, familiers et bruités.

> Cours de TALN — EMSI (version complète avec solutions et interprétations).

---

## ✨ Contenu

| Partie | Sujet | Ce qui est réalisé |
|---|---|---|
| **1** | **Étiquetage morpho-syntaxique (POS)** | Annotation token par token (lemme, catégorie UD, traits), fréquences des catégories, adjectifs les plus fréquents par classe (positif / négatif) |
| **2** | **Entités nommées (NER)** | Extraction des entités, encodage **BIO**, tableau d'entités, évaluation (précision / rappel) face à une référence annotée à la main |
| **3** | **Analyse en dépendances** | Arcs de dépendance, extraction « qui fait quoi » (sujet–verbe–objet), extraction de couples **aspect–opinion** (ex. *service → rapide*) |
| **Bonus A** | **EntityRuler** | Ajout d'un gazetteer de noms de restaurants pour corriger le NER |
| **Bonus B** | **Gestion de la négation** | Détection de « pas / jamais » pour distinguer *bon* de *pas bon* |

Chaque partie se termine par une **interprétation** rédigée, et le notebook comprend un tableau de **cinq erreurs typiques du modèle** avec explication et remède.

---

## 📁 Structure du dépôt

```
taln-spacy-avis-restaurants/
├── taln-spacy-avis-restaurants.ipynb   # Notebook complet (exercices, solutions, interprétations)
└── README.md
```

---

## 📊 Données

Le corpus est **fourni directement dans le notebook** (aucun fichier externe) :

- **100 avis annotés** (54 positifs / 46 négatifs) : 26 avis issus du TP3, 10 avis du cours et 64 nouveaux avis.
- **12 avis dédiés au NER**, avec une annotation de référence (*gold*) pour évaluer le modèle.

Les avis sont volontairement difficiles : majuscules expressives (« TRÈS lent »), allongements (« Trooop bien »),
fautes de frappe (« magnifque »), émojis, mentions `@`, URL, négations (« pas bon du tout ») et opinions mixtes
(« le service était lent mais les plats étaient bons »).

---

## ⚙️ Prérequis

- **Python 3.10+**
- `spacy`, `pandas`, `matplotlib`
- Modèle spaCy `fr_core_news_sm` (3.8.0)

### Installation

```bash
pip install spacy pandas matplotlib
python -m spacy download fr_core_news_sm
```

> Le notebook installe automatiquement ces dépendances ainsi que le modèle lors de l'exécution de la section 0.2.

---

## 🚀 Utilisation

```bash
jupyter notebook taln-spacy-avis-restaurants.ipynb
```

1. Renseigner la cellule **0.1 Identification** (nom, prénom, classe, groupe, site).
2. Exécuter toutes les cellules dans l'ordre (**Run All**).
3. Les cellules `✅ Vérification` affichent ✅ ou ❌ pour valider chaque fonction sans interrompre le notebook.

### Fonctions principales

| Fonction | Rôle |
|---|---|
| `annoter(doc)` | DataFrame `token / lemme / pos / traits` |
| `adjectifs(docs, etiquettes, classe)` | Compteur des adjectifs d'une classe donnée |
| `entites(doc)` | Liste de couples `(texte, type)` |
| `bio(doc)` | Encodage BIO (`Karim/B-PER Benali/I-PER`) |
| `extraire_entites(docs)` | DataFrame `avis / entité / type` |
| `arcs(doc)` | Triplets `(token, relation, tête)` |
| `qui_fait_quoi(doc)` | Triplets sujet–verbe–objet |
| `opinions(doc)` | Couples aspect–opinion (règles `amod` et copule) |
| `opinions_neg(doc)` | Idem, avec prise en compte de « pas » / « jamais » |

---

## 🔎 Résultats clés

### POS
- Catégories dominantes : **NOUN (263), PUNCT (199), ADJ (164)**, ce qui est cohérent avec des avis courts, expressifs et évaluatifs.
- Adjectifs positifs : *bon, rapide, délicieux, chaleureux, magnifique*. Adjectifs négatifs : *froid, lent, sale, élevé*.
- Le POS ignore la polarité : *bon* apparaît aussi dans des avis négatifs (« pas bon »).

### NER
| Configuration | Précision | Rappel |
|---|---|---|
| `fr_core_news_sm` seul | **0,36** | **0,36** |
| + `EntityRuler` (gazetteer) | **0,73** | **0,76** |

Le modèle, entraîné sur de la presse française, confond les types (restaurant → `LOC`/`MISC`, prénoms arabes → `LOC`),
se trompe sur les frontières (« serveur Youssef ») et produit de fausses entités sur les mots en majuscule en début de phrase (*Dîner*, *Brunch*).

### Dépendances
Couples aspect–opinion les plus fréquents : `(service, rapide)` ×14, `(service, lent)` ×6, `(tajine, délicieux)` ×4, `(rapport, bon)` ×4, `(cadre, magnifique)` ×3.
Ils relient chaque jugement à un aspect précis, ce que de simples collocations (bigrammes) ne permettent pas, même quand les mots sont éloignés.

### Négation
L'extraction avec négation détecte 6 couples négatifs : `(couscous, pas bon)`, `(plat, pas bon)`, `(personnel, pas aimable)`, `(poisson, pas frais)`, `(prix, pas élevé)`, `(légume, pas cher)`.
⚠️ « pas cher » et « pas élevé » sont en réalité **positifs** : une simple inversion ne suffit pas, il faut un lexique d'opinion qui gère la négation.

---

## ⚠️ Limites connues

- **Petit modèle** (`sm`) : erreurs de POS sur le langage familier (*salé* → ADV, *lent* → ADV, *tout* → NOUN).
- **Fautes de frappe** : lemmatisation par règles, donc mots mal lemmatisés (*magnifque* → *magnifqu*).
- **EntityRuler** : liste fermée, qui ne généralise pas, demande de la maintenance et peut créer des homonymies (*Le Palmier* / un palmier). Les scores du bonus sont par ailleurs **optimistes**, car les règles sont testées sur le jeu d'évaluation lui-même.
- Les valeurs chiffrées peuvent varier légèrement selon la version du modèle.

---

## 🔧 Pistes d'amélioration

- Utiliser un modèle plus grand (`md`, `lg`, `trf`).
- Ajouter un **correcteur orthographique** avant l'analyse.
- **Ré-entraîner ou affiner** le NER sur des avis annotés marocains.
- Construire un **lexique d'opinion** pour passer des couples aspect–opinion à une polarité fiable.
- Évaluer l'EntityRuler sur un jeu de test distinct du jeu de règles.

---

## 🛠️ Stack technique

`Python` · `spaCy 3.8` · `fr_core_news_sm 3.8.0` · `pandas` · `matplotlib` · `Jupyter`
