# syntax = docker/dockerfile:1
FROM python:3.13-slim-bookworm AS development_build

ARG DJANGO_ENV=development \
    UID=1000 \
    GID=1000

ENV DJANGO_ENV=${DJANGO_ENV} \
    PYTHONFAULTHANDLER=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONHASHSEED=random \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PROJECT_ENVIRONMENT=/usr/local \
    UV_CACHE_DIR='/var/cache/uv'

SHELL ["/bin/bash", "-eo", "pipefail", "-c"]

RUN apt-get update && apt-get upgrade -y \
    && apt-get install --no-install-recommends -y \
        bash \
        curl \
        gettext \
        git \
        libpq-dev \
        ffmpeg \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/*

COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

WORKDIR /code

RUN groupadd -g "${GID}" -r paprika \
    && useradd -d '/code' -g paprika -l -r -u "${UID}" paprika \
    && chown paprika:paprika -R '/code' \
    && mkdir -p '/var/www/paprika/static' '/var/www/paprika/media' \
    && chown paprika:paprika '/var/www/paprika/static' '/var/www/paprika/media'

COPY --chown=paprika:paprika ./pyproject.toml ./uv.lock /code/

RUN --mount=type=cache,target="$UV_CACHE_DIR" \
    echo "Building for environment: $DJANGO_ENV" \
    && uv sync \
    $(if [ "$DJANGO_ENV" = 'production' ]; then echo '--no-dev'; fi)

USER paprika

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

FROM development_build AS production_build

COPY --chown=paprika:paprika . /code

EXPOSE 8000

CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "4", "--threads", "2"]
