# Postgres image
FROM postgres:14.1-alpine

# Environment
# ENV POSTGRES_DB=db POSTGRES_USER=usr POSTGRES_PASSWORD=pwd

# Initialize database with SQL scripts
COPY docker-entrypoint-initdb.d /docker-entrypoint-initdb.d/

# The port number the container should expose
EXPOSE 5432