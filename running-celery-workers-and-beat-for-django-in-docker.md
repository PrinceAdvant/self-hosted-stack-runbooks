# Running Celery workers and beat for Django in Docker

Django apps often need background work: sending email, generating reports, calling slow APIs. Celery is the usual choice. In Docker, the cleanest layout runs the web server, workers and the scheduler as separate services from the same image.

## The services

```
services:
  web:
    image: ghcr.io/acme/django-app:1.8.0
    command: gunicorn config.wsgi:application --bind 0.0.0.0:8000
  worker:
    image: ghcr.io/acme/django-app:1.8.0
    command: celery -A config worker --loglevel=info --concurrency=2
  beat:
    image: ghcr.io/acme/django-app:1.8.0
    command: celery -A config beat --loglevel=info
  redis:
    image: redis:7.4-alpine
```

One image, three commands. Deploy them together so code versions always match.

## Run exactly one beat

Beat schedules periodic tasks. Two beat processes means every scheduled task runs twice. Never scale the beat service above one replica, and do not run beat inside the worker with `-B` once you have more than one worker container.

## Concurrency

Start with concurrency equal to the CPU cores available to the worker container. For I/O-heavy tasks, more is fine; for CPU-heavy tasks, fewer and more containers can be better. Set a memory limit and `--max-tasks-per-child` so leaks are recycled.

## Avoid lost tasks on deploy

When a worker is stopped, it should finish current tasks. Docker sends SIGTERM, Celery begins a warm shutdown, and Docker kills it after the stop timeout, 10 seconds by default. Raise it for long tasks:

```
worker:
  stop_grace_period: 120s
```

Also set `task_acks_late = True` and `worker_prefetch_multiplier = 1` for important tasks, so a killed task returns to the queue instead of disappearing.

## Make tasks idempotent

With late acknowledgement, a task may run twice after a crash. Design tasks so a second run is harmless, for example by checking whether the email was already sent.

## Broker choice

Redis is simple and fine for most apps. If you use the same Redis for caching, set `noeviction` or use a separate instance, since an evicted queue key is a lost task.

## Health and visibility

- Check worker health with `celery -A config inspect ping` as a health command
- Flower gives a web view of queues and failures; keep it behind authentication

## Migrations

Run migrations once per deploy as a release step, not in the worker or beat command.

The web side of the setup, including Gunicorn, static files and settings, is covered in this guide to [deploying Django on a VPS](https://peon.sh/blogs/deploy-django-vps).

Disclosure: I help build [Peon](https://peon.sh/).
