# Orchestrateur ou integration continue ? (TP06)

Environnement : Python 3.12, Prefect 3.9.2, DVC 3.67.1 (pas de conflit mlflow / prefect).

| Critere                           | Prefect (orchestrateur) | GitHub Actions (CI) |
|-----------------------------------|-------------------------|---------------------|
| Ce qui declenche l'execution      | une planification (cron `0 6 * * 1`, lundi 6 h), un Quick run dans l'interface, ou `python flow_mlops.py` | chaque `git push` (et le bouton Run workflow, `workflow_dispatch`) |
| Ou s'execute le traitement        | sur notre poste : le processus `serve` (planifier.py) ou un worker du work pool `pool-tp`, avec les vraies donnees DVC | sur une machine Linux de GitHub (ubuntu-latest), qui regenere les donnees v2 avec `donnees.py` et `nouvelles_donnees.py` |
| Qui surveille, et ou              | l'interface Prefect (http://127.0.0.1:4200) : runs, etat de chaque tache, durees, journaux (sortie de `dvc repro`) | l'onglet Actions du depot : chaque etape, son journal, et une croix rouge ou une coche verte sur le commit |
| Ce qui se passe en cas d'echec    | la tache `valider` passe en Failed, le flow run est Failed et `notifier` n'est pas executee ; `rejouer_pipeline` est reessayee 2 fois avant d'echouer | l'etape « Verifier la regle de qualite » sort avec le code 1, le job est rouge et le commit est marque en echec ; on corrige avec `git revert` |
| Type de tache adapte              | traitements recurrents ou longs sur les donnees de production : reentrainement hebdomadaire, surveillance | verification rapide de chaque modification du code avant qu'elle n'arrive en production |
| Ce qui est versionne dans Git     | `flow_mlops.py` et `planifier.py` (le workflow et sa planification) ; le deploiement `hebdomadaire-pool` est enregistre sur le serveur (pas de `prefect.yaml` cree) | `.github/workflows/ci.yml`, `verifier_qualite.py`, `requirements-ci.txt` |

## Observations

- Run normal : F1 0.8423 >= 0.82, flow Completed, notification envoyee.
- Run degrade (`max_depth: 2`) : F1 0.25 < 0.82, tache `valider` en Failed, `notifier` non executee.
- Quick run sans `serve` ni worker : le run reste en Scheduled (personne ne l'execute) ; il a ete supprime.
- Worker sur `pool-tp` : le Quick run de `hebdomadaire-pool` a ete execute par le worker, etat Completed.
- GitHub Actions : run vert (commit 243a46f), run rouge a l'etape de qualite (7c94516, `max_depth: 2`), run vert apres `git revert` (8d50319).

## Conclusion (5 lignes)

1. GitHub Actions verifie le code a chaque modification : un commit qui degrade le modele est signale en rouge en quelques minutes.
2. Prefect execute et surveille un traitement recurrent (le reentrainement du lundi) sur les vraies donnees, avec un etat et des journaux par tache.
3. La meme regle de qualite (`seuil_f1` de params.yaml) est appliquee des deux cotes : elle arrete le workflow comme elle bloque le commit.
4. La CI ne voit pas le stockage DVC local et regenere les donnees ; Prefect, lui, tourne la ou sont les donnees.
5. Les deux sont complementaires : la CI protege le code avant la production, l'orchestrateur fait tourner la production.
