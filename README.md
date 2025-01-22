# Setup Laravel on Docker

## Description  
This guide walks you through setting up a Laravel project using Docker. Simplify your development workflow by creating a consistent and portable environment for your team. Say goodbye to the hassle of manual installations and ensure everyone works on the same setup, effortlessly.

---
เรามาเตรียมโครงสร้างโปรเจกต์ที่หน้าตาประมาณนี้ก่อนครับ
```bash
# myproject  หรือชื่ออื่นที่ต้องการ
myproject/
 ├── app/ # Laravel application
 ├── Dockerfile # Dockerfile สำหรับการตั้งค่า PHP และ Apache
 ├── docker-compose.yml # Docker Compose configuration
```
---
# ต่อไปแก้ไข ไฟล์ docker-compose.yml
```bash
version: '3.8'

services:
  # Laravel app service
  app:
    build:
      context: .
      dockerfile: Dockerfile

    # เราสามารถแก้ไข projectname_app เป็นชื่อที่ต้องการได้เลยครับ เช่น myweb_app
    container_name: myweb_app 
    volumes:
      - ./app:/var/www/html
    ports:
      - "8000:80"
    networks:
      - app-network
    depends_on:
      - db

  # MySQL database service
  db:
    image: mysql:5.7
  # เราสามารถแก้ไข app_db เป็นชื่อที่ต้องการได้เลยครับ เช่น myweb_db
    container_name: myweb_db
    environment:
      # ตั้งค่ารหัสผ่านเพื่อเข้าถึงฐานข้อมูล
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: dababasename
      MYSQL_USER: username_user
      MYSQL_PASSWORD: username_password
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app-network

  # phpMyAdmin service
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: makeplan_phpmyadmin
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: rootpassword
    ports:
      - "8080:80"
    networks:
      - app-network
    depends_on:
      - db

networks:
  app-network:
    driver: bridge

volumes:
  db_data:
```

----
# 2. ไฟล์ Dockerfile
ในไฟล์นี้ เราจะตั้งค่า Apaches, PHP และ Composer เพื่อใช้ใน Laravel

```bash
# ใช้ PHP 8.2 พร้อม Apache
FROM php:8.2-apache

# ติดตั้ง PHP extensions ที่จำเป็น
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libonig-dev \
    libzip-dev \
    zip \
    unzip \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd pdo_mysql mbstring zip bcmath opcache

# ติดตั้ง Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# ตั้งค่า DocumentRoot ให้ Laravel
ENV APACHE_DOCUMENT_ROOT /var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

# เปิดใช้งาน Apache mod_rewrite
RUN a2enmod rewrite

# ตั้งสิทธิ์ให้โฟลเดอร์ storage และ bootstrap/cache
RUN mkdir -p /var/www/html/storage /var/www/html/bootstrap/cache
RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache \
    && chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache

# กำหนด Working Directory
WORKDIR /var/www/html
```

----
# 3. ขั้นตอนการตั้งค่า
## สร้าง Laravel Project ในโฟลเดอร์ app/

- **เข้าไปที่โฟลเดอร์ app/**:
```bash
cd app

```
- **ติดตั้ง Laravel**:
```bash
composer create-project laravel/laravel .
```
- **ตั้งค่า .env: เปิดไฟล์ .env และตั้งค่าการเชื่อมต่อฐานข้อมูลให้ตรงกับค่าที่กำหนดใน docker-compose.yml**:

```bash
#เปิด vscode (หรือ text editor อื่น) เพื่อทำการแก้ไข .env
code app
```

```.env
#แก้ไข .env ให้ตรงกับไฟล์ docker-compose.yml
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=dababasename
DB_USERNAME=username_user
DB_PASSWORD=rootpassword
```

## เริ่มต้น Docker Compose
- **กลับไปที่โฟลเดอร์หลัก (makeplan/)**
```bash
cd ..
```

- **รัน Docker Compose**
```bash
docker-compose up -d
```
- **ตรวจสอบสถานะของ Container**
```bash
docker-compose ps
```
---

## เข้าใช้งาน Laravel ผ่าน localhost
- <a href="http://localhost:8000">http://localhost:8000<a>

## เข้าใช้งาน phpMyAdmin ผ่าน localhost 
- โดยใช้ PMA_HOST: db และข้อมูลฐานข้อมูลใน .env**
<a href="http://localhost:8080">http://localhost:8080<a>

---
## แก้ไขปัญหาเมื่อเจอ 403 Forbidden
- ตรวจสอบว่า DocumentRoot ใน Dockerfile ตั้งค่าเป็น /var/www/html/public 
- ตรวจสอบสิทธิ์ของโฟลเดอร์ storage และ bootstrap/cache ให้สามารถเขียนได้
```bash
sudo chmod -R 775 app/storage app/bootstrap/cache
sudo chown -R www-data:www-data app/storage app/bootstrap/cache
```

## การเพิ่มฐานข้อมูล
- **เข้าสู่ Container ของ app**
```bash
docker-compose exec app bash
```
- **รันคำสั่ง Migration เพื่อสร้างตาราง**
```bash
docker-compose exec app bash
```
