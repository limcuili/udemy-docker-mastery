# create your custom drupal image here, based of official drupal
FROM drupal:8.8.2

# the "-y" when installing git is to specify that you're saying yes and thus won't get a prompt when installing git
# recall that the backslash and ampersands (\ &&) says "if the first command succeeds, then also do this one."
RUN apt-get update \
    && apt-get install -y git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html/themes

# docker runs everything at root and thus these git files are downloaded to root.
# Apache expects it somewhere else so we move them with "chown -R" (change owner of all directories and subfiles).
RUN git clone --branch 8.x-3.x --single-branch --depth 1 https://git.drupal.org/project/bootstrap.git \
    && chown -R www-data:www-data bootstrap

WORKDIR /var/www/html