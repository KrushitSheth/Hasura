1. Describe any issues you discovered with the shared deployment artifacts and the steps you took to remediate the issue(s).
-> Given deployment file for docker compose:
docker-compose.yaml
##############################################
version: "3.6"
services:
  redis:
    image: redis:latest
    restart: always
    ports:
      - "6379:6379"
  postgres:
    image: postgres:15
    restart: always
    ports:
      - "5432:5432"
    volumes:
      - ./Chinook_PostgreSql.sql:/docker-entrypoint-initdb.d/Chinook_PostgreSql.sql
      - ./Chinook_PostgreSql_AutoIncrementPKs.sql:/docker-entrypoint-initdb.d/Chinook_PostgreSql_AutoIncrementPKs.sql
      - ./Chinook_PostgreSql_SerialPKs.sql:/docker-entrypoint-initdb.d/Chinook_PostgreSql_SerialPKs.sql
    environment:
      POSTGRES_PASSWORD: postgrespassword
  graphql-engine:
    image: hasura/graphql-engine:v2.42.0
    ports:
      - "8080:8080"
    restart: always
    environment:
      HASURA_GRAPHQL_ADMIN_SECRET: "Pinkgreen"
      HASURA_GRAPHQL_METADATA_DATABASE_URL: postgres://chinook:chinook@postgres:5432/chinook
      HASURA_GRAPHQL_ENABLE_CONSOLE: "true"
      HASURA_GRAPHQL_DEV_MODE: "true"
      HASURA_GRAPHQL_REDIS_URL: "redis://redis:6379"
      HASURA_GRAPHQL_RATE_LIMIT_REDIS_URL: "redis://redis:6379"
      PG_DATABASE_URL: postgres://chinook:chinook@postgres:5432/chinook
volumes:
  db_data:
##############################################

Now,
STEP 1: Download following files from chinook dataset and save it in the docker-compose.yaml file's directory:
a) https://github.com/lerocha/chinook-database/blob/master/ChinookDatabase/DataSources/Chinook_PostgreSql.sql
b) https://github.com/lerocha/chinook-database/blob/master/ChinookDatabase/DataSources/Chinook_PostgreSql_AutoIncrementPKs.sql
c) https://github.com/lerocha/chinook-database/blob/master/ChinookDatabase/DataSources/Chinook_PostgreSql_SerialPKs.sql

STEP 2: Modify docker-compose.yaml file to use above files attached as volumes and also modify following key:value pair:
a) volumes:
      - ./Chinook_PostgreSql.sql:/docker-entrypoint-initdb.d/Chinook_PostgreSql.sql
      - ./Chinook_PostgreSql_AutoIncrementPKs.sql:/docker-entrypoint-initdb.d/Chinook_PostgreSql_AutoIncrementPKs.sql
      - ./Chinook_PostgreSql_SerialPKs.sql:/docker-entrypoint-initdb.d/Chinook_PostgreSql_SerialPKs.sql
b) image: hasura/graphql-engine:v2.42.0  (as per latest hasura v2 documentation) Reference: https://raw.githubusercontent.com/hasura/graphql-engine/stable/install-manifests/docker-compose/docker-compose.yaml
c) ports:
      - "8080:8080"       (as hasura grapgql engine container listens on 8080)
d) PG_DATABASE_URL: postgres://chinook:chinook@postgres:5432/chinook      (set as environment variable so no need to give username and password url in hasura console)
e) HASURA_GRAPHQL_ENABLE_CONSOLE: "true" (helps to view everything as GUI)
f) HASURA_GRAPHQL_ADMIN_SECRET: "Pinkgreen"   (for logging into hasura console)

STEP 3: Use below command to run the docker-compose.yaml file:
docker compose up -d             (In the directory where the docker-compose.yaml file is present)

STEP 4: After all 3 containers comes up exec into postgres container and execute following commands:
a) docker ps  (to get the postgres container id)
b) docker exec -it <container-id> bash
c) cd docker-entrypoint-initdb.d/ 
d) psql -U chinook -f Chinook_PostgreSql.sql
e) psql -U chinook -d chinook
f) chinook=# \password

STEP 5: Open any browser and use following url to access hasura console:
http://localhost:8080/                            (and give password set in HASURA_GRAPHQL_ADMIN_SECRET key value pair to login)

STEP 6: Click on Data > Manage and click on "Connect Database" choose postgres and then choose "Connect Existing Database".

STEP 7: Fill database name and connect database via environment variable in this case "PG_DATABASE_URL" key value pair and click on "Connect Database".

STEP 8: You can now see database schema and relations and click on track all so that hasura can reduce the complexity via creating SQL statements automatically.
