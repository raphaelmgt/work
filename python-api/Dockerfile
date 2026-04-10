# On part d'une image légère python:3.11
FROM python:3.11-slim

WORKDIR /app

COPY . .

# On installe les dépendances présentes dans requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 8000

# On définit les commandes lancées lorsque le conteneur démarre
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
