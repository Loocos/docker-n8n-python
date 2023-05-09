FROM nikolaik/python-nodejs:python3.10-nodejs16

# changing user `pn` to `node`
RUN usermod --login node --move-home --home /home/node pn
RUN groupmod --new-name node pn

ARG N8N_VERSION

RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi

RUN apt-get update && apt-get -y install graphicsmagick gosu git tini
RUN DEBIAN_FRONTEND=noninteractive TZ=Europe/Paris apt-get -y install tzdata

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
RUN apt-get update && \
    apt-get install -y wget \
    fontconfig && \
    wget http://ftp.br.debian.org/debian/pool/contrib/m/msttcorefonts/ttf-mscorefonts-installer_3.8_all.deb && \
    apt --fix-broken install -y ./ttf-mscorefonts-installer_3.8_all.deb && \
    rm ttf-mscorefonts-installer_3.8_all.deb && \
    fc-cache -f -v
RUN rm -rf /root /tmp/* && mkdir /root

ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD true

ENV NODE_ICU_DATA /usr/lib/node_modules/full-icu

#RUN apt-get update && \
#     apt-get install -y nmap && \
#     echo "deb http://deb.debian.org/debian sid main" >> /etc/apt/sources.list && \
#     apt-get update && \
#     apt-get install -y \
#         chromium \
#         libharfbuzz0b \
#         libfreetype6 \
#         fonts-freefont-ttf \
#         libnss3 \
#     && apt-get clean && \
#     rm -rf /var/lib/apt/lists/*
RUN apt-get update && apt-get install -y \
      chromium \
      chromium-l10n \
      fonts-liberation \
      fonts-roboto \
      hicolor-icon-theme \
      libcanberra-gtk-module \
      libexif-dev \
      libgl1-mesa-dri \
      libgl1-mesa-glx \
      libpangox-1.0-0 \
      libv4l-0 \
      fonts-symbola \
      --no-install-recommends \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir -p /etc/chromium.d/ \
    && /bin/echo -e 'export GOOGLE_API_KEY="AIzaSyCkfPOPZXDKNn8hhgu3JrA62wIgC93d44k"\nexport GOOGLE_DEFAULT_CLIENT_ID="811574891467.apps.googleusercontent.com"\nexport GOOGLE_DEFAULT_CLIENT_SECRET="kdloedMFGdGla2P1zacGjAQh"' > /etc/chromium.d/googleapikeys


# Install n8n-nodes-python module
RUN cd /usr/lib/node_modules/n8n && npm install n8n-nodes-python && npm install n8n-nodes-puppeteer && npm install jsonrepair

WORKDIR /data

COPY docker-entrypoint.sh /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 5678/tcp
