Tu peux configurer ton environnement sous Windows pour que tes serveurs FastAPI et Vue.js démarrent automatiquement avec ton PC et se relancent en cas d'arrêt. Voici les étapes détaillées :

---

## 🔹 1. **Utiliser le Planificateur de tâches Windows**
Le Planificateur de tâches peut lancer des scripts au démarrage de ton PC.

### Étapes :
1. **Créer un script de démarrage**  
   Crée un fichier `start_servers.bat` dans un dossier facile d'accès (ex. `C:\scripts`).

   **Exemple de script (`start_servers.bat`) :**
   ```bat
   @echo off
   echo Démarrage des serveurs...

   :: Démarrer XAMPP (Apache & MySQL)
   start "" "C:\xampp\xampp_start.exe"

   :: Attendre un peu pour s'assurer que XAMPP est lancé
   timeout /t 5

   :: Activer un environnement virtuel Python si tu en utilises un
   cd /d C:\chemin\vers\ton\projet\backend
   call venv\Scripts\activate  || echo "Pas d'environnement virtuel"

   :: Lancer FastAPI avec Uvicorn
   start uvicorn main:app --host 0.0.0.0 --port 8000 --reload

   :: Lancer Vue.js
   cd /d C:\chemin\vers\ton\projet\frontend
   start npm run dev

   exit
   ```

2. **Créer une tâche planifiée :**
   - Ouvre **le Planificateur de tâches** (`taskschd.msc`).
   - Clique sur **Créer une tâche** (pas "Créer une tâche de base").
   - Onglet **Général** :
     - Nom : `Lancer FastAPI et Vue.js`
     - Exécuter avec les **autorisation maximales**.
   - Onglet **Déclencheurs** :
     - Nouveau ➝ Débuter **au démarrage** du PC.
   - Onglet **Actions** :
     - Nouveau ➝ **Démarrer un programme** ➝ Sélectionne `start_servers.bat`.
   - Onglet **Conditions** :
     - Désactive **"Démarrer la tâche uniquement si l’ordinateur est sur secteur"** (si c'est un PC portable).
   - Onglet **Paramètres** :
     - Coche **"Redémarrer la tâche en cas d'échec"** toutes les 5 minutes, jusqu'à 3 tentatives.

   ➜ **Valide et teste en redémarrant ton PC.**
