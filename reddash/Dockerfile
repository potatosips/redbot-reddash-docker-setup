FROM python:3.11-slim-bookworm

ARG REDDASH_VERSION=1.8.2

RUN useradd --create-home --uid 1000 --shell /usr/sbin/nologin reddash \
    && python -m pip install --no-cache-dir --upgrade pip setuptools wheel \
    && python -m pip install --no-cache-dir "Red-Web-Dashboard==${REDDASH_VERSION}"

USER reddash
WORKDIR /home/reddash
ENTRYPOINT ["reddash"]
