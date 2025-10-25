# 1. Partir de l'image Postgres officielle et légère
FROM postgres:17.2-alpine

# 2. Copier les scripts d'initialisation
# L'image Postgres est conçue pour exécuter (par ordre alpha)
# tous les scripts .sql trouvés dans ce dossier au premier démarrage.
COPY ./init/ /docker-entrypoint-initdb.d/