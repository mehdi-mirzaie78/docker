
# Docker

### SECTION 1 (20 minutes)
 - Virtual Macnines VS Docker
 - When VM and when Docker?
 - Benefits of each one

---------------------------------------------------

### SECTION 2 (40 minutes)
 - image and tags
 - basic commands: pull images python3.10 & python 3.9 work with them
 - containers
 - work around some commands and features
 - Dockerfile -> create image of a simple django project
 - .dockerignore
 - etc(Push to registry, load, save) *


#### Dockerfile
```Dockerfile
FROM python:3.11
LABEL Maintainer="maktab@gmail.com"
LABEL Owner="Maktab93"
LABEL version="v1.1"
WORKDIR /app
COPY requirements.txt .
RUN pip install -U pip
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```
<br>


---------------------------------------------------

### Break (15 minutes)

---------------------------------------------------

### SECTION 3 - Postgres and Django (45 minutes)
 - run postgres container without volume and with a volume
 - connect local django server to postgres container using port-forwarding
   - what happens if i had 15 containers and I wanted to connect them together -> we talk about it in docker-compose section.
 - connecting the container shell
 - importance of volumes and consistency of data



#### Django Settings for postgres database
```python
DATABASES = {
    'default': {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": "posgres",
        "USER": "postgres",
        "PASSWORD": "postgres",
        "HOST": "localhost",
        "PORT": "5432"
    }
}
```

#### Start a Postgres Container
```bash
docker run --name db -e POSTGRES_PASSWORD=postgres -e POSTGRES_USER=postgres -d  postgres
```

#### With port-forwording
```bash
-p 5432:5432 postgres
```

#### With docker volume OR With connecting local place to container
```bash
-v volume_name

-v /tmp:/var/lib/postgresql/data
```


#### Connecting to postgres container
```bash
docker exec -it db bash
psql -U postgres
```

#### Full command for postgres image:
```
docker run --name basic-postgres --rm -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=4y7sV96vA9wv46VR -e PGDATA=/var/lib/postgresql/data/pgdata -v /tmp:/var/lib/postgresql/data -p 5432:5432 -it postgres:14.1-alpine
```

---------------------------------------------------

### SECTION 4 - docker-compose & network & volume (1 hour)
 - Simple Django Project with docker
 - postgres container + app container
 - connecting these to once without .env file
 - the other time using .env file for both container and django project (because it is a sensetive information)
 



#### Dockerfile
```Dockerfile
FROM python:3.11
LABEL Maintainer="maktab@gmail.com"
LABEL Owner="Maktab93"
LABEL version="v1.1"
WORKDIR /app
COPY requirements.txt .
RUN pip install -U pip
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```
<br>

#### docker-compose.yml
```yml
services:
  db:
    container_name: postgres
    image: postgres:latest
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    networks:
      - main_net
    ports:
      - "5432:5432"
    restart: on-failure
    volumes:
      - postgres_data:/var/lib/postgresql/postgres_data
    
  app:
    build: .
    command: sh -c "python manage.py migrate && python manage.py runserver 0.0.0.0:8000"
    container_name: app
    volumes: 
      - .:/app
    depends_on:
      - db
    expose:
      - "8000"
    networks:
      - main_net
    ports:
      - "8000:8000"
    restart: on-failure

volumes:
  postgres_data:

networks:
  main_net:

```


