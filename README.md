<p align="center">
    <a href="https://pocketbase.io" target="_blank" rel="noopener">
        <img src="https://i.imgur.com/5qimnm5.png" alt="PocketBase - open source backend in 1 file" />
    </a>
</p>

## Fork Explanation ❤️ Pocketbase  
[Pocketbase Documentation](https://pocketbase.io/docs)  

Pocketbase is a great product and very efficient for small-mid projects. It has no any additional setup for any other features and it is very easy to use on a single server.  

In our use-case we really need to use postgres as a main database and operate it manually. Also we love what Pocketbase does with CRUD operation and RBAC implementations via simple notations. So we want to use it. Thus we forked it and make it compatible with postgres using its own library called ["pocketbase/dbx"](https://github.com/pocketbase/dbx).  

We are still working on it and we will update the documentation as soon as we finish the project.  

We just added a following features additinonally to the Pocketbase:
- We use [Twitter Snowflake for ID generation](https://github.com/AlperRehaYAZGAN/postgresbase/blob/master/migrations/1640988000_init.go#L48). Every table ID is generated by postgres function as data type varchar(32)  
- We converted [created and updated columns](https://github.com/AlperRehaYAZGAN/postgresbase/blob/master/migrations/1640988000_init.go#L73-L74) to postgres native date types `TIMESTAMPTZ` to support native date operations  
- We write [json functions for postgres](https://github.com/AlperRehaYAZGAN/postgresbase/blob/master/migrations/1640988000_init.go) in migration files to support json equivalent operations from Pocketbase.  
- We add support [RSA256 JWT Public Private Keys](https://github.com/AlperRehaYAZGAN/postgresbase/blob/master/tools/security/jwt.go) while encoding and decoding token. In our case we need to implement Pocketbase to our existing project with RSA keypair. Currently (Pocketbase v0.20.5) supports symmetric encoding only and we extend it.  


## Usage  
You can easily fork and setup the project.  

```bash
# clone and download libraries
git clone https://github.com/AlperRehaYAZGAN/postgresbase
cd postgresbase
go mod download

# docker-compose has 3 service for test pocketbase all features:
# 1. Postgres: runs on port 5432
# 1. postgres://user:pass@localhost/logs?sslmode=disable
# 2. minio: UI runs on port 9001 and API on 9000  (minio123:minio123)
# 2. s3://minio123:minio123@localhost:9000/public
# (dont forget to manually create bucket called "public" via web ui to establish s3 connection from pocketbase)
# 3. mailhog: port: SMTP-1025 and UI-8025
# 3. smtp://localhost:1025 - http://localhost:8025
docker-compose up -d

# before run the project, you need to create and set RSA Public key pair for JWT before run the application.
# you can use following command to generate RSA key pair
openssl genrsa -out ./keys/private.pem 2048
openssl rsa -in ./keys/private.pem -outform PEM -pubout -out ./keys/public.pem

# after generating keys, you can set as environment variables
export JWT_PRIVATE_KEY=$(cat ./keys/private.pem)
export JWT_PUBLIC_KEY=$(cat ./keys/public.pem)

# export is success you can run the project ✅

# run the project with postgres connection info
CGO_ENABLED=0 \
LOGS_DATABASE="postgresql://user:pass@localhost/logs?sslmode=disable" \
DATABASE="postgresql://user:pass@localhost/postgres?sslmode=disable" \
    go run -tags pq github.com/pocketbase/pocketbase/examples/base serve  

```

### Docker

```bash
# build docker image
docker buildx build --platform linux/amd64 -t <your-name>/postgresbase:1.0.0 .  

# before running application generate RSA256 public-private key pair for jwt signing
# you can use following command to generate RSA key pair
openssl genrsa -out ./keys/private.pem 2048
openssl rsa -in ./keys/private.pem -outform PEM -pubout -out ./keys/public.pem

# run docker image
docker run -d --name postgresbase \
    -p 8090:8090 \
    -e LOGS_DATABASE="postgresql://user:pass@<postgres-ip>:5432/logs?sslmode=disable" \
    -e DATABASE="postgresql://user:pass@<postgres-ip>:5432/postgres?sslmode=disable" \
    -e JWT_PRIVATE_KEY="$(cat $PWD/keys/private.pem)" \
    -e JWT_PUBLIC_KEY="$(cat $PWD/keys/public.pem)" \
    <your-name>/postgresbase:1.0.0 serve --http=0.0.0.0:8090
```
