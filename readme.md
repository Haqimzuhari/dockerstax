# Docker + Node.js + PHP Development Setup

This document contains step-by-step setup instructions for using Docker Compose to run Node.js (Next.js and Express.js with TypeScript) and PHP (for vanilla PHP and Laravel) in isolated environments.

---

## 🔧 Docker Compose for Node.js

Base image: `node:22-alpine`

### ✅ Shared Docker Compose Base (for Next.js and Express.js)

```yaml
docker-compose.yml:

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
    # command: npm run dev  # Uncomment when ready to run
    ports:
      - "3000:3000"
```

### 📦 A. Next.js Setup

#### 1. Bring up and build the container

```sh
docker-compose up -d --build
```

#### 2. Enter the container as non-root user (sh shell)

```sh
docker-compose exec nodeapp sh
```

#### 3. Verify Node.js and npm versions

```sh
node -v
npm -v
```

#### 4. Create a new Next.js project (from [nextjs.org](https://nextjs.org/docs/getting-started/installation))

```sh
npx create-next-app@latest
```

- When prompted to create a new folder, proceed (e.g., `my-app`).

#### 5. Exit the container

```sh
exit
```

#### 6. Bring down the container

```sh
docker-compose down
```

#### 7. Move Next.js files. You can move manually or follow below command

```sh
mv my-app/* ./
mv my-app/.* ./ 2>/dev/null || true
rm -rf my-app
```

#### 8. Uncomment `command: npm run dev` in docker-compose.yml

#### 9. Bring up Docker Compose again

```sh
docker-compose up
```

#### 10. Access Next.js app at:

[http://localhost:3000](http://localhost:3000)

---

### 📦 B. Express.js with TypeScript Setup

You can use the same Docker Compose setup as for Next.js, but you must change the port number (e.g., use 4000) to avoid conflicts. Next.js and Express.js cannot share the same port.

#### 1. Bring up and build the container

```sh
docker-compose up -d --build
```

#### 2. Enter the container as non-root user (sh shell)

```sh
docker-compose exec nodeapp sh
```

#### 3. Verify Node.js and npm versions

```sh
node -v
npm -v
```

#### 4. Create Express.js app with TypeScript

```sh
npm init -y
npm install express
npm install -D typescript ts-node @types/node @types/express
npx tsc --init
```

#### 5. Create files:

```ts
// index.ts
import express from 'express';
const app = express();
const port = 3000;
app.get('/', (req, res) => res.send('Hello from Express + TypeScript!'));
app.listen(port, () => console.log(`Server running on http://localhost:${port}`));
```

#### 6. Update package.json

```json
"scripts": {
  "dev": "ts-node index.ts"
}
```

#### 7. Exit the container

```sh
exit
```

#### 8. Bring down the container

```sh
docker-compose down
```

#### 9. Uncomment `command: npm run dev` in docker-compose.yml

#### 10. Bring up Docker Compose again

```sh
docker-compose up
```

#### 11. Access Express app at:

[http://localhost:4000](http://localhost:3000)

---

## 🐘 Docker Compose for PHP / Laravel

Base image: `php:8.2-cli`

### ✅ PHP Docker Compose (Minimal)

```yaml
docker-compose.yml:

version: '3.8'

services:
  phpapp:
    image: php:8.2-cli
    container_name: php-app
    working_dir: /app
    volumes:
      - .:/app
      - ./php.ini:/usr/local/etc/php/conf.d/custom.ini
    # user: "1000:1000"  # Uncomment after installing extensions
    tty: true
    # command: php artisan serve --host=0.0.0.0 --port=8000  # Laravel dev server
    # command: php -S 0.0.0.0:8080  # Vanilla PHP dev server
    ports:
      - "8000:8000"
      - "8080:8080"
```

### 🔧 Initial PHP Setup

#### 1. Build and enter container

```sh
docker-compose up -d --build
docker-compose exec phpapp bash
```

#### 2. Add user and group 1000 inside container

```sh
groupadd -g 1000 devgroup
useradd -u 1000 -g devgroup -m devuser
```

#### 3. Install required tools

```sh
apt update && apt install -y curl zip unzip
```

#### 4. Install Composer (from [getcomposer.org](https://getcomposer.org/download/))

```sh
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

#### 5. Install PHP extensions needed for Laravel

```sh
apt install -y libzip-dev libpng-dev libonig-dev libxml2-dev

docker-php-ext-install pdo pdo_mysql zip mbstring tokenizer xml bcmath gd fileinfo
```

#### 6. Exit

```sh
exit
docker-compose down
```

Now your PHP environment is ready and can be used with a non-root user.

Uncomment the user section in `docker-compose.yml`:

```yaml
user: "1000:1000"
```

---

### 🧩 Add custom `php.ini`

Create `php.ini`:

```ini
memory_limit = 512M
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 180
date.timezone = Asia/Kuala_Lumpur
```

Now you can start a vanilla PHP project.

Uncomment the vanilla PHP command in `docker-compose.yml`:

```yaml
command: php -S 0.0.0.0:8080
```

---

### 🚀 Laravel Setup

#### 1. Uncomment `user: "1000:1000"` in docker-compose.yml

#### 2. Start container again

```sh
docker-compose up -d
```

#### 3. Login as non-root user

Uncomment the user section in `docker-compose.yml`:

```
user: "1000:1000"
```

Bring up docker

```sh
docker-compose exec phpapp bash
```

#### 4. Install Node.js and npm using NVM

(From [https://github.com/nvm-sh/nvm#install--update-script](https://github.com/nvm-sh/nvm#install--update-script))

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
. ~/.nvm/nvm.sh
nvm install --lts
```

#### 5. Verify PHP extensions

```sh
php -m
```

Ensure the required Laravel extensions are listed.

#### 6. Create Laravel project

**DO NOT** install Laravel and **DO NOT** run

```
laravel new my-laravel-app
```

Instead, use composer to begin Laravel project

Lates version

```sh
composer create-project --prefer-dist "laravel/laravel" my-laravel-app
```

Older version

```
composer create-project --prefer-dist "laravel/laravel:^10.0" my-laravel-app
```

#### 7. Exit and bring down container

```sh
exit
docker-compose down
```

#### 8. Move Laravel files into working directory manually or follow below command

```sh
mv my-laravel-app/* ./
mv my-laravel-app/.* ./ 2>/dev/null || true
rm -rf my-laravel-app
```

#### 9. Uncomment Laravel `command:` section

```yaml
# command: php artisan serve --host=0.0.0.0 --port=8000
```

#### 10. Start development

```sh
docker-compose up
```

Visit: [http://localhost:8000](http://localhost:8000)

**⚠️ Do not delete the PHP image unless you're ready to reinstall everything.**

---
