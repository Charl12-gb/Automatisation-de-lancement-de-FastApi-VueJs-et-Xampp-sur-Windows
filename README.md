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
   setlocal enabledelayedexpansion

   :: Définition des chemins
   set "logFile=C:\xampp\htdocs\LeConcept\server_logs.txt"
   set "envFile=C:\xampp\htdocs\LeConcept\concept_front\.env"
   
   :: Enregistrer l'heure et la date du script
   echo [%date% %time%] Démarrage du script >> "%logFile%"
   
   :: Vérifier si l'utilisateur est connecté à un réseau
   set "network_status=offline"
   for /f "tokens=2 delims=:" %%a in ('ipconfig ^| findstr /C:"IPv4"') do (
       set ip=%%a
       set ip=!ip:~1!
       set network_status=online
   )
   
   :: Définir l'adresse IP à utiliser
   if "%network_status%"=="online" (
       set "access_ip=%ip%"
       echo [%date% %time%] Adresse IP locale détectée : %access_ip% >> "%logFile%"
   ) else (
       set "access_ip=127.0.0.1"
       echo [%date% %time%] Pas de connexion réseau détectée, utilisation de 127.0.0.1 >> "%logFile%"
   )
   
   :: Lire l'ancienne IP depuis le fichier .env
   set old_ip=
   for /f "tokens=2 delims==" %%a in ('findstr "VUE_APP_BASE_URL" "%envFile%"') do set old_ip=%%a
   
   :: Nettoyer l'ancienne IP (supprimer http:// et espaces)
   set old_ip=%old_ip:http://=%
   set old_ip=%old_ip: =%
   
   :: Comparer les IPs et mettre à jour si nécessaire
   if "%access_ip%"=="%old_ip%" (
       echo [%date% %time%] L'adresse IP n'a pas changé : %access_ip% >> "%logFile%"
   ) else (
       echo [%date% %time%] Mise à jour du fichier .env avec l'adresse : %access_ip% >> "%logFile%"
       (
           echo VUE_APP_BASE_API_URL=http://%access_ip%:8000/api/
           echo VUE_APP_BASE_URL=http://%access_ip%
           echo VUE_APP_FILE_URL=http://%access_ip%:8000/
       ) > "%envFile%"
   )
   
   :: Arrêter les processus existants pour éviter les conflits de port
   echo [%date% %time%] Arrêt des processus FastAPI et Vue.js en cours... >> "%logFile%"
   taskkill /F /IM python.exe /T 2>nul
   taskkill /F /IM node.exe /T 2>nul
   timeout /t 3
   echo [%date% %time%] Processus arrêtés. >> "%logFile%"
   
   :: Afficher l'adresse IP pour l'utilisateur avec une boîte de dialogue
   echo Set objShell = CreateObject("WScript.Shell") > msgbox.vbs
   echo objShell.Popup "Pour vous connecter à l'application :`nhttp://%access_ip%:8080`nSi vous êtes hors réseau, utilisez :`nhttp://127.0.0.1:8080", 10, "Information", 64 >> msgbox.vbs
   start /min msgbox.vbs
   
   :: Afficher aussi dans la console
   echo.
   echo *************************************************
   echo Pour vous connecter à l'application :
   echo http://%access_ip%:8080
   echo Si vous êtes hors réseau, utilisez :
   echo http://127.0.0.1:8080
   echo *************************************************
   echo.
   
   :: Démarrer XAMPP
   echo [%date% %time%] Démarrage de XAMPP... >> "%logFile%"
   start "" "C:\xampp\xampp_start.exe"
   timeout /t 5
   
   :: Activer l'environnement virtuel Python
   cd /d C:\xampp\htdocs\LeConcept\concept_api
   :: call venv\Scripts\activate  || echo "Pas d'environnement virtuel"
   :: echo [%date% %time%] Environnement Python activé >> "%logFile%"
   
   :: Lancer FastAPI
   start uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
   echo [%date% %time%] FastAPI lancé sur http://%access_ip%:8000 >> "%logFile%"
   
   :: Lancer Vue.js
   cd /d C:\xampp\htdocs\LeConcept\concept_front
   start npm run serve -- --host 0.0.0.0 --port 8080
   echo [%date% %time%] Vue.js lancé sur http://%access_ip%:8080 >> "%logFile%"
   
   :: Vérifier si l'utilisateur est connecté à un réseau
   set "network_status=offline"
   for /f "tokens=2 delims=:" %%a in ('ipconfig ^| findstr /C:"IPv4"') do (
       set ip=%%a
       set ip=!ip:~1!
       set network_status=online
   )
   
   :: Si connecté, utiliser l'adresse IP locale, sinon utiliser 127.0.0.1
   if "%network_status%"=="online" (
       set "access_ip=%ip%"
   ) else (
       set "access_ip=127.0.0.1"
   )
   timeout /t 5
   
   :: Ouvrir l'URL dans le navigateur selon l'adresse IP
   start http://%access_ip%:8080
   
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
