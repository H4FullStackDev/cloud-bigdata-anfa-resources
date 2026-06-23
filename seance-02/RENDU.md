# RENDU - Séance 02 : Docker Compose, PySpark et Jupyter

## Informations de l'étudiant

* **Nom et prénom :** HODIA Dimitri
* **Identifiant GitHub :** [H4FullStackDev]

---

# Objectif de la séance

Cette séance avait pour objectif de découvrir l'utilisation de Docker Compose afin d'orchestrer plusieurs conteneurs au sein d'une même infrastructure. Nous avons également utilisé PySpark pour l'analyse de données, MinIO comme stockage objet compatible S3 et Jupyter Notebook pour l'exploration des données.

---

# Travaux réalisés

## 1. Création d'une image Docker personnalisée

Création d'un Dockerfile permettant de :

* Utiliser Python 3.11 comme image de base.
* Installer Java 17 nécessaire au fonctionnement de Spark.
* Installer les dépendances Python via requirements.txt.
* Copier le code source de l'application.
* Exécuter automatiquement le script d'analyse PySpark.

Construction de l'image :

```bash
docker build -t anfa-analyse:v1 .
```

---

## 2. Exécution du script d'analyse

L'image Docker a été exécutée avec un montage du dossier contenant les fichiers CSV du référentiel.

Commande utilisée :

```bash
docker run --rm -v "%cd%\..\data\referentiel:/data/referentiel:ro" anfa-analyse:v1
```

Résultats obtenus :

* Nombre de lignes de bus : 12
* Nombre d'arrêts uniques : 60
* Nombre total de bus : 100
* Nombre de bus actifs : 93
* Capacité totale de la flotte : 4538 places

Analyse réalisée avec succès à l'aide de PySpark.

---

## 3. Utilisation du cache Docker

Une seconde exécution de la commande :

```bash
docker build -t anfa-analyse:v1 .
```

a permis d'observer l'utilisation du cache Docker.

Les couches correspondant à :

* l'installation de Java,
* l'installation des dépendances Python,
* la configuration du répertoire de travail,

ont été réutilisées grâce au mécanisme de cache.

Cela permet de réduire considérablement le temps de construction des images.

---

## 4. Mise en place de Docker Compose

Création d'une architecture composée de trois services :

### MinIO

Stockage objet compatible Amazon S3.

Ports exposés :

* 9000 : API S3
* 9001 : Console d'administration

### Jupyter Notebook

Environnement d'exploration et d'analyse des données.

Port exposé :

* 8888

### anfa-app

Application d'analyse PySpark exécutée dans un conteneur dédié.

Démarrage de l'infrastructure :

```bash
docker compose up -d --build
```

Vérification :

```bash
docker compose ps
```

Résultat :

* anfa-minio : Healthy
* anfa-jupyter : Healthy
* anfa-app : Exited (0)

---

## 5. Exploration des données avec Jupyter

Création du notebook :

```text
exploration_minio.ipynb
```

Installation de boto3 :

```python
%pip install boto3==1.34.0
```

Connexion à MinIO :

```python
s3 = boto3.client(
    "s3",
    endpoint_url="http://minio:9000",
    aws_access_key_id="anfa-app-key",
    aws_secret_access_key="anfa-app-secret-2026",
    region_name="us-east-1",
)
```

Vérification du bucket :

```python
s3.list_buckets()
```

Lecture d'un fichier CSV depuis MinIO avec boto3 puis chargement dans un DataFrame pandas.

---

# Ce que j'ai retenu

Au cours de cette séance, j'ai appris :

* La différence entre une image Docker et un conteneur.
* Le rôle d'un Dockerfile dans la construction d'une image.
* Le fonctionnement du cache Docker et son impact sur les performances.
* L'utilisation de Docker Compose pour orchestrer plusieurs services.
* La communication entre conteneurs via le réseau interne Docker.
* L'utilisation de MinIO comme alternative locale à Amazon S3.
* L'utilisation de boto3 pour accéder à un stockage compatible S3.
* L'utilisation de PySpark pour effectuer des traitements analytiques sur des données.
* L'utilisation de Jupyter Notebook pour l'exploration et l'analyse interactive des données.

---

# Difficultés rencontrées

### Conflit de conteneur MinIO

Lors du lancement de Docker Compose, une erreur est apparue :

```text
The container name "/anfa-minio" is already in use
```

Cause :

Un conteneur MinIO issu du TP précédent existait déjà.

Solution :

```bash
docker stop anfa-minio
docker rm anfa-minio
```

puis relancement de :

```bash
docker compose up -d --build
```

### Perte du bucket MinIO

Après la création d'un nouveau volume Docker Compose, le bucket `anfa-raw` n'était plus disponible.

Solution :

* Recréation du bucket.
* Recréation des clés d'accès.
* Réexécution du script d'upload des fichiers CSV.

---

# Conclusion

Cette séance m'a permis de comprendre comment construire une image Docker, orchestrer plusieurs services avec Docker Compose et mettre en place une architecture de traitement de données reposant sur MinIO, PySpark et Jupyter Notebook. J'ai également découvert l'utilisation du cache Docker et les mécanismes de communication entre conteneurs au sein d'un même réseau Docker.
