## Agile Software Practice - Assignment 1.

### ⚠️⚠️⚠️ TO THESE WHO REFERENCED THIS REPO:

PLEASE NOTE THAT THIS REPO IS A ASSIGNMENT, YOU SHALL NOT COPY ANY CODE FROM THIS REPO.

IF YOU DO SO, YOU WILL BE RESPONSIBLE FOR ANY CONSEQUENCES.

PLEASE HAVE CITATION IF YOU REFERENCED THIS REPO.


reference:

1. how to seed database: https://levelup.gitconnected.com/dockerising-your-database-for-local-development-seeding-data-mariadb-sql-and-mongodb-nosql-f18ba6d8c9ed

2. how to start service conditionally: https://docs.docker.com/compose/how-tos/profiles/#start-specific-profiles

## Explanation

### Launch Order

use `depends_on` to control.

### Network Isolation

we have 3 network namespace here: 

1. app_network (movies and redis)
2. db_network (movies, mongodb and seeding-mongo)
3. admin_network (mongo-express and mongodb)

### Dev/Prod Mode

use `profiles` to control.

e.g.  `docker-compose -f .\docker-compose.yml --profile dev up -d` runs all services (normal ones and dev ones.)

### Seeding

```dockerfile
seeding-mongo:
    image: mongo:8.0-rc
    depends_on:
      - mongodb
    profiles:
      - dev # only start this service in the dev profile
    volumes:
      - ./seeding.json:/seeding.json
      - ./seeding-mongo.sh:/seeding-mongo.sh
    environment:
      - MONGODB_DATABASE=${MONGODB_DATABASE}
      - MONGODB_URI=mongodb://${MONGODB_USERNAME}:${MONGODB_PASSWORD}@mongodb:27017
    command:
      /seeding-mongo.sh
    networks:
      - db_network
```

this one-time service is used to run `seeding-mongo.sh` to seed the database, run the following bash.

mongoimport tool is used here to import a array json to mongo db.

```bash
mongoimport \
  --host mongodb \
  --username admin \
  --password password \
  --db tmdb_movies \
  --collection movies \
  --type json \
  --file seeding.json \
  --jsonArray \
  --authenticationDatabase=admin # fix auth error
```

