FROM nikolaik/python-nodejs:python3.10-nodejs14

# changing user `pn` to `node`
RUN usermod --login node --move-home --home /home/node pn
RUN groupmod --new-name node pn

ARG N8N_VERSION

RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi

RUN \
	apt-get update && \
	apt-get -y install graphicsmagick git tzdata tini su-exec

# Install python requirements
RUN mkdir /requirements
COPY requirements.txt /requirements/requirements.txt
RUN python -m pip install --upgrade pip setuptools wheel
RUN pip install -r /requirements/requirements.txt

# Set a custom user to not have n8n run as root
USER root

RUN apt-get update && \
    apt-get install -y build-essential ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
RUN rm -rf /root /tmp/* && mkdir /root
RUN apt-get remove -y build-essential

RUN npm_config_user=root npm install -g full-icu n8n@${N8N_VERSION}

# Install fonts
RUN apk --no-cache add --virtual fonts msttcorefonts-installer fontconfig && \
	update-ms-fonts && \
	fc-cache -f && \
	apk del fonts && \
	find  /usr/share/fonts/truetype/msttcorefonts/ -type l -exec unlink {} \; \
	&& rm -rf /root /tmp/* /var/cache/apk/* && mkdir /root

RUN apt-get update && \
    apt-get install -y fontconfig msttcorefonts && \
    update-ms-fonts && \
    fc-cache -f && \
    apt-get remove -y msttcorefonts && \
    find /usr/share/fonts/truetype/msttcorefonts/ -type l -exec unlink {} \; && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /root /tmp/*
RUN mkdir /root

ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD true

ENV NODE_ICU_DATA /usr/lib/node_modules/full-icu

RUN apt-get update && \
    apt-get install -y nmap && \
    echo "deb http://deb.debian.org/debian sid main" >> /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y \
        chromium \
        libharfbuzz0b \
        libfreetype6 \
        fonts-freefont-ttf \
        libnss3 \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install n8n-nodes-python module
RUN cd /usr/lib/node_modules/n8n && npm install n8n-nodes-python && npm install n8n-nodes-puppeteer

WORKDIR /data

COPY docker-entrypoint.sh /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 5678/tcp
