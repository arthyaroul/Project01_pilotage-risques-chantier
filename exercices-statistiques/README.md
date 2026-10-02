# Exercices de statistiques appliquées

Exercices réalisés pendant le bootcamp *Data Essentials* (Jedha, 2026), sur des cas inspirés d'entreprises réelles.

| Notebook | Contenu |
|---|---|
| [`01_intervalles_de_confiance.ipynb`](01_intervalles_de_confiance.ipynb) | Intervalles de confiance d'une moyenne (loi de Student) et d'une proportion, calcul de taille d'échantillon |
| [`02_tests_AB.ipynb`](02_tests_AB.ipynb) | Tests d'hypothèses unilatéraux : test z sur une proportion, test t apparié et test de Welch, test A/B entre deux versions d'une fonctionnalité |

**Cas traités :** Facebook, Google Ads, Nintendo, Apple (intervalles de confiance) · Qonto, Swile, Vinted (tests d'hypothèses).

Chaque exercice suit la même structure : question posée, hypothèses, calcul (manuel puis vérifié avec une bibliothèque), décision et conclusion rédigée.

## Lancer les notebooks

Les jeux de données ont été fournis par le bootcamp et ne sont pas inclus dans le dépôt. Pour relancer les notebooks, placer les fichiers dans un dossier `data/`, ou sur Google Colab, les téléverser et remplacer `DATA_DIR` par `"/content/"`.

```bash
pip install -r requirements.txt
```

## Outils

Python · pandas · NumPy · SciPy · statsmodels · Matplotlib

**Arthy Aroul** · [LinkedIn](https://www.linkedin.com/in/arthy-aroul)
