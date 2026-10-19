# TP3 - DVC : versionner les donnees et rendre le pipeline reproductible

Projet de detection de fraude bancaire (suite du TP2). Git versionne le code et les
tickets `.dvc`, DVC versionne donnees/modele/rapports, MLflow garde les runs.

## Reproduire
```
python -m venv .venv && .venv\Scripts\Activate.ps1
pip install -r requirements.txt
dvc repro            # rejoue uniquement les etapes modifiees
```
Le stockage DVC (`../dvc-stockage`) est un dossier local : `dvc pull` ne marche donc
que sur la machine qui le contient. Pour partager les donnees, remplacer le remote par
un stockage accessible (S3, Azure, Google Drive...) : `dvc remote modify stockage url ...`.

## Versions
- `v1` : 5000 transactions, F1 = 0.8636
- `v2` : 7000 transactions, F1 = 0.8423
Navigation : `git checkout v1 && dvc checkout`, puis `git checkout main && dvc checkout`.
