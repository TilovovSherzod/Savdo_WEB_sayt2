### SAVDO - Online Do'kon

SAVDO - Uzbek tilidagi online do'kon loyihasi. Bu loyiha Django frameworkida yaratilgan bo'lib, maxsulotlarni ko'rish, savatga qo'shish, buyurtma berish va boshqarish imkoniyatlarini o'z ichiga oladi.

## Loyiha haqida

Bu loyiha quyidagi imkoniyatlarni taqdim etadi:

- Maxsulotlarni ko'rish va qidirish
- Maxsulotlarni savatga qo'shish
- Buyurtma berish
- Foydalanuvchi profili
- Admin panel orqali maxsulotlarni boshqarish
- Ombor boshqaruvi
- Hisobotlar
- Barcode skaner


## Talablar

- Python 3.8+
- Docker va Docker Compose
- Git


## Loyihani yuklab olish

Loyihani GitHub'dan yuklab olish uchun:

```shellscript
git clone https://github.com/username/savdo_web_sayt.git
cd savdo_web_sayt
```

## Docker orqali ishga tushirish

### 1. Docker-compose faylini yaratish

Loyiha papkasida `docker-compose.yml` faylini yarating:

```yaml
version: '3'

services:
  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    env_file:
      - ./.env
    environment:
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_DB=postgres

  web:
    build: .
    restart: always
    depends_on:
      - db
    env_file:
      - ./.env
    volumes:
      - .:/code
      - static_volume:/code/staticfiles
      - media_volume:/code/media
    command: >
      sh -c "python manage.py migrate &&
             python manage.py collectstatic --noinput &&
             gunicorn savdo.wsgi:application --bind 0.0.0.0:8000"

  nginx:
    image: nginx:1.21
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf
      - static_volume:/code/staticfiles
      - media_volume:/code/media
    depends_on:
      - web

volumes:
  postgres_data:
  static_volume:
  media_volume:
```

### 2. Dockerfile yaratish

Loyiha papkasida `Dockerfile` faylini yarating:

```dockerfile
FROM python:3.9

WORKDIR /code

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

RUN pip install --upgrade pip
COPY requirements.txt /code/
RUN pip install -r requirements.txt

COPY . /code/
```

### 3. .env faylini yaratish

Loyiha papkasida `.env` faylini yarating:

```plaintext
DEBUG=0
SECRET_KEY=your-secret-key-here
POSTGRES_DB=postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_HOST=db
POSTGRES_PORT=5432
```

### 4. requirements.txt faylini yaratish

Loyiha papkasida `requirements.txt` faylini yarating:

```plaintext
Django==4.2.7
psycopg2-binary==2.9.5
Pillow==9.4.0
gunicorn==20.1.0
djangorestframework==3.14.0
djangorestframework-simplejwt==5.2.2
django-cors-headers==3.14.0
```

### 5. Nginx konfiguratsiyasi

`nginx` papkasini yarating va ichida `nginx.conf` faylini yarating:

```shellscript
mkdir -p nginx
```

`nginx/nginx.conf` faylini yarating:

```plaintext
server {
    listen 80;
    server_name localhost;

    location /static/ {
        alias /code/staticfiles/;
    }

    location /media/ {
        alias /code/media/;
    }

    location / {
        proxy_pass http://web:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 6. Docker orqali ishga tushirish

```shellscript
# Docker image'larni yaratish va konteynerlarni ishga tushirish
docker-compose up -d --build

# Admin foydalanuvchisini yaratish
docker-compose exec web python manage.py createsuperuser

# Log'larni ko'rish
docker-compose logs -f
```

## Serverga joylash

### 1. VPS serverda o'rnatish

1. Serverga SSH orqali ulanish:


```shellscript
ssh username@server_ip
```

2. Kerakli dasturlarni o'rnatish:


```shellscript
sudo apt update
sudo apt install -y docker.io docker-compose git
sudo systemctl enable docker
sudo systemctl start docker
```

3. Loyihani yuklab olish:


```shellscript
git clone https://github.com/username/savdo_web_sayt.git
cd savdo_web_sayt
```

4. `.env` faylini sozlash (yuqoridagi misol asosida)
5. Docker orqali ishga tushirish:


```shellscript
sudo docker-compose up -d --build
```

6. Admin foydalanuvchisini yaratish:


```shellscript
sudo docker-compose exec web python manage.py createsuperuser
```

### 2. Domain nomi bilan sozlash

1. `nginx/nginx.conf` faylini yangilang:


```plaintext
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location /static/ {
        alias /code/staticfiles/;
    }

    location /media/ {
        alias /code/media/;
    }

    location / {
        proxy_pass http://web:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

2. SSL sertifikat o'rnatish (Certbot yordamida):


```shellscript
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

3. Docker konteynerlarni qayta ishga tushirish:


```shellscript
sudo docker-compose down
sudo docker-compose up -d --build
```

## Loyihani ishlatish

1. Admin paneliga kirish: `http://localhost/admin/` yoki `https://yourdomain.com/admin/`
2. Kategoriyalar va maxsulotlar qo'shish
3. Saytni ko'rish: `http://localhost/` yoki `https://yourdomain.com/`


## Xatoliklarni bartaraf etish

### CSRF xatoligi

Agar savatga qo'shish yoki boshqa AJAX so'rovlarda CSRF xatoligi chiqsa, quyidagi o'zgarishlarni tekshiring:

1. `asosiy/views.py` faylida `@csrf_exempt` dekoratori qo'shilganligini tekshiring:


```python
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
@require_POST
def savatga_qoshish(request):
    # ...
```

2. JavaScript fayllarida CSRF token headerlar olib tashlangan bo'lishi kerak:


```javascript
fetch("/savatga-qoshish/", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    // CSRF token header olib tashlangan
  },
  // ...
})
```

### Docker bilan bog'liq muammolar

1. Konteynerlar holatini tekshirish:


```shellscript
docker-compose ps
```

2. Log'larni ko'rish:


```shellscript
docker-compose logs -f
```

3. Konteynerlarni qayta ishga tushirish:


```shellscript
docker-compose down
docker-compose up -d --build
```

4. Ma'lumotlar bazasini qayta ishga tushirish:


```shellscript
docker-compose down
docker volume rm savdo_web_sayt_postgres_data
docker-compose up -d --build
```

## Loyihani yangilash

1. Yangi o'zgarishlarni yuklab olish:


```shellscript
git pull origin main
```

2. Docker konteynerlarni qayta ishga tushirish:


```shellscript
docker-compose down
docker-compose up -d --build
```

3. Migratsiyalarni bajarish:


```shellscript
docker-compose exec web python manage.py migrate
```

4. Statik fayllarni yig'ish:


```shellscript
docker-compose exec web python manage.py collectstatic --noinput
```

## Xavfsizlik

1. `.env` faylida xavfsiz parollar va kalitlar ishlatilganligiga ishonch hosil qiling
2. Ishlab chiqarish muhitida `DEBUG=0` qiymatini o'rnating
3. Muntazam ravishda tizimni yangilab turing
4. Faqat kerakli portlarni ochiq qoldiring (80, 443)


## Qo'shimcha ma'lumot

Loyiha haqida qo'shimcha ma'lumot olish uchun Django hujjatlarini ko'ring: [https://docs.djangoproject.com/](https://docs.djangoproject.com/)
