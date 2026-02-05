# TP1 Back-end - API Rest pour du ML

Ce projet contient une API rest pour l'évalutation de crédit bancaire selon plusieurs caractéristiques comme l'âge, le revenu, ... On se sert d'un modèle d'intelligence artficielle afin de faire la prédiction selon les données des utilisateurs. Afin d'avoir une API sécurisée, nous utilisons une authentification JWT afin de protégé les ressources (prédiction, ...). Nous stockons les données dans une base de données PostGreSQL local. 

---

## Architecture du code et API

```
design-rest-api/
|
├── app/                      # Code source principal
│   ├── __init__.py
│   ├── main.py               # Point d'entrée de FastAPI
│   ├── config.py             # Configuration
│   ├── database.py           # Modèles SQLAlchemy
│   ├── models.py             # Modèles Pydantic
│   ├── schemas.py            # Schémas de validation
│   ├── auth.py               # Authentification JWT (Tokens)
│   ├── crud.py               # Opérations sur la base de données PostgreSQL
│   ├── dependencies.py       # Dépendances FastAPI
│   ├── security.py           # Logique de sécurité
│   ├── predictor.py          # Logique de prédiction du modèle
|   |
│   └── routers/              # Endpoints organisés
│       ├── auth.py           # Routes d'authentification
│       ├── predictions.py    # Routes de prédiction
│       ├── admin.py          # Routes administrateur
│       └── model.py          # Routes du modèle ML
|
├── models/                   # Modèles ML et training
├── data/                     # Données pour l'apprentissage
├── tests/                    # Tests unitaires
├── postman/                  # Postman
├── docs/                     # Documentation de l'API
├── docker-compose.yml        # Configuration Docker
├── Dockerfile                # Image docker
├── requirements.txt          # Dépendances Python nécessaire pour le projet
├── test_api.sh               # Script de test
├── init_bd.py                # Initialisation base de données
└── README.md                 # Documentation de l'utilisation de l'API
```

```
│   Client Web    │    │   FastAPI        │    │  PostgreSQL     │
│                 │<-->│   + JWT Auth     │<-->│   Database      │
│                 │    │   + ML Model     │    │                 │

```

---

## Comment on utilise l'API ?

> Toutes les dépendances du requirement.txt se feront automatiquement lors de la création du containeur. Ainsi que la création de la base de données

### 1. Setup avec Docker Compose

```bash
# Lancer PostgreSQL + API
docker-compose up -d

# Initialiser la DB (dans le conteneur)
docker-compose exec api python init_db.py

# Voir les logs
docker-compose logs -f api 
```

### 2. Exemple d'authentification

```bash
# Inscription
curl -X POST http://localhost:8000/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@example.com",
    "username": "john",
    "password": "SecurePass123",
    "full_name": "John Doe"
  }'

# Login
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=john&password=SecurePass123"

# Récupérer le token et l'utiliser
TOKEN="..."  # Le token reçu du login

# Faire une prédiction (avec token)
curl -X POST http://localhost:8000/predictions/predict \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "age": 35,
    "income": 3200,
    "credit_amount": 15000,
    "duration": 48
  }'

# Tester l'historique des prédictions
curl -X GET http://localhost:8000/predictions/history \
  -H "Authorization: Bearer $TOKEN"
```

---

## Technos utilisées dans le projet

* **FastAPI** : Framework web Python pour créer des API rapides avec documentation automatique.

* **PostgreSQL** : Base de données relationnelle pour le stockage persistant des données.

* **JWT** : Standard de jetons sécurisés pour l'authentification des utilisateurs.

* **Pydantic** : Outil de validation de données et de définition de schémas.

* **Docker & Compose** : Conteneurisation pour simplifier le déploiement et l'exécution.

* **FLAML** : Bibliothèque d'AutoML pour optimiser le modèle de prédiction.

---

## Les différentes endpoints de l'API

###### Authentification & Gestion Utilisateur

* **`POST /auth/register`** : Création d'un nouveau utilisateur.

* **`POST /auth/login`** : Authentification et génération du jeton d'accès JWT.

* **`GET /auth/me`** : Récupération des informations du profil de l'utilisateur connecté.

###### Prédictions de Crédit

* **`POST /predictions/predict`** : Prédiction de crédit

* **`GET /predictions/history`** : Consultation de l'historique personnel des prédictions.

* **`GET /predictions/stats`** : Visualisation des indicateurs de performance des prédictions de l'utilisateur.

###### Administration

* **`GET /admin/users`** : Liste exhaustive des utilisateurs enregistrés.

* **`GET /admin/stats`** : Tableau de bord des statistiques globales d'utilisation du service.

###### Documentation Technique

* **`GET /docs`** : Interface interactive Swagger UI pour tester les endpoints en temps réel.

* **`GET /redoc`** : Documentation statique détaillée via l'interface ReDoc.

---

## Les paramètres de l'API

- **Port** : 8000 
  
  > Port configurable dans le code : ``app/main.py``
  
  

- **Base de données** : PostgreSQL sur le port 5432
  
  > Port configurable dans le code : ``app/config.py``
  
  

- **Expiration des tokens JWT** : 60min
  
  > Durée configurable dans le code : ``app/auth.py``
* **Algorithme JWT** : HS256
  
  > Algorithme choisi configurable dans le code : ``app/auth.py``

---

## Les commandes pour Docker

```bash
# Voir les logs
docker-compose logs -f api

# Accéder au conteneur API
docker-compose exec api bash

# Redémarrer les services
docker-compose restart

# Reconstruction complète
docker-compose down
docker-compose up -d --build
```

---

## Les principales fonctionnalités

###### Authentification

- Création d'un utilisateurs avec un nom / adresse mail unique

- Utilisation de Token

- gestion des possibles erreurs pendant les requêtes

- endpoints

###### Admin

- Lister tous les utilisateurs

- Différentes statistiques disponibles  

###### Modèle de Machine Learning

- Utilisation de FLAML

- Historique des prédictions

- Historique par utilisateurs de l'API 

###### Test

- Test unitaires
- Test authentifaction
- Test prédiction
- Test API

###### Utilisation

- Containerisation avec Docker

- Base de données ProstGreSQL

- Documentation de l'API 

---

## Vigilance

- Quand un token expire, on ne peut pas en créer un autre. L'utilisateur devra utilisé son mot de passe. Il faudrait pour bien faire avec un nouveau endpoint `/auth/refresh`

- Quand une modification dans le code, il faut totalement relancer le container (voir commande Docker). La base de données PostgreSQSL sera sauvegardé. 

---

## Failles et limites du projet

- Gestion de la `SECRET_KEY` : en effet, notre clé pour les Tokens est en dur dans notre code dans le fichier `app/config.py`. Pour l'instant, ce n'est pas vraiment un soucis mais si nous mettons en production, il est impératif de changer cet clé par une variable d'environnement que l'on mettra pas sur le github mais dans un `.env`
  
  

- Un utilisateur peut saturer l'endpoint de la prédictions `/predictions/predict` des milliers voir millions de fois sans qu'il est de limite. Cela va surcharger l'API et la faire crash dans le pire des cas. Empechant donc les autres utilisateurs l'utilisation du modèle et de l'API entière. (DDoS). Il faut donc limiter les utilisateurs de l'API en nombre de requête par seconde ou minute.
  
  

- Le code n'est pas vraiment commenté. Il n'est donc pas facile de lire et comprendre facilement l'objectif des différentes fonctions même si leurs noms sont assez explicites Cela peut poser des problèmes plus tard pour la maintenabilité. Il faudrait faire une donc faire une documentation sérieuse du code. Et ajouter une documentation complète dans le dossier `doc/`.
  
  

- Le modèle ML est chargé directement dans le processus de l'API. Si une prédiction fait planter Python ou consomme toute la RAM, l'API entière sera down. Il faut isoler le modèle du processus de l'API afin que si le modèle a un problème, toute l'API ne soit pas morte mais juste le endpoint de prédiction.
  
  

- L'API n'est pas très générique car si on modifie les paramètres d'entrée du ``predict/`, toutes les applications qui utilisent l'API s'arrêteront de fonctionner. Donc, l'API n'est pas évolutive sans casser l'existant. Il faudrait faire un versioning à l'aide de l'URL.

---



 Client → Login → JWT Token → API (vérifie token) → DB → Réponse

### Configuration DB

```python
# Structure de la base de données expliquée
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)     # Identifiant unique
    email = Column(String, unique=True)        # Un seul email par utilisateur
    username = Column(String, unique=True)     # Nom d'utilisateur unique
    hashed_password = Column(String)           # Mot de passe chiffré pour la sécurité
    is_admin = Column(Boolean, default=False)  # Droits administrateur

class Prediction(Base):
    __tablename__ = "predictions"
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.id"))  # Lien vers l'utilisateur
    # ... les autres champs de prédiction, tout ce qui est utile pour l'utilisateur
```

### Authentification JWT

**Concepts à expliquer :**

- **Qu'est-ce qu'un JWT ?** un jeton auto contenu avec une expiration (60min dans le code par exemple)
- **Pourquoi hasher les mots de passe ?** pour la sécurité avec la librairie python bcrypt
- **Comment vérifier un token ?** Middleware de FastAPI, en gros c'est un filtre qui intercepte les requêtes entre le client et l'application pour sécuriser chaque requête et réponse
  
  

Un token a 3 états : 

- Quand on s'authentifie avec un nouveau compte, un nouveau token est crée

- Quand on souhaite d'identifier, on vérifie le token

- Quand le token expire (60min)
  
  

**Code progressif :**

```python
# auth.py - Version simple d'abord
def create_access_token(username: str) -> str:
    return f"fake-token-for-{username}"

# Puis version réelle avec JWT
def create_access_token(
    data: dict,
    expires_delta: Optional[timedelta] = None
) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=15))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
```



---

## Exercices autonomes

**Endpoint de suppression de compte** 

```python
app.put("/me")
async def delete_account()
```

**Endpoint pour changer le mot de passe**

```python
@app.put("/auth/change-password")
async def change_password(old_password: str, new_password: str, ...)
```

**Limite de requêtes par jour (rate limiting)** 

```python
from slowapi import Limiter
@limiter.limit("100/day")
```

**Export de l'historique en CSV**

```python
@app.get("/predictions/export.csv")
async def export_predictions(current_user: User = Depends(...)):
```

**Dashboard admin pour visualiser les stats**

```python
# TODO
```
