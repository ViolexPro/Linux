FROM php:8.3-apache

# Установка системных зависимостей
RUN apt-get update && apt-get install -y \
    wget \
    unzip \
    libzip-dev \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libxml2-dev \
    libcurl4-openssl-dev \
    libonig-dev \
    libldap2-dev \
    libssl-dev \
    libicu-dev \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Установка PHP расширений
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
    mysqli pdo pdo_mysql zip gd xml curl mbstring soap ldap intl

RUN a2enmod rewrite

# Скачивание Moodle через wget
WORKDIR /var/www
RUN wget https://github.com/moodle/moodle/archive/MOODLE_405_STABLE.zip && \
    unzip MOODLE_405_STABLE.zip && \
    mv moodle-MOODLE_405_STABLE moodle && \
    rm MOODLE_405_STABLE.zip

# Создание директорий
RUN mkdir -p /var/www/moodledata /var/www/moodle/sessions

# Установка прав
RUN chown -R www-data:www-data /var/www/moodle /var/www/moodledata && \
    chmod -R 755 /var/www/moodle && \
    chmod -R 777 /var/www/moodledata && \
    chmod -R 755 /var/www/moodle/sessions

# Настройка PHP
RUN echo 'max_input_vars = 5000' > /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'max_execution_time = 600' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'memory_limit = 256M' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'post_max_size = 500M' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'upload_max_filesize = 500M' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'date.timezone = Europe/Moscow' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'session.save_handler = files' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'session.save_path = /var/www/moodle/sessions' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'session.use_strict_mode = 1' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'session.use_cookies = 1' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'session.use_only_cookies = 1' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'session.cookie_httponly = 1' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'session.cookie_samesite = Lax' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'session.gc_maxlifetime = 86400' >> /usr/local/etc/php/conf.d/moodle.ini && \
    echo 'session.cookie_lifetime = 86400' >> /usr/local/etc/php/conf.d/moodle.ini

# Создание .htaccess
RUN echo '<IfModule mod_rewrite.c>' > /var/www/moodle/.htaccess && \
    echo '    RewriteEngine On' >> /var/www/moodle/.htaccess && \
    echo '    RewriteCond %{REQUEST_FILENAME} !-d' >> /var/www/moodle/.htaccess && \
    echo '    RewriteCond %{REQUEST_FILENAME} !-f' >> /var/www/moodle/.htaccess && \
    echo '    RewriteRule ^(.*)$ index.php [QSA,L]' >> /var/www/moodle/.htaccess && \
    echo '</IfModule>' >> /var/www/moodle/.htaccess

# Настройка виртуального хоста Apache (только HTTP, без HTTPS)
RUN echo '<VirtualHost *:80>' > /etc/apache2/sites-available/000-default.conf && \
    echo '    DocumentRoot /var/www/moodle' >> /etc/apache2/sites-available/000-default.conf && \
    echo '    <Directory /var/www/moodle>' >> /etc/apache2/sites-available/000-default.conf && \
    echo '        Options Indexes FollowSymLinks MultiViews' >> /etc/apache2/sites-available/000-default.conf && \
    echo '        AllowOverride All' >> /etc/apache2/sites-available/000-default.conf && \
    echo '        Require all granted' >> /etc/apache2/sites-available/000-default.conf && \
    echo '    </Directory>' >> /etc/apache2/sites-available/000-default.conf && \
    echo '</VirtualHost>' >> /etc/apache2/sites-available/000-default.conf

# Создание config.php с переменными окружения
RUN echo '<?php' > /var/www/moodle/config.php && \
    echo 'unset($CFG);' >> /var/www/moodle/config.php && \
    echo 'global $CFG;' >> /var/www/moodle/config.php && \
    echo '$CFG = new stdClass();' >> /var/www/moodle/config.php && \
    echo '' >> /var/www/moodle/config.php && \
    echo '// Database' >> /var/www/moodle/config.php && \
    echo '$CFG->dbtype    = "mariadb";' >> /var/www/moodle/config.php && \
    echo '$CFG->dblibrary = "native";' >> /var/www/moodle/config.php && \
    echo '$CFG->dbhost    = getenv("MOODLE_DB_HOST") ?: "mariadb";' >> /var/www/moodle/config.php && \
    echo '$CFG->dbname    = getenv("MOODLE_DB_NAME") ?: "moodle";' >> /var/www/moodle/config.php && \
    echo '$CFG->dbuser    = getenv("MOODLE_DB_USER") ?: "moodleuser";' >> /var/www/moodle/config.php && \
    echo '$CFG->dbpass    = getenv("MOODLE_DB_PASSWORD") ?: "your_strong_password";' >> /var/www/moodle/config.php && \
    echo '$CFG->prefix    = "mdl_";' >> /var/www/moodle/config.php && \
    echo '' >> /var/www/moodle/config.php && \
    echo '// Paths' >> /var/www/moodle/config.php && \
    echo '$CFG->wwwroot   = getenv("MOODLE_WWWROOT") ?: "https://moodle.pzsp.ru";' >> /var/www/moodle/config.php && \
    echo '$CFG->dataroot  = getenv("MOODLE_DATAROOT") ?: "/var/www/moodledata";' >> /var/www/moodle/config.php && \
    echo '$CFG->directorypermissions = 0777;' >> /var/www/moodle/config.php && \
    echo '' >> /var/www/moodle/config.php && \
    echo '// SSL/Proxy settings (без reverseproxy!)' >> /var/www/moodle/config.php && \
    echo '$CFG->sslproxy = true;' >> /var/www/moodle/config.php && \
    echo '' >> /var/www/moodle/config.php && \
    echo '// Session settings' >> /var/www/moodle/config.php && \
    echo '$CFG->sessiontimeout = 86400;' >> /var/www/moodle/config.php && \
    echo '$CFG->sessioncookie = "MOODLEID";' >> /var/www/moodle/config.php && \
    echo '$CFG->samesitecookie = "Lax";' >> /var/www/moodle/config.php && \
    echo '$CFG->cookiehttponly = true;' >> /var/www/moodle/config.php && \
    echo '$CFG->cookiesecure = true;' >> /var/www/moodle/config.php && \
    echo '$CFG->cookiepath = "/";' >> /var/www/moodle/config.php && \
    echo '' >> /var/www/moodle/config.php && \
    echo '// Performance' >> /var/www/moodle/config.php && \
    echo '$CFG->debug = 0;' >> /var/www/moodle/config.php && \
    echo '$CFG->debugdisplay = 0;' >> /var/www/moodle/config.php && \
    echo '' >> /var/www/moodle/config.php && \
    echo 'require_once(__DIR__ . "/lib/setup.php");' >> /var/www/moodle/config.php

RUN chown www-data:www-data /var/www/moodle/config.php /var/www/moodle/.htaccess && \
    chmod 644 /var/www/moodle/config.php && \
    chmod 644 /var/www/moodle/.htaccess

WORKDIR /var/www/moodle
EXPOSE 80
CMD ["apache2-foreground"]
