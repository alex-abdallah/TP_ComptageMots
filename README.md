# TP — Comptage de mots avec Hadoop MapReduce et PySpark

**Auteur :** Alex Abdallah  
**Cours :** Fondements et Applications de Big Data

## Objectif

Implémenter et comparer quatre méthodes de comptage des occurrences
de mots dans un fichier :

1. Python séquentiel simple.
2. Pseudo-MapReduce en Python.
3. Hadoop MapReduce.
4. Apache Spark avec PySpark.

## Fichiers de test

Le fichier initial contient 1 000 lignes, avec un mot par ligne
et des répétitions.

Ce fichier est dupliqué m fois pour produire les tailles suivantes :

| Facteur m | Nombre de lignes |
|---:|---:|
| 1 | 1 000 |
| 10 | 10 000 |
| 100 | 100 000 |
| 1 000 | 1 000 000 |

Les mêmes fichiers sont utilisés pour les quatre méthodes.
Les données sont générées par le notebook.

## Contenu du dépôt

- `TP_ComptageMots.ipynb` : explications, code, tests, comparaison et conclusions.
- `compose.yaml` : configuration des services Hadoop.
- `config` : paramètres HDFS, YARN et MapReduce.
- `wordcount_spark.py` : programme de comptage avec PySpark.

## Environnement

- Windows et VS Code avec un noyau Python pour le notebook.
- Docker Desktop et Docker Compose.
- Hadoop 3.3.6, via l’image `apache/hadoop:3` utilisée lors des tests.
- Spark 3.5.3, via l’image `apache/spark:3.5.3-python3`.
- pandas pour les tableaux comparatifs.

Hadoop fonctionne dans quatre conteneurs sur un seul ordinateur.
Spark fonctionne en mode local avec deux threads de calcul (`local[2]`).

## Exécution

### 1. Préparer le notebook

Ouvrir `TP_ComptageMots.ipynb` dans VS Code avec les extensions
Python et Jupyter.

Exécuter les cellules de génération des fichiers, puis les cellules
Python simple et pseudo-MapReduce.

Si nécessaire, installer pandas dans une cellule :

```python
%pip install pandas
```

### 2. Démarrer Hadoop

Ouvrir un terminal dans le dossier du dépôt, puis exécuter :

```powershell
docker compose pull
docker compose up -d
```

Vérifier HDFS et YARN :

```powershell
docker compose exec -T namenode hdfs dfsadmin -report
docker compose exec -T resourcemanager yarn node -list
```

Un DataNode actif et un nœud YARN `RUNNING` doivent être disponibles.

Copier les fichiers générés dans HDFS :

```powershell
docker compose exec -T namenode hdfs dfs -mkdir -p /tp/input
docker compose exec -T namenode bash -lc 'hdfs dfs -put -f /data/mots_m*.txt /tp/input/'
```

Exécuter ensuite les cellules Hadoop du notebook.

### 3. Exécuter PySpark

Télécharger l’image :

```powershell
docker pull apache/spark:3.5.3-python3
```

Exécuter les cellules PySpark du notebook. Elles créent le programme
`wordcount_spark.py`, le lancent dans Docker et vérifient les comptes.

### 4. Comparer les résultats

Exécuter les cellules de comparaison pour afficher les temps des
quatre méthodes sur chaque taille de fichier.

### 5. Arrêter Hadoop

Après les tests :

```powershell
docker compose stop
```

## Résultats et limites

Les quatre méthodes produisent les mêmes comptes sur les fichiers testés.

Python simple est la méthode la plus rapide pour les tailles étudiées.
PySpark est plus rapide que Hadoop dans cette expérience.

Les chronométrages ne couvrent pas exactement les mêmes étapes :
Hadoop inclut le lancement de la commande et la soumission du job,
tandis que PySpark exclut le démarrage du conteneur et la création
de la session Spark.

Les tests sont réalisés sur un seul ordinateur, avec un fichier maximal
d’environ 7,1 Mo. Ils ne permettent pas de conclure sur les performances
d’un cluster de plusieurs machines ou sur des volumes beaucoup plus grands.

Les résultats détaillés et les conclusions figurent dans le notebook.