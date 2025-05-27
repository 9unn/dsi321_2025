FROM python:3.11-slim

ENV DEBIAN_FRONTEND=noninteractive \
    APP_HOME=/home/prefect_user/app \
    CHROME_BIN=/usr/bin/chromium \
    CHROMEDRIVER_PATH=/usr/local/bin/chromedriver

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    chromium \
    chromium-driver \
    libglib2.0-0 libnss3 libnspr4 libdbus-1-3 libatk1.0-0 libatk-bridge2.0-0 \
    libcups2 libdrm2 libexpat1 libgbm1 libpango-1.0-0 libx11-6 libx11-xcb1 \
    libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 \
    libxi6 libxrandr2 libxrender1 libxss1 libxtst6 fonts-liberation \
    procps && \
    ln -s /usr/bin/chromium /usr/bin/google-chrome && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN groupadd --gid 1000 prefect_group && \
    useradd --uid 1000 --gid prefect_group -m prefect_user && \
    mkdir -p ${APP_HOME} && chown -R prefect_user:prefect_group ${APP_HOME}

USER prefect_user
WORKDIR ${APP_HOME}

COPY --chown=prefect_user:prefect_group requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt && \
    pip install --no-cache-dir --user "prefect==3.4.1" selenium webdriver-manager lakefs-client pyarrow

ENV PATH="/home/prefect_user/.local/bin:${PATH}"

COPY --chown=prefect_user:prefect_group . .