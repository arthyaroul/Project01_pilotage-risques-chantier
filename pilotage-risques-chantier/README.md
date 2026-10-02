# Pilotage des risques de chantier

**Peut-on prédire le niveau de risque d'un chantier BTP à partir de ses indicateurs opérationnels et environnementaux ?**

Projet de fin de bootcamp *Data Essentials* (Jedha, août – septembre 2026), réalisé en binôme par **Arthy Aroul** et **Aurélie Mahé**.

---

## En bref

| | |
|---|---|
| **Question** | Identifier en amont les chantiers les plus à risque pour prioriser les audits de sécurité |
| **Données** | 1 000 projets de génie civil, 28 variables (coûts, délais, météo, ressources, sécurité) |
| **Démarche** | Exploration, détection d'une fuite de données, reformulation de la cible, 3 modèles comparés à une baseline naïve |
| **Résultat** | Aucun modèle ne fait mieux que le hasard (MCC ≈ 0) : le dataset, synthétique, ne contient pas de signal exploitable |
| **Apport du projet** | Évaluer un modèle honnêtement, plutôt que présenter un F1-score flatteur comme une réussite |

---

## Contexte

Sur un chantier, les risques sont multiples (accidents, dérives de budget et de planning, conditions climatiques) et les ressources d'audit sont limitées. Le suivi reste souvent manuel et réactif.

L'objectif était de construire un outil qui aide à **identifier en amont les chantiers les plus à risque**, pour prioriser les audits et les interventions. Les utilisateurs visés : conducteurs de travaux, maîtres d'ouvrage, responsables HSE et assureurs.

## Données

- **Source :** BIM-AI Integrated Dataset, Kaggle
https://www.kaggle.com/datasets/ziya07/bim-ai-integrated-dataset
- **Contenu :** 1 000 projets (tunnels, barrages, ponts, routes, bâtiments) répartis dans 5 villes américaines, décrits par 28 variables : coûts et durées prévus et réels, vibrations, fissures, capacité portante, météo, qualité de l'air, consommation d'énergie, matériaux, heures de travail, accidents, score et niveau de risque.
- **Cible :** `Risk_Level` (Low, Medium, High).

Un premier dataset (*Building Performance Dataset*) avait été écarté après exploration : entièrement synthétique, avec des distributions trop uniformes. Le dataset retenu s'est révélé présenter le même problème de fond.

Le fichier n'est pas inclus dans le dépôt : voir [`data/README.md`](data/README.md).

## Démarche

### 1. Exploration des données · [`01_EDA.ipynb`](notebooks/01_EDA.ipynb)

- Données complètes et cohérentes : aucune valeur manquante, aucun doublon.
- Mais plusieurs signes de données **générées aléatoirement** : catégories quasi parfaitement équilibrées, distributions uniformes, absence de liens attendus dans la réalité (un barrage ne consomme pas plus d'énergie qu'une route).
- Corrélation maximale entre une variable et le score de risque : **0,07**, soit quasi nulle.

### 2. Détection d'une fuite de données

`Risk_Level` est directement dérivé de `Safety_Risk_Score`. Garder ce score parmi les variables explicatives reviendrait à donner la réponse au modèle. Il a été retiré, ainsi que deux variables qui en dupliquent l'information (`Crack_Width`, `Image_Analysis_Score`).

### 3. Reformulation de la cible

Les trois classes d'origine étaient déséquilibrées (50 % High, 34 % Medium, 16 % Low). La question a été reformulée en décision binaire, plus actionnable : *ce chantier doit-il être audité en priorité ?* La classe Medium a été répartie entre Low et High selon la médiane de son score, ce qui donne 67 % High et 33 % Low.

### 4. Modélisation · [`02_arbre_de_decision.ipynb`](notebooks/02_arbre_de_decision.ipynb) · [`03_random_forest_et_comparaison.ipynb`](notebooks/03_random_forest_et_comparaison.ipynb)

Trois modèles de classification, du plus simple au plus robuste : régression logistique, arbre de décision et random forest. Le déséquilibre des classes a été traité (SMOTE, puis pondération des classes) et les hyperparamètres optimisés par GridSearchCV.

### 5. Évaluation

- **F1 macro** : équilibre entre precision et recall sur les deux classes.
- **MCC** (coefficient de corrélation de Matthews) : métrique de référence, qui vaut 0 pour une prédiction au hasard et n'est pas trompée par le déséquilibre des classes.
- **Baseline naïve** : prédire « High » pour tous les chantiers. Un modèle utile doit faire mieux.

## Résultats

| Modèle | F1 macro | MCC | Accuracy |
|---|:---:|:---:|:---:|
| Baseline naïve (toujours High) | 0,40 | 0,00 | 67,5 % |
| Régression logistique | 0,41 | −0,06 | 65 % |
| Arbre de décision | 0,44 | −0,03 | 63 % |
| Random forest optimisée | 0,56 | 0,00 | 65 % |

*Résultats sur le jeu de test (200 projets), tels que présentés lors du Demo Day. Les notebooks révisés utilisent une validation croisée à 5 plis : les valeurs peuvent légèrement varier, la conclusion reste la même.*

Au fil des itérations, le F1 macro est passé de 0,29 à 0,56 :

| Itération | F1 macro |
|---|:---:|
| Modèle initial, 3 classes | 0,29 |
| Passage à 2 classes | 0,42 |
| Ajout de SMOTE | 0,46 |
| Optimisation par GridSearchCV | 0,51 |
| Pondération des classes (class_weight) | 0,56 |

**Mais le MCC est resté proche de 0 à chaque étape**, et aucun modèle ne dépasse l'accuracy de la baseline naïve. La hausse du F1 venait des réglages de rééquilibrage, qui déplacent les prédictions entre les classes, et non d'une meilleure compréhension du risque. Une lecture fondée sur le seul F1 aurait conduit à une conclusion fausse.

## Enseignements

- **Changer la matière plutôt que complexifier le modèle.** Aucun algorithme ne peut apprendre une relation qui n'existe pas dans les données.
- **Toujours comparer à une baseline naïve.** C'est le moyen le plus simple de savoir si un modèle apporte réellement quelque chose.
- **Choisir des métriques adaptées au déséquilibre des classes.** L'accuracy et même le F1 peuvent masquer un modèle qui ne fait pas mieux que le hasard.
- **Penser au moment de la prédiction.** Pour prédire le risque en amont, les coûts et durées réels ne sont pas encore connus : sur des données réelles, il faudrait les exclure pour éviter une fuite temporelle.

## Pistes

- Travailler sur des données réelles : historique de chantiers d'une entreprise, bases d'accidents du travail, suivis de chantier.
- Reformuler en régression sur le score de risque, plutôt qu'en classification.
- Approfondir la question métier : *quel coût sommes-nous prêts à accepter pour réduire un risque d'accident grave ?* La réponse orienterait le compromis entre recall (ne manquer aucun chantier dangereux) et precision (éviter les audits inutiles).

## Structure du dépôt

```
pilotage-risques-chantier/
├── README.md
├── requirements.txt
├── data/
│   └── README.md                               # comment obtenir le dataset
├── notebooks/
│   ├── 01_EDA.ipynb                            # exploration des données
│   ├── 02_arbre_de_decision.ipynb              # premier modèle interprétable
│   └── 03_random_forest_et_comparaison.ipynb   # random forest, GridSearch, comparaison finale
└── presentation/
    └── presentation_pilotage_risques_chantier.pdf
```

## Reproduire l'analyse

```bash
git clone https://github.com/arthyaroul/pilotage-risques-chantier.git
cd pilotage-risques-chantier
pip install -r requirements.txt
```

Télécharger le dataset dans `data/` (voir [`data/README.md`](data/README.md)), puis lancer les notebooks dans l'ordre.

Sur **Google Colab** : ouvrir un notebook, téléverser le CSV, puis remplacer `DATA_PATH` par `"/content/bim_ai_civil_engineering_dataset.csv"`.

## Outils

Python · pandas · NumPy · scikit-learn · Matplotlib · Seaborn · Google Colab · Google Slides

## Auteures

**Arthy Aroul** · [LinkedIn](https://www.linkedin.com/in/arthy-aroul) · Profil hybride architecture et data, en formation Data Scientist, spécialisée sur les données du bâtiment et de l'ESG.

**Aurélie Mahé**

Projet réalisé dans le cadre du bootcamp *Data Essentials* de Jedha.
