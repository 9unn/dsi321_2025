FROM python:3.11-slim

ENV DEBIAN_FRONTEND=noninteractive \
    APP_HOME=/home/prefect_user/app \
    CHROMEDRIVER_PATH=/usr/local/bin/chromedriver

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libglib2.0-0 libnss3 libnspr4 libdbus-1-3 libatk1.0-0 libatk-bridge2.0-0 \
    libcups2 libdrm2 libexpat1 libgbm1 libpango-1.0-0 libx11-6 libx11-xcb1 \
    libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 \
    libxi6 libxrandr2 libxrender1 libxss1 libxtst6 fonts-liberation \
    lsb-release xdg-utils \
    wget gnupg unzip procps jq ca-certificates && \
    wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/google-chrome-archive-keyring.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome-archive-keyring.gpg] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends google-chrome-stable && \
    CHROMEDRIVER_DOWNLOAD_URL=$(wget -qO- https://googlechromelabs.github.io/chrome-for-testing/last-known-good-versions-with-downloads.json | jq -r '.channels.Stable.downloads.chromedriver[] | select(.platform=="linux64") | .url') && \
    echo "Downloading ChromeDriver from: ${CHROMEDRIVER_DOWNLOAD_URL}" && \
    wget -q "${CHROMEDRIVER_DOWNLOAD_URL}" -O chromedriver_linux64.zip && \
    unzip chromedriver_linux64.zip -d /tmp/chromedriver_temp && \
    if [ -f /tmp/chromedriver_temp/chromedriver-linux64/chromedriver ]; then \
        mv /tmp/chromedriver_temp/chromedriver-linux64/chromedriver ${CHROMEDRIVER_PATH}; \
    elif [ -f /tmp/chromedriver_temp/chromedriver ]; then \
        mv /tmp/chromedriver_temp/chromedriver ${CHROMEDRIVER_PATH}; \
    else \
        echo "Error: chromedriver not found in expected location after unzip." && exit 1; \
    fi && \
    chmod +x ${CHROMEDRIVER_PATH} && \
    rm chromedriver_linux64.zip && \
    rm -rf /tmp/chromedriver_temp && \
    apt-get purge -y --auto-remove wget gnupg unzip jq && \
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
