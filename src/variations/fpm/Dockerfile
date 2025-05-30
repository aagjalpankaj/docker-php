ARG BASE_OS_VERSION='bookworm'
ARG PHP_VERSION='8.4'
ARG PHP_VARIATION='fpm'

##########
# FPM: Main Image
##########
FROM php:${PHP_VERSION}-${PHP_VARIATION}-${BASE_OS_VERSION}

ARG DEPENDENCY_PACKAGES_ALPINE='fcgi shadow'
ARG DEPENDENCY_PACKAGES_DEBIAN='libfcgi-bin procps zip'
ARG DEPENDENCY_PHP_EXTENSIONS='opcache pcntl pdo_mysql pdo_pgsql redis zip'
ARG REPOSITORY_BUILD_VERSION='dev'

LABEL org.opencontainers.image.title="serversideup/php (fpm)" \
    org.opencontainers.image.description="Supercharge your PHP experience. Based off the official PHP images, serversideup/php includes pre-configured PHP extensions and settings for enhanced performance and security. Optimized for Laravel and WordPress." \
    org.opencontainers.image.url="https://serversideup.net/open-source/docker-php/" \
    org.opencontainers.image.source="https://github.com/serversideup/docker-php" \
    org.opencontainers.image.documentation="https://serversideup.net/open-source/docker-php/docs/" \
    org.opencontainers.image.vendor="ServerSideUp" \
    org.opencontainers.image.authors="Jay Rogers (@jaydrogers)" \
    org.opencontainers.image.version="${REPOSITORY_BUILD_VERSION}" \
    org.opencontainers.image.licenses="GPL-3.0-or-later"

ENV APP_BASE_DIR=/var/www/html \
    COMPOSER_ALLOW_SUPERUSER=1 \
    COMPOSER_HOME=/composer \
    COMPOSER_MAX_PARALLEL_HTTP=24 \
    DISABLE_DEFAULT_CONFIG=false \
    LOG_OUTPUT_LEVEL=warn \
    PHP_DATE_TIMEZONE="UTC" \
    PHP_DISPLAY_ERRORS=Off \
    PHP_DISPLAY_STARTUP_ERRORS=Off \
    PHP_ERROR_LOG="/dev/stderr" \
    PHP_ERROR_REPORTING="22527" \
    PHP_FPM_PM_CONTROL=dynamic \
    PHP_FPM_PM_MAX_CHILDREN="20" \
    PHP_FPM_PM_MAX_SPARE_SERVERS="3" \
    PHP_FPM_PM_MIN_SPARE_SERVERS="1" \
    PHP_FPM_PM_START_SERVERS="2" \
    PHP_FPM_POOL_NAME="www" \
    PHP_FPM_PROCESS_CONTROL_TIMEOUT="10s" \
    PHP_MAX_EXECUTION_TIME="99" \
    PHP_MAX_INPUT_TIME="-1" \
    PHP_MEMORY_LIMIT="256M" \
    PHP_OPCACHE_ENABLE="0" \
    PHP_OPCACHE_INTERNED_STRINGS_BUFFER="8" \
    PHP_OPCACHE_MAX_ACCELERATED_FILES="10000" \
    PHP_OPCACHE_MEMORY_CONSUMPTION="128" \
    PHP_OPCACHE_REVALIDATE_FREQ="2" \
    PHP_OPEN_BASEDIR="" \
    PHP_POST_MAX_SIZE="100M" \
    PHP_SESSION_COOKIE_SECURE=Off \
    PHP_UPLOAD_MAX_FILE_SIZE="100M" \
    SHOW_WELCOME_MESSAGE=true

# copy our scripts
COPY --chmod=755 src/common/ /

# clear php configurations and install pecl extensions & dependencies
RUN rm -rf /usr/local/etc/php-fpm.d/*.conf && \
    docker-php-serversideup-dep-install-alpine "${DEPENDENCY_PACKAGES_ALPINE}" && \
    docker-php-serversideup-dep-install-debian "${DEPENDENCY_PACKAGES_DEBIAN}"  && \
    docker-php-serversideup-install-php-ext-installer && \
    \
    # Ensure /var/www/ has the correct permissions
    chown -R www-data:www-data /var/www && \
    chmod -R 755 /var/www && \
    \
    # Set the image version
    echo "${REPOSITORY_BUILD_VERSION}" > /etc/serversideup-php-version && \
    \
    # Make composer cache directory
    mkdir -p "${COMPOSER_HOME}" && \
    chown -R www-data:www-data "${COMPOSER_HOME}" && \
    \
    # Ensure "php-fpm-healthcheck" is installed
    echo "⬇️ Downloading php-fpm-healthcheck..." && \
    curl -o /usr/local/bin/php-fpm-healthcheck https://raw.githubusercontent.com/renatomefi/php-fpm-healthcheck/v0.5.0/php-fpm-healthcheck && \
    chmod +x /usr/local/bin/php-fpm-healthcheck && \
    \
    # Install default PHP extensions
    install-php-extensions "${DEPENDENCY_PHP_EXTENSIONS}"

# copy our fpm configurations
COPY src/php-fpm.d/ /

# install composer from Composer's official Docker image
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

RUN docker-php-serversideup-set-file-permissions --owner www-data:www-data --service fpm

ENTRYPOINT ["docker-php-serversideup-entrypoint"]

STOPSIGNAL SIGQUIT

EXPOSE 9000

WORKDIR ${APP_BASE_DIR}

USER www-data

CMD ["php-fpm"]

HEALTHCHECK --interval=5s --timeout=3s --retries=3 \
    CMD php-fpm-healthcheck || exit 1
