# Docker

Afin de créer une image docker pour chaque application, j'ai simplement défini un `Dockerfile` à la racine de chaque dossier contenant les API.

- Une fois les fichiers ajoutés, on peut créer les images associées : `docker build -t mon-api-python:1.0 .`

- Exécuter l'image précédemment créée : `docker run -p 8000:8000 mon-api-python`

- Faire un appel sur l'API : `curl http://localhost:8000/hello`

Si toutes les étapes précédentes se sont bien déroulées, on obtient une page qui affiche "word".