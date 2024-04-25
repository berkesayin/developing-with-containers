## Using Containers

Developing, testing and deploying application with containers.

### Table Of Contents

[**1.About The Application**](#about)
[**2.Create Docker Assets**](#docker-assets)
[**3.Run The Application**](#run-app)
[**4.Add A Local DB And Persist Data**]
[**5.Run The App Again**](#run-app-again)
[**6.Run PG Admin As A Container With Same Network**](#run-pg-admin)

### About The Application <a name="about"></a>

### Create Docker Assets <a name="docker-assets"></a>

```sh
docker init 
```

#### Directory Tree 

```
├── docker-nodejs-sample/
│ ├── spec/
│ ├── src/
│ ├── .dockerignore
│ ├── .gitignore
│ ├── compose.yaml
│ ├── Dockerfile
│ ├── package-lock.json
│ ├── package.json
│ ├── README.Docker.md
│ └── README.md
```

### Run The Application <a name="run-app"></a>

`compose.yaml` file. 

```yaml
services:
  server:
    build:
      context: .
    environment:
      NODE_ENV: production
    ports:
      - 3000:3000
```

Run this command inside the `developing-with-containers` directory. 

```sh
docker compose up --build
```
#### Image Created 

```sh
docker images
```

```sh
REPOSITORY                               TAG           IMAGE ID        CREATED           SIZE
developing-with-containers-server        latest        a987af256514    52 seconds ago    153MB
```
#### Container Running 

```sh
docker ps
```

```sh
CONTAINER ID   IMAGE                               COMMAND                  CREATED         STATUS         PORTS                    NAMES
937461c2d7b7   developing-with-containers-server   "docker-entrypoint.s…"   3 minutes ago   Up 3 minutes   0.0.0.0:3000->3000/tcp   developing-with-containers-server-1
```

Go to http://localhost:3000/ and add new todo item. 

#### Stop The Application

```sh
docker compose down
```

or `ctrl + c` and 

```sh
docker compose rm
```

### Add A Local DB And Persist Data <a name="docker-assets"></a>

Containers can be used to set up local services, like a database. Update the `compose.yaml` file for `postgre` db container. 

#### Update `compose.yaml` file:

```yaml
services:
  server:
    build:
      context: .
    container_name: todo-server-c
    ports:
      - 3000:3000
    environment:
      NODE_ENV: production
      POSTGRES_HOST: db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD_FILE: /run/secrets/db-password
      POSTGRES_DB: example
    depends_on:
      db:
        condition: service_healthy
    secrets:
      - db-password
    networks:
      - todo-network
  db:
    image: postgres
    container_name: postgres-db-c
    restart: always
    user: postgres
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=example
      - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
      - POSTGRES_HOST_AUTH_METHOD=md5 #added 
      - PGDATA=/var/lib/postgresql/data/pgdata #added 
    expose:
      - 5432
    ports:
      - "5432:5432"
    healthcheck:
      test: [ "CMD", "pg_isready" ]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - todo-network
volumes:
  db-data:
secrets:
  db-password:
    file: db/password.txt
networks:
  todo-network:
    driver: bridge
```

#### Create db/password.txt

In the root directory, create a new directory named `db`. Inside it, create a file named `password.txt`. Add a password there. The password must be on a single line.

`db/password.txt` : Resample-Landlady5-Bottle

### Run The App Again <a name="run-app-again"></a>

```sh
docker compose up --build 
```

#### Images

```sh
docker images
```

```sh
REPOSITORY                             TAG           IMAGE ID       CREATED         SIZE
developing-with-containers-server      latest        b84a095f6745   18 hours ago    153MB
postgres                               latest        d4ffc32b30ba   2 months ago    453MB
```

#### Containers Running 

```sh
docker ps
```

```sh
CONTAINER ID   IMAGE                               COMMAND                  CREATED         STATUS                   PORTS                    NAMES
05e6b9a6e1c1   developing-with-containers-server   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes             0.0.0.0:3000->3000/tcp   todo-server-c
cd7628aaaeee   postgres                            "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes (healthy)   0.0.0.0:5432->5432/tcp   postgres-db-c
```

![img](./assets/app1.png)

### Run PG Admin As A Container With Same Network <a name="run-pg-admin"></a>

We may need to check `database` and `tables` with `datas` created for the application. For this, we will use `pgadmin` as a container. It will be using the same `network` with the `application (todo-server-c)` and `postgres (postgres-db-c)` containers.  

#### Check Network Created 

The `network` is created with the compose configuration after running `docker compose up --build command`. To see the `docker network` for our application use this command: 

```sh
docker network ls
```

```sh
NETWORK ID     NAME                                      DRIVER    SCOPE
42a5d9a6172c   developing-with-containers_todo-network   bridge    local
```

#### Check Network Used By Containers

**Container Name**: `todo-server-c` & **Container ID**: `05e6b9a6e1c1`

```sh
docker inspect 05e6b9a6e1c1 -f "{{json .NetworkSettings.Networks }}"
```

```json
{
  "developing-with-containers_todo-network": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": [
      "todo-server-c",
      "server"
    ],
    "MacAddress": "02:42:c0:a8:00:03",
    "NetworkID": "42a5d9a6172c8e092d1aff994931297d5b6b6b8ad39418fb2fd2b5f3804af21b",
    "EndpointID": "fd90f32c7f0f37bc81a8ab8c9b8f36aea1fbdfb6b993a5dd1784a74bdfeaf310",
    "Gateway": "192.168.0.1",
    "IPAddress": "192.168.0.3",
    "IPPrefixLen": 20,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "DriverOpts": null,
    "DNSNames": [
      "todo-server-c",
      "server",
      "05e6b9a6e1c1"
    ]
  }
}
```

**Container Name**: `postgres-db-c` & **Container ID**: `cd7628aaaeee`

```sh
docker inspect cd7628aaaeee -f "{{json .NetworkSettings.Networks }}"
```

```json
{
  "developing-with-containers_todo-network": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": [
      "postgres-db-c",
      "db"
    ],
    "MacAddress": "02:42:c0:a8:00:02",
    "NetworkID": "42a5d9a6172c8e092d1aff994931297d5b6b6b8ad39418fb2fd2b5f3804af21b",
    "EndpointID": "ea4bb939b72e09813e6ee1d6e252b53bba68d59b4aed72e3429c77810db06e42",
    "Gateway": "192.168.0.1",
    "IPAddress": "192.168.0.2",
    "IPPrefixLen": 20,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "DriverOpts": null,
    "DNSNames": [
      "postgres-db-c",
      "db",
      "cd7628aaaeee"
    ]
  }
}
```
#### Run PGAdmin As A Container With Same Network

We will use `PG Admin` as another container connected to application through same `network`. The image: **dpage/pgadmin4**

```sh
docker run -p 8080:80 \
  -e 'PGADMIN_DEFAULT_EMAIL=sayinberke34@gmail.com' \
  -e 'PGADMIN_DEFAULT_PASSWORD=Resample-Landlady5-Bottle' \
  --network=developing-with-containers_todo-network \
  --name=pgadmin-server-c \
  -d dpage/pgadmin4
```

```sh
docker ps
```

```sh
CONTAINER ID   IMAGE                               COMMAND                  CREATED          STATUS                    PORTS                           NAMES
30b61c36a9c2   dpage/pgadmin4                      "/entrypoint.sh"         2 seconds ago    Up 2 seconds              443/tcp, 0.0.0.0:8080->80/tcp   pgadmin-server-c
05e6b9a6e1c1   developing-with-containers-server   "docker-entrypoint.s…"   23 minutes ago   Up 22 minutes             0.0.0.0:3000->3000/tcp          todo-server-c
cd7628aaaeee   postgres                            "docker-entrypoint.s…"   23 minutes ago   Up 23 minutes (healthy)   0.0.0.0:5432->5432/tcp          postgres-db-c
```

#### Check PG Admin Container's Network

```sh
docker inspect 30b61c36a9c2 -f "{{json .NetworkSettings.Networks }}"
```

```json
{
  "developing-with-containers_todo-network": {
    "IPAMConfig": null,
    "Links": null,
    "Aliases": null,
    "MacAddress": "02:42:c0:a8:00:04",
    "NetworkID": "42a5d9a6172c8e092d1aff994931297d5b6b6b8ad39418fb2fd2b5f3804af21b",
    "EndpointID": "3fde12256fa7d1a028a69258453f6d0b3d4240489a2e2eb6066ed72c77118263",
    "Gateway": "192.168.0.1",
    "IPAddress": "192.168.0.4",
    "IPPrefixLen": 20,
    "IPv6Gateway": "",
    "GlobalIPv6Address": "",
    "GlobalIPv6PrefixLen": 0,
    "DriverOpts": null,
    "DNSNames": [
      "pgadmin-server-c",
      "30b61c36a9c2"
    ]
  }
}
```

All 3 containers, `todo-server-c`, `postgres-db-c` and `pgadmin-server-c` use the same network named `developing-with-containers_todo-network` with `network ID: 42a5d9a6172c`.

The container `pgadmin-server-c` created and uses port `8080`. Get access: http://localhost:8080/

![img](./assets/pgadmin1.png)

**E-Mail / Username**: sayinberke34@gmail.com
**Password**: Resample-Landlady5-Bottle

**Admin Dashboard After Login** 

![img](./assets/pgadmin2.png)

**Add New Server**

![img](./assets/pgadmin3.png)

![img](./assets/pgadmin4.png)

- `General - Name`: `my-server`
- `Connection - Hostname / address`: `db` (`db` service at `compose`)
- `Connection - port`: `5432` (the exposed and default `PostgreSQL port` at `compose`)
- `Connection - Maintanance database`: `example` (`db name` at `compose`)
- `Connection - Username`: `postgres` (`db user` at `compose`)
- `Connection - Password`: `Resample-Landlady5-Bottle` (used for `container`)

`my-server/databases/example/Schemas/Tables/todo_items`

![img](./assets/pgadmin5.png)

`Application`

![img](./assets/app2.png)

