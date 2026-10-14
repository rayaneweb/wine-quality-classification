# Projet Vin - Classification de la qualite

Projet de machine learning visant a predire si un vin est considere comme
**bon** (`quality >= 7`) a partir de mesures physicochimiques.

Le projet utilise les jeux de donnees *Wine Quality* de Cortez et al. sur les
vins rouges et blancs de type vinho verde. Les donnees sont nettoyees,
analysees, puis utilisees pour comparer plusieurs modeles de classification.

## Contenu du projet

```text
.
|-- data/
|   |-- winequality-red.csv
|   `-- winequality-white.csv
|-- notebooks/
|   |-- projet_vin_ml_simple.ipynb
|   `-- model_vin.joblib
|-- rapport/
|-- requirements.txt
`-- README.md
```

## Objectif

La variable cible est binaire :

- `1` : bon vin, lorsque `quality >= 7`
- `0` : vin non classe comme bon, lorsque `quality < 7`

Le projet met l'accent sur les metriques adaptees a une classe desequilibree :
precision, rappel, F1-score et AUC, plutot que sur l'accuracy seule.

## Donnees

Les deux fichiers CSV contiennent :

- 1599 vins rouges
- 4898 vins blancs
- 11 mesures physicochimiques
- une note de qualite `quality`

Variables utilisees :

- `fixed acidity`
- `volatile acidity`
- `citric acid`
- `residual sugar`
- `chlorides`
- `free sulfur dioxide`
- `total sulfur dioxide`
- `density`
- `pH`
- `sulphates`
- `alcohol`
- `type` ajoute dans le notebook (`0` = rouge, `1` = blanc)

## Methode

Le notebook suit les etapes suivantes :

1. Chargement et fusion des donnees rouge/blanc.
2. Verification des valeurs manquantes et des doublons.
3. Suppression des doublons.
4. Creation de la cible binaire `good`.
5. Analyse exploratoire et visualisations.
6. Decoupage train/test stratifie.
7. Selection de variables avec `SelectKBest`, RFE et importance Random Forest.
8. Comparaison de plusieurs modeles :
   - regression logistique
   - k plus proches voisins
   - arbre de decision
   - Naive Bayes
   - SVM
   - Random Forest
9. Optimisation par Grid Search et Random Search.
10. Ajustement du seuil de decision.
11. Evaluation finale sur le jeu de test.
12. Interpretation des variables importantes.
13. Sauvegarde du modele avec `joblib`.

## Installation

Creer puis activer un environnement virtuel :

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Installer les dependances :

```powershell
pip install -r requirements.txt
```

## Lancer le notebook

```powershell
jupyter notebook notebooks/projet_vin_ml_simple.ipynb
```

Executer les cellules dans l'ordre pour reproduire l'analyse, entrainer les
modeles et generer le fichier `model_vin.joblib`.

## Utiliser le modele sauvegarde

Le modele sauvegarde contient :

- `model` : pipeline scikit-learn retenu
- `features` : liste des variables attendues
- `threshold` : seuil de decision optimise
- `model_name` : nom du modele retenu

Exemple :

```python
import joblib
import pandas as pd

artefact = joblib.load("notebooks/model_vin.joblib")

vin = pd.DataFrame([{
    "fixed acidity": 7.0,
    "volatile acidity": 0.27,
    "citric acid": 0.36,
    "residual sugar": 20.7,
    "chlorides": 0.045,
    "free sulfur dioxide": 45.0,
    "total sulfur dioxide": 170.0,
    "density": 1.001,
    "pH": 3.0,
    "sulphates": 0.45,
    "alcohol": 8.8,
    "type": 1
}])

modele = artefact["model"]
features = artefact["features"]
threshold = artefact["threshold"]

if hasattr(modele, "predict_proba"):
    score = modele.predict_proba(vin[features])[:, 1][0]
else:
    score = modele.decision_function(vin[features])[0]

prediction = "bon vin" if score >= threshold else "pas bon"
print(score, prediction)
```

## Resultats et interpretation

Le notebook montre que :

- les classes sont desequilibrees, avec environ 19 % de bons vins apres
  nettoyage ;
- l'accuracy seule est trompeuse, car un modele qui predit toujours "pas bon"
  obtient deja un score eleve ;
- l'alcool est la variable la plus liee a la qualite ;
- la densite et l'acidite volatile ont plutot un effet defavorable ;
- les performances doivent etre lues avec le F1-score, le rappel, la precision
  et l'AUC.

## Source des donnees

P. Cortez, A. Cerdeira, F. Almeida, T. Matos et J. Reis,
*Modeling wine preferences by data mining from physicochemical properties*,
Decision Support Systems, 2009.
