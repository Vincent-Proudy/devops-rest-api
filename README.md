# TP1 Back-end

## Instructions de déploiement

### 1. Setup local (sans Docker)

```bash
# Installer PostgreSQL
# Mac: brew install postgresql
# Ubuntu: sudo apt-get install postgresql

# Démarrer PostgreSQL
# Mac: brew services start postgresql
# Ubuntu: sudo service postgresql start

# Créer la base de données
createdb credit_scoring_db

# Installer les dépendances Python
pip install -r requirements.txt

# Initialiser la DB
python init_db.py

# Lancer l'API
uvicorn app.main:app --reload
```

### 2. Setup avec Docker Compose (recommandé)

```bash
# Lancer PostgreSQL + API
docker-compose up -d

# Initialiser la DB (dans le conteneur)
docker-compose exec api python init_db.py

# Voir les logs
docker-compose logs -f api
```

### 3. Tester l'authentification

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
TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJqb2huIiwiZXhwIjoxNzcwMzEzNzg1fQ.uHfc2fy7mWdPCgX-XUDVXmso_u1YWBzUlvWFQx-m8Ck"  # Le token reçu du login

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
```

---

## 📊 Déroulement du TD avec Auth + DB

### Phase 1 : Comprendre l'architecture (20 min)

**Vous expliquez (avec schéma) :**

   Client → Login → JWT Token → API (vérifie token) → DB → Réponse

   Client Web             FastAPI                    PostgreSQL  
   (Postman,     -->    + JWT Auth       -->   Database   
   Frontend...)           + ML Model             (Docker)      

 **Démo live :**

1. Inscription d'un utilisateur
2. Login et récupération du token
3. Utilisation du token pour faire une prédiction
4. Consultation de l'historique

### Phase 2 : Configuration DB (30 min)

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

**🛑 CHECKPOINT : "Lancez PostgreSQL et créez la DB"**

### Phase 3 : Authentification JWT (45 min)

**Concepts à expliquer :**

- **Qu'est-ce qu'un JWT ?** un jeton auto contenu avec une expiration (60min dans le code par exemple)
- **Pourquoi hasher les mots de passe ?** Sécurité avec notamment la librairie python bcrypt
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

### Phase 4 : Endpoints protégés (60 min)

**Montrer la magie des dépendances FastAPI :**

```python
@app.post("/predict")
async def predict(
    request: CreditRequest,
    current_user: User = Depends(get_current_active_user)  # ✨ Magie !
):
    # Si pas de token valide, FastAPI renvoie 401 automatiquement
    # Sinon, current_user contient l'utilisateur
```

**🛑 CHECKPOINT : "Testez /predict sans token → 401"**

### Phase 5 : Data Collection (40 min)

**Enregistrer les prédictions :**

```python
# Dans /predict, après la prédiction
db_prediction = create_prediction(
    db, user_id, age, income, ...
)
# "Maintenant on peut analyser toutes les requêtes !"
```

**Créer l'endpoint d'historique :**

```python
@app.get("/predictions/history")
async def history(current_user: User = Depends(...)):
    return get_user_predictions(db, current_user.id)
```

### Phase 6 : Tests avec Postman (45 min)

**Collection mise à jour :**  

1.Register User
2.Login (sauver le token en variable)
3.Get Current User (avec token)
4.Predict (avec token)
5.Get History (avec token)
6.Get Stats (avec token)

**Astuce Postman pour auto-token :**

```javascript
// Dans "Tests" du login
pm.environment.set("auth_token", pm.response.json().access_token);

// Puis dans les autres requêtes, Header:
// Authorization: Bearer {{auth_token}}
```

---

## 📝 Exercices autonomes suggérés

1. **Endpoint de suppression de compte**
2. **Endpoint pour changer le mot de passe**
3. **Limite de requêtes par jour (rate limiting)**
4. **Export de l'historique en CSV**
5. **Dashboard admin pour visualiser les stats**
