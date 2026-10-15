# PulseNotify

Flight price alert backend: Django + DRF + SimpleJWT, PostgreSQL, Redis, Celery (worker + Beat).

## Setup

```bash
# 1. Start PostgreSQL + Redis
docker compose up -d

# 2. Create a virtualenv and install dependencies
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt

# 3. Environment
#    .env holds SECRET_KEY, DB credentials and DJANGO_SETTINGS_MODULE.
#    It is git-ignored - never commit it.
#    Use DJANGO_SETTINGS_MODULE=pulsenotify.settings.production when deploying.

# 4. Migrate
python manage.py migrate
```

## Run (three processes, three terminals)

```bash
# Terminal 1 - API server
python manage.py runserver

# Terminal 2 - Celery worker (send_notification tasks arrive here)
celery -A pulsenotify worker --loglevel=info

# Terminal 3 - Celery Beat (check_prices fires here every 60 seconds)
celery -A pulsenotify beat --loglevel=info
```

On Windows, start the worker with `celery -A pulsenotify worker --loglevel=info --pool=solo`.

## Endpoints

| Method | Path | Auth |
|--------|------|------|
| POST | `/api/auth/register/` | public |
| POST | `/api/auth/login/` | public |
| POST | `/api/alerts/` | JWT |
| GET | `/api/alerts/` | JWT |
| DELETE | `/api/alerts/<id>/` | JWT |
| GET | `/api/flights/price/?route=DEL-BOM` | none (internal mock feed) |
| GET | `/api/admin/summary/` | JWT, admin role only |
attached screen shots *.png for post man requests
Send the token as `Authorization: Bearer <access>`.

## Making a user an admin

Set `role` to `admin` on the user's row in the `pulse_userprofile` table (e.g. in PGAdmin), then log in again.

## Tests

```bash
python manage.py test
```

## Postman

Import `PulseNotify.postman_collection.json`. Set the `admin_username` / `admin_password` collection variables to a user whose role you changed to `admin`, then run the collection in order.
