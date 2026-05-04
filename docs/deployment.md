# Render Deployment

Target domain: `10c.edwards.phd`

Recommended free/low-cost stack:

- Render Free Web Service for the Django app.
- Supabase Free Postgres for the database.
- Cloudinary Free later for user-uploaded media. The current app only uses committed static guide illustrations, so Cloudinary is optional for this launch.

Keep the GitHub repository private if it includes `source_materials/`, because those PDFs are exam-board resources.

## 1. Push The Project

```bash
.venv/bin/python manage.py test
.venv/bin/python manage.py check
git add .
git commit -m "Prepare GCSE VLE for Render"
git push origin main
```

## 2. Create Supabase Database

1. Create a Supabase project.
2. Open Project Settings > Database.
3. Copy the direct PostgreSQL connection string.
4. Use the pooled connection string if Supabase recommends it for hosted apps.
5. Keep the password safe; it goes into Render as `DATABASE_URL`.

Render env value:

```bash
DATABASE_URL=postgresql://...
DATABASE_SSL_REQUIRE=True
```

## 3. Create Render Web Service

1. In Render, create a new Web Service from the GitHub repo.
2. Select the free instance type.
3. Use these commands:

```bash
Build Command: ./build.sh
Start Command: gunicorn config.wsgi:application
```

The repo also includes `render.yaml`, so Render can import the same values as a Blueprint if preferred.

## 4. Render Environment Variables

Set these in Render:

```bash
DJANGO_SECRET_KEY=replace-with-a-long-random-secret
DJANGO_DEBUG=False
DJANGO_ALLOWED_HOSTS=10c.edwards.phd,gcse-exam-revision.onrender.com
DJANGO_CSRF_TRUSTED_ORIGINS=https://10c.edwards.phd,https://gcse-exam-revision.onrender.com
DJANGO_SECURE_SSL_REDIRECT=True
DJANGO_SESSION_COOKIE_SECURE=True
DJANGO_CSRF_COOKIE_SECURE=True
DATABASE_URL=postgresql://...
DATABASE_SSL_REQUIRE=True
QUESTION_GENERATOR_PROVIDER=local
QUESTION_GENERATOR_API_KEY=
```

Generate a secret key locally with:

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

## 5. Custom Domain

In Render:

1. Open the Web Service.
2. Go to Settings > Custom Domains.
3. Add `10c.edwards.phd`.
4. Render will show the DNS record to create.

At your DNS provider, add the record Render gives you. For a subdomain this is usually a `CNAME`, for example:

```text
10c -> your-render-service.onrender.com
```

Render provisions HTTPS automatically once DNS verifies.

## 6. First Launch Checks

After deploy finishes, open:

```text
https://10c.edwards.phd/en/
```

Check:

- Home page loads.
- Login works.
- `student`, `teacher`, and `admin` demo users exist.
- Past-paper practice pages load.
- Static guide illustrations load.
- Generated questions work with the local generator.

## 7. Updating Later

Push to `main`; Render will rebuild automatically.

The build script runs:

```bash
collectstatic
migrate
import_past_papers --replace
seed_demo
```

That means schema changes, static files, and the structured WJEC question bank are refreshed on each deployment.
