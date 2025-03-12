Pour automatiser le démarrage de FastAPI, Vue.js et XAMPP sur Windows et assurer leur relance en cas d'arrêt, voici les étapes à suivre :

1. Configurer XAMPP pour démarrer au lancement

Ouvrez le Panneau de configuration XAMPP.

Cochez Apache et MySQL dans la colonne "Auto Start" si vous utilisez MySQL.

Allez dans Config > Service and Port Settings, assurez-vous que les services sont bien configurés.

Activez XAMPP en tant que service en allant dans Config > Autostart of modules.


2. Créer un fichier batch pour lancer FastAPI et Vue.js

Créez un fichier .bat pour démarrer FastAPI et Vue.js automatiquement :

📜 Script start_servers.bat

```
@echo off
:: Démarrer XAMPP (si ce n'est pas encore fait)
start /B "" "C:\xampp\xampp_start.exe"

:: Attendre quelques secondes pour s'assurer que MySQL et Apache sont bien lancés
timeout /t 5

:: Activer l'environnement Python et lancer FastAPI
cd /d C:\chemin\vers\backend
call venv\Scripts\activate
start /B uvicorn main:app --host 0.0.0.0 --port 8000 --reload

:: Démarrer Vue.js
cd /d C:\chemin\vers\frontend
start /B npm run dev

:: Fermer la fenêtre de commande
exit
```

Modifiez les chemins selon votre installation.

3. Exécuter le script au démarrage de Windows

Méthode 1 : Utiliser le planificateur de tâches Windows

1. Ouvrez Planificateur de tâches (taskschd.msc).


2. Cliquez sur Créer une tâche de base.


3. Nommez la tâche (ex: "Lancer mes serveurs").


4. Définissez le déclencheur sur Au démarrage de l'ordinateur.


5. Dans l'onglet Actions, sélectionnez Démarrer un programme, et pointez vers start_servers.bat.


6. Activez Exécuter avec les privilèges les plus élevés (important pour éviter les permissions).



Méthode 2 : Ajouter au dossier Démarrage

1. Appuyez sur Win + R et tapez shell:startup, puis Entrée.


2. Placez un raccourci du fichier start_servers.bat dans ce dossier.




4. Surveiller et relancer en cas d'arrêt

Pour assurer la relance automatique :

Avec nssm (Non-Sucking Service Manager) :

1. Téléchargez nssm.


2. Décompressez-le et ouvrez un terminal dans le dossier win64.


3. Installez FastAPI et Vue.js en tant que services :

```
nssm install FastAPI "C:\chemin\vers\python.exe" "C:\chemin\vers\backend\main.py"
nssm install VueJS "C:\chemin\vers\node.exe" "C:\chemin\vers\frontend\npm run dev"
```

4. Démarrez les services avec nssm start FastAPI et nssm start VueJS.


5. Les services redémarreront automatiquement en cas de crash.




Avec cette configuration, FastAPI, Vue.js et XAMPP démarreront automatiquement au lancement de Windows et redémarreront en cas d'arrêt.

