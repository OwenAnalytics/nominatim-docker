ARG nominatim_version=3.6.0

FROM amazonlinux:latest as builder

ARG nominatim_version

# Let the container know that there is no TTY
ARG DEBIAN_FRONTEND=noninteractive

# Install packages
RUN yum -y update \
 && yum install -y amazon-linux-extras \
 && yum install -y  \
	postgresql12-contrib postgresql12-devel postgis30_12 \
        wget git cmake make gcc gcc-c++ libtool policycoreutils-python-utils \
        llvm-toolset ccache clang-tools-extra \
        php-pgsql php php-intl php-json libpq-devel \
        bzip2-devel proj-devel boost-devel \
        python3-pip python3-setuptools python3-devel \
        expat-devel zlib-devel \
    php \
    curl

# Build Nominatim
RUN cd /srv \
 && curl --silent -L http://www.nominatim.org/release/Nominatim-${nominatim_version}.tar.bz2 -o v${nominatim_version}.tar.bz2 \
 && tar xf v${nominatim_version}.tar.bz2 \
 && rm v${nominatim_version}.tar.bz2 \
 && mv Nominatim-${nominatim_version} nominatim \
 && cd nominatim \
 && mkdir build \
 && cd build \
 && cmake .. \
 && make


FROM amazonlinux:latest

ARG nominatim_version

LABEL \
  org.opencontainers.image.title="nominatim" \
  org.opencontainers.image.description="Docker image for Nominatim." \
  org.opencontainers.image.url="https://github.com/owenanalytics/nominatim-docker" \
  org.opencontainers.image.licenses="MIT" \
  app.tag="nominatim${nominatim_version}"

# Let the container know that there is no TTY
ARG DEBIAN_FRONTEND=noninteractive

# Set locale and install packages
ENV LANG C.UTF-8
RUN yum  -y update \
 && yum install -y locales \
 && locale-gen en_US.UTF-8 \
 && update-locale LANG=en_US.UTF-8 \
 && t install -y -qq --no-install-recommends \
    postgresql-server-dev-12 \
    postgresql-12-postgis-3 \
    postgresql-12-postgis-3-scripts \
    postgresql-contrib-12 \
    apache2 \
    php \
    php-pgsql \
    libapache2-mod-php \
    php-pear \
    php-db \
    php-intl \
    python3-dev \
    python3-psycopg2 \
    curl \
    ca-certificates \
    libboost-system\
    libboost-filesystem \
    sudo \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* \
 && rm -rf /tmp/* /var/tmp/*

# Copy the application from the builder image
COPY --from=builder /srv/nominatim /srv/nominatim

# Configure Nominatim
COPY local.php /srv/nominatim/build/settings/local.php

# Configure Apache
COPY nominatim.conf /etc/apache2/sites-enabled/000-default.conf

# Allow remote connections to PostgreSQL
RUN echo "host all  all    0.0.0.0/0  trust" >> /etc/postgresql/12/main/pg_hba.conf \
 && echo "listen_addresses='*'" >> /etc/postgresql/12/main/postgresql.conf

# Set the entrypoint
COPY docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 5432
EXPOSE 8080
