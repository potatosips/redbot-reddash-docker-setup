FROM python:3.11-slim-bookworm

# Update this only through the controlled update procedure in docs/05.
ARG REDBOT_VERSION=3.5.24

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates chromium default-jre-headless ffmpeg git tini \
        libasound2 libatk-bridge2.0-0 libatk1.0-0 libcairo2 libcups2 \
        libdrm2 libffi-dev libgbm1 libglib2.0-0 libgtk-3-0 libnacl-dev \
        libnss3 libopus-dev libpango-1.0-0 libpangocairo-1.0-0 \
        libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxdamage1 \
        libxext6 libxfixes3 libxkbcommon0 libxrandr2 libsodium-dev \
    && rm -rf /var/lib/apt/lists/*

RUN useradd --create-home --uid 1000 --shell /usr/sbin/nologin redbot \
    && python -m pip install --no-cache-dir --upgrade pip wheel \
    && python -m pip install --no-cache-dir "Red-DiscordBot[postgres]==${REDBOT_VERSION}"

USER redbot
WORKDIR /home/redbot
ENV HOME=/home/redbot \
    XDG_CONFIG_HOME=/home/redbot/.chromium-config \
    XDG_CACHE_HOME=/home/redbot/.chromium-cache \
    XDG_DATA_HOME=/home/redbot/.chromium-data \
    BROWSER_PATH=/usr/bin/chromium

ENTRYPOINT ["/usr/bin/tini", "--", "redbot"]
