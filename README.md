# Akash MongoDB Restore

An auto-restoring MongoDB server running on Akash, with backups taken on a
configurable schedule. Backups are stored on Skynet, which is a decentralised
cloud storage technology.

Ultimately this is a two container setup, one MongoDB server and one scheduler
container to restore the database on boot, and run a cronjob to back it up. 

### Using with an app container

Alternatively add your own app container to the deploy.yml and expose Mongo's
standard 27017 port to your application only for a local server.

For example:

```
services:
  app: 
    image: myappimage:v1
    depends_on: 
      - service: mongodb

  scheduler:
    image: skynetlabs/akash-mongo-scheduler:v1.0.1
    container_name: scheduler
    depends_on:
      - mongodb
    environment:
      - MONGODB_DATABASE=akash_mongo
      - MONGODB_USERNAME=root
      - MONGODB_PASSWORD=password
      - SKYNET_SEED=supersecret
      - SKYNET_DATAKEY=mongobackup

  mongodb:
    image: skynetlabs/akash-mongo-mongodb:v1.0.0
    restart: always
    container_name: mongodb
    ports:
      - "27017:27017"

```

### Environment variables

- `MONGODB_USERNAME=root` - your MongoDB username
- `MONGODB_PASSWORD=password` - your MongoDB password
- `MONGODB_DATABASE=akash_mongo` - your MongoDB database name

- `SKYNET_SEED=supersecret` - a passphrase to encrypt your backups with
- `SKYNET_DATAKEY=secret` - a datakey that decides where the backup gets stored

- `BACKUP_SCHEDULE=*/5 * * * *` - the cron schedule for backups


## Usage

You can run the application locally using Docker compose. 

Copy the `.env.dist` file to `.env` and update if necessary.
Run `docker-compose up` to build and run the application

The database will be stored on Skynet, under a Skylink that is decided by the
`SKYNET_SEED` and `SKYNET_DATAKEY`. Using the same values for those environment
variables will allow you to restore your database. Note that the database will
also get encrypted using the provided `SKYNET_SEED`

You can also deploy this to Akash by creating a deployment using `deploy.yml` as
the manifest file. See https://docs.akash.network/guides/deploy for more
information on how to deploying an application on Akash.
