FROM php:7.2-apache

# iniファイルの読込
COPY ./docker/app/php.ini /usr/local/etc/php/

# Composerインストール
COPY --from=composer:2.6 /usr/bin/composer /usr/bin/composer

# Middlewareインストール
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    npm \
    gnupg \
    ca-certificates \
    curl \
    git \
    vim \
    zip \
    unzip \
    libzip-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libonig-dev \
    libssl-dev \
    default-libmysqlclient-dev \
    libmagickwand-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Apacheの設定ファイルを読み込む
COPY ./docker/app/apache2/000-default.conf /etc/apache2/sites-available/
COPY ./docker/app/apache2/default-ssl.conf /etc/apache2/sites-available/
COPY ./docker/app/apache2/fqdn.conf /etc/apache2/conf-available/
RUN a2enconf fqdn \
    && a2enmod rewrite

# pdo_mysql拡張モジュールのインストール（mysqlとの接続が可能に）
RUN docker-php-ext-install pdo_mysql mysqli

# GD拡張モジュールのインストール
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

# ext-exif拡張機能のインストール
RUN docker-php-ext-install exif

# Imagickのインストール
RUN pecl install imagick \
    && docker-php-ext-enable imagick

# PHP 7.2に対応したXdebugをインストール
RUN pecl install xdebug-2.9.8 \
    && docker-php-ext-enable xdebug

# npmパッケージ / vite のインストール
RUN npm install npm@9.5.1 -g cross-env \
    && npm install -g vite

# Xdebugの設定ファイルを作成
RUN echo "zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20170718/xdebug.so" > /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# laravelに必要なmodRewriteの有効化
RUN a2enmod rewrite

EXPOSE 80 443 5173 