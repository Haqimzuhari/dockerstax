# 🐳 Dockerstax - Your Docker Development Environment

> A modern and modular Docker Compose setup for fullstack development with Node.js, PHP, PostgreSQL, MySQL, Redis, and Nginx.


## 📚 Table of Contents

* [🔧 Docker Compose for Node.js](#-docker-compose-for-nodejs)
* [📦 A. Next.js Setup](#-a-nextjs-setup)
* [📦 B. Express.js with TypeScript Setup](#-b-expressjs-with-typescript-setup)
* [🐘 Docker Compose for PHP / Laravel](#-docker-compose-for-php--laravel)
* [🐘 PostgreSQL Setup](#-postgresql-setup)
* [💼 MySQL Setup](#-mysql-setup)
* [🧠 Redis Setup](#-redis-setup)
* [🌐 Nginx Reverse Proxy Setup](#-nginx-reverse-proxy-setup)

---

## 🔧 Docker Compose for Node.js

**Base image:** `node:22-alpine`

### ✅ Shared Node Service (Next.js or Express.js)

```yaml
version: '3.8'

services:
  nodeapp:
    image: node:22-alpine
    container_name: node-app
    working_dir: /app
    volumes:
      - .:/app
    user: "1000:1000"
    tty: true
    # command: npm run dev
    ports:
      - "3000:3000"
```

---

## 📦 A. Next.js Setup

### 🧰 Setup

```sh
docker-compose up -d --build
docker-compose exec nodeapp sh
node -v
npm -v
npx create-next-app@latest
exit
docker-compose down
mv my-app/* ./
mv my-app/.* ./ 2>/dev/null || true
rm -rf my-app
```

### ⚙️ Final Steps

* Uncomment `command: npm run dev` in your `docker-compose.yml`
* Restart: `docker-compose up`

### 🔍 Access

[http://localhost:3000](http://localhost:3000)

---

## 📦 B. Express.js with TypeScript Setup

> ⚠️ Change the port from 3000 to 4000 to avoid conflict with Next.js.

### 🧰 Setup

```sh
docker-compose up -d --build
docker-compose exec nodeapp sh
node -v
npm -v

npm init -y
npm install express
npm install -D typescript ts-node @types/node @types/express
npx tsc --init
```

Create `index.ts`:

```ts
import express from 'express';
const app = express();
const port = 3000;
app.get('/', (req, res) => res.send('Hello from Express + TypeScript!'));
app.listen(port, () => console.log(`Server running on http://localhost:${port}`));
```

Update `package.json`:

```json
"scripts": {
  "dev": "ts-node index.ts"
}
```

### ⚙️ Final Steps

* Exit the container: `exit`
* Stop: `docker-compose down`
* Uncomment `command: npm run dev`
* Restart: `docker-compose up`

### 🔍 Access

[http://localhost:4000](http://localhost:4000)

---

## 🐘 Docker Compose for PHP / Laravel

**Base image:** `php:8.2-cli`

### ✅ Minimal PHP Service

```yaml
services:
  phpapp:
    image: php:8.2-cli
    container_name: php-app
    working_dir: /app
    volumes:
      - .:/app
      - ./php.ini:/usr/local/etc/php/conf.d/custom.ini
    tty: true
    # command: php artisan serve --host=0.0.0.0 --port=8000
    # command: php -S 0.0.0.0:8080
    ports:
      - "8000:8000"
      - "8080:8080"
```

---

## 📦 Laravel Setup

### 🧰 Setup

```sh
docker-compose up -d --build
docker-compose exec phpapp bash

# Create non-root user
addgroup --gid 1000 devgroup
adduser --uid 1000 --gid 1000 --disabled-password devuser

# Install tools and Composer
apt update && apt install -y curl zip unzip
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer

# Install PHP extensions
apt install -y libzip-dev libpng-dev libonig-dev libxml2-dev

docker-php-ext-install pdo pdo_mysql zip mbstring tokenizer xml bcmath gd fileinfo
exit
docker-compose down
```

### ⚙️ Final Steps

* Uncomment `user: "1000:1000"` in `docker-compose.yml`
* Uncomment Laravel command:

  ```yaml
  command: php artisan serve --host=0.0.0.0 --port=8000
  ```
* Restart: `docker-compose up`

### 🚀 Create Project

```sh
docker-compose exec phpapp bash
composer create-project --prefer-dist laravel/laravel my-laravel-app
exit
```

Move files:

```sh
mv my-laravel-app/* ./
mv my-laravel-app/.* ./ 2>/dev/null || true
rm -rf my-laravel-app
```

### 🔍 Access

[http://localhost:8000](http://localhost:8000)

---

## 🐘 PostgreSQL Setup

### ✅ Docker Compose Service

```yaml
services:
  postgres:
    image: postgres:15
    container_name: postgres_db
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"
    volumes:
      - ./data:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - postgres_external_network

networks:
  postgres_external_network:
    external: true
```

### 🔍 Notes

* Default port: `5432`
* Volume: `./data`
* Hostname: `postgres`
* Network: `postgres_external_network`

---

## 💼 MySQL Setup

### ✅ Docker Compose Service

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: mysql_db
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: app_db
      MYSQL_USER: app_user
      MYSQL_PASSWORD: app_pass
    ports:
      - "3306:3306"
    volumes:
      - ./mysql-data:/var/lib/mysql
    restart: unless-stopped
    networks:
      - mysql_external_network

networks:
  mysql_external_network:
    external: true
```

### 🔍 Notes

* Default port: `3306`
* Hostname: `mysql`
* Network: `mysql_external_network`
* Volume: `./mysql-data`

---

## 🧠 Redis Setup

### ✅ Docker Compose Service

```yaml
services:
  redis:
    image: redis:7-alpine
    container_name: redis_cache
    ports:
      - "6379:6379"
    volumes:
      - ./redis-data:/data
    restart: unless-stopped
    networks:
      - redis_external_network

networks:
  redis_external_network:
    external: true
```

### 🔍 Notes

* Default port: `6379`
* Hostname: `redis`
* Network: `redis_external_network`
* Volume: `./redis-data`

---

## 🌐 Nginx Reverse Proxy Setup

### ✅ Docker Compose Service

```yaml
services:
  nginx:
    image: nginx:stable-alpine
    container_name: nginx_proxy
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - nodeapp
      - phpapp
    networks:
      - nginx_external_network

networks:
  nginx_external_network:
    external: true
```

### 🧰 Sample `nginx.conf`

```nginx
worker_processes 1;

events {
  worker_connections 1024;
}

http {
  server {
    listen 80;

    location /next {
      proxy_pass http://nodeapp:3000;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /laravel {
      proxy_pass http://phpapp:8000;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
    }
  }
}
```

### 🔍 Access

* Next.js: [http://localhost/next](http://localhost/next)
* Laravel: [http://localhost/laravel](http://localhost/laravel)

> 💡 Make sure all services are attached to their respective external networks.
