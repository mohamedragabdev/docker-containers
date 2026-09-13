# Docker Notes and Laravel/MySQL Starter

مجموعة ملاحظات عملية لتعلّم Docker وDockerfile وDocker Compose، مع إعداد ابتدائي لتشغيل تطبيق Laravel بجانب قاعدة بيانات MySQL.

> هذا المستودع مرجع تعليمي وقالب بداية. ملف Compose الحالي ليس مشروع Laravel مكتملًا ولا يعمل مباشرة بدون ربطه بمشروع Laravel حقيقي وتعديل القيم الموضحة أدناه.

## المحتويات

- شرح أوامر Docker الأساسية وإدارة الصور والحاويات.
- شرح تعليمات Dockerfile مع أمثلة عملية.
- شرح مفاهيم Docker Compose مثل الخدمات، الشبكات، المنافذ، والـ volumes.
- Dockerfile مبني على PHP لتجهيز بيئة Laravel.
- ملف Compose يعرّف خدمتي تطبيق الويب وMySQL.

## هيكل المستودع

```text
.
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yaml
├── docker-notes.md
├── docker_compose_notes_general.md
├── dockerfileSt.md
└── README.md
```

## المتطلبات

- Docker Engine أو Docker Desktop.
- Docker Compose v2، ويُستخدم من خلال الأمر `docker compose`.
- مشروع Laravel محلي إذا أردت تشغيل خدمة `web-laravel` فعليًا.

تحقق من التثبيت:

```bash
docker --version
docker compose version
docker info
```

في Linux يمكن إضافة المستخدم إلى مجموعة Docker لتشغيل الأوامر بدون `sudo`:

```bash
sudo usermod -aG docker "$USER"
```

بعد ذلك سجّل الخروج ثم ادخل مرة أخرى حتى يصبح التغيير فعالًا.

## استخدام ملفات الملاحظات

ابدأ بالترتيب التالي:

1. [docker-notes.md](docker-notes.md): الصور والحاويات وأوامر `run` و`start` والمنافذ.
2. [dockerfileSt.md](dockerfileSt.md): تعليمات Dockerfile مثل `FROM` و`RUN` و`COPY` و`CMD` و`ENTRYPOINT`.
3. [docker_compose_notes_general.md](docker_compose_notes_general.md): الخدمات والشبكات والـ volumes والتواصل بين الحاويات.

## إعداد Laravel وMySQL

الملف [docker/Dockerfile](docker/Dockerfile) يقوم بالآتي:

- استخدام صورة PHP.
- ضبط مجلد العمل على `/app`.
- تثبيت `zip` و`unzip`.
- تثبيت Composer.
- تفعيل إضافة `pdo_mysql`.
- تشغيل Laravel على المنفذ `8000`.

والملف [docker/docker-compose.yaml](docker/docker-compose.yaml) يعرّف:

- `web-laravel`: خدمة تُبنى من Dockerfile وتُنشر على `localhost:8000`.
- `mysql-db`: قاعدة بيانات MySQL 8.0.
- شبكة مشتركة باسم `backend-net`.
- volume باسم `db-vol` لبيانات قاعدة البيانات.

### قبل التشغيل

عدّل ملف `docker/docker-compose.yaml` أولًا:

1. استبدل `/path_to_your_project` بالمسار الحقيقي لمشروع Laravel.
2. استبدل قيم `ROOT_PASSWORD` و`DATABASE_NAME` و`USER_NAME` و`USER_PASSWORDNAME` بقيم فعلية، ويفضل وضع الأسرار في ملف `.env` غير مرفوع إلى GitHub.
3. صحّح مسار MySQL من `/var/lib/musql` إلى `/var/lib/mysql` حتى يتم حفظ البيانات في الـ volume بشكل صحيح.
4. تأكد من أن مشروع Laravel يحتوي على `artisan` داخل المجلد المربوط إلى `/app`.
5. اضبط إعدادات قاعدة البيانات في ملف Laravel `.env`، مثل `DB_HOST=mysql-db` و`DB_PORT=3306`، لأن اسم الخدمة هو اسم المضيف داخل شبكة Compose.

مثال لإعدادات Laravel:

```dotenv
DB_CONNECTION=mysql
DB_HOST=mysql-db
DB_PORT=3306
DB_DATABASE=your_database
DB_USERNAME=your_user
DB_PASSWORD=your_password
```

### البناء والتشغيل

من مجلد المستودع:

```bash
docker compose -f docker/docker-compose.yaml up --build
```

بعد نجاح التشغيل افتح:

```text
http://localhost:8000
```

لتشغيل الخدمات في الخلفية:

```bash
docker compose -f docker/docker-compose.yaml up --build -d
```

لعرض الحالة والسجلات:

```bash
docker compose -f docker/docker-compose.yaml ps
docker compose -f docker/docker-compose.yaml logs -f
```

لإيقاف الخدمات مع الإبقاء على بيانات MySQL:

```bash
docker compose -f docker/docker-compose.yaml down
```

لحذف الخدمات والـ volume والبيانات نهائيًا:

```bash
docker compose -f docker/docker-compose.yaml down -v
```

## بناء Dockerfile بشكل منفصل

للبناء من مجلد `docker`:

```bash
cd docker
docker build -t laravel-php-starter .
```

لتشغيل الصورة بعد ربط مشروع Laravel:

```bash
docker run --rm -it \
  -p 8000:8000 \
  -v /absolute/path/to/laravel-project:/app \
  laravel-php-starter
```

## أوامر Docker سريعة

```bash
# عرض الصور
docker image ls

# عرض الحاويات العاملة
docker container ls

# عرض كل الحاويات
docker container ls -a

# إنشاء حاوية بدون تشغيلها
docker container create -it ubuntu bash

# تشغيل حاوية تفاعلية
docker run --name my-ubuntu -it ubuntu bash

# تشغيل Nginx مع نشر منفذ
docker run -d --name my-nginx -p 8080:80 nginx
```

## ملاحظات أمنية وتشغيلية

- لا ترفع كلمات المرور أو مفاتيح الإنتاج إلى GitHub.
- لا تستخدم قيمًا مثل `ROOT_PASSWORD` في بيئة إنتاجية.
- استخدم ملف `.env` أو مدير أسرار، وأضف ملفات الأسرار إلى `.gitignore`.
- صورة PHP في Dockerfile غير مثبتة على إصدار محدد؛ تثبيت إصدار واضح يجعل البناء أكثر قابلية للتكرار.
- يُفضّل تشغيل التطبيق بمستخدم غير `root` عند تجهيز بيئة إنتاجية.
- `EXPOSE` يصف المنفذ فقط ولا ينشره إلى الجهاز المضيف؛ النشر يتم من خلال `-p` أو `ports` في Compose.

## استكشاف الأخطاء

### المنفذ 8000 مستخدم

غيّر منفذ الجهاز المضيف فقط:

```yaml
ports:
  - "8001:8000"
```

ثم استخدم `http://localhost:8001`.

### لا يوجد ملف `artisan`

تحقق من أن مسار المشروع الموجود في bind mount صحيح وأنه يحتوي على مشروع Laravel كامل.

### التطبيق لا يصل إلى MySQL

من داخل شبكة Compose استخدم اسم الخدمة `mysql-db` بدلًا من `localhost`، وتأكد من تطابق بيانات الاتصال بين Compose وملف Laravel `.env`.

### فحص ملف Compose قبل التشغيل

```bash
docker compose -f docker/docker-compose.yaml config
```

## الهدف من المستودع

الهدف هو بناء فهم عملي للعلاقة بين:

```text
Dockerfile  -> بناء Image
Compose     -> تشغيل وتنسيق Services
Container   -> نسخة تشغيل من Image
Volume      -> حفظ البيانات خارج دورة حياة Container
Network     -> ربط Services ببعضها
```

## الترخيص

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.