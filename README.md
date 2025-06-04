[![Bugsink](https://img.shields.io/badge/Bugsink-1.5.4-blue.svg)](https://github.com/bugsink/bugsink/releases/tag/1.5.4)
[![Dokku](https://img.shields.io/badge/Dokku-Repo-blue.svg)](https://github.com/dokku/dokku)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/d1ceward-on-dokku/bugsink_on_dokku/graphs/commit-activity)

# Run Bugsink on Dokku

## Overview

This guide explains how to deploy [Bugsink](https://github.com/bugsink/bugsink), a lightweight and open-source error monitoring tool, on a [Dokku](https://dokku.com/) host. Dokku is a lightweight PaaS that simplifies deploying and managing applications using Docker.

## Prerequisites

Before proceeding, ensure you have the following:

- A working [Dokku host](https://dokku.com/docs/getting-started/installation/).
- The [MariaDB plugin](https://github.com/dokku/dokku-mariadb) for database support.
- (Optional) The [Let's Encrypt plugin](https://github.com/dokku/dokku-letsencrypt) for SSL certificates.

## Setup Instructions

### 1. Create the App

Log into your Dokku host and create the `bugsink` app:

```bash
dokku apps:create bugsink
```

### 2. Configure the Database

Install, create, and link the PostgreSQL plugins to the app:

```bash
# Install plugins
dokku plugin:install https://github.com/dokku/dokku-mariadb.git mariadb

# Create instances
dokku mariadb:create bugsink

# Link plugins to the app
dokku mariadb:link bugsink bugsink
```

### 3. Configure Environment Variables

Set up the required environment variables for Bugsink:

```bash
# Set the base URL to your app's domain (use https if SSL is enabled)
dokku config:set bugsink BASE_URL=https://bugsink.example.com

# Set your preferred time zone (see: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
dokku config:set bugsink TIME_ZONE=Europe/Paris

# Generate and set a secure secret key
dokku config:set bugsink SECRET_KEY=$(openssl rand -hex 32)
```

### 4. Configure the Domain and Ports

Set the domain for your app to enable routing:

```bash
dokku domains:set bugsink bugsink.example.com
```

Map the internal port `8000` to the external port `80`:

```bash
dokku proxy:ports-set bugsink http:80:8000
```

### 5. Deploy the App

You can deploy the app to your Dokku server using one of the following methods:

#### Option 1: Deploy Using `dokku git:sync`

If your repository is hosted on a remote Git server with an HTTPS URL, you can deploy the app directly to your Dokku server using `dokku git:sync`. This method also triggers a build process automatically. Run the following command:

```bash
dokku git:sync --build bugsink https://github.com/d1ceward-on-dokku/bugsink_on_dokku.git
```

#### Option 2: Clone the Repository and Push Manually

If you prefer to work with the repository locally, you can clone it to your machine and push it to your Dokku server manually:

1. Clone the repository:

    ```bash
    # Via SSH
    git clone git@github.com:d1ceward-on-dokku/bugsink_on_dokku.git

    # Via HTTPS
    git clone https://github.com/d1ceward-on-dokku/bugsink_on_dokku.git
    ```

2. Add your Dokku server as a Git remote:

    ```bash
    git remote add dokku dokku@example.com:bugsink
    ```

3. Push the app to your Dokku server:

    ```bash
    git push dokku master
    ```

Choose the method that best suits your workflow.

### 6. Enable SSL (Optional)

Secure your app with an SSL certificate from Let's Encrypt:

1. Add the HTTPS port:

    ```bash
    dokku ports:add bugsink https:443:8000
    ```

2. Install the Let's Encrypt plugin:

    ```bash
    dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
    ```

3. Set the contact email for Let's Encrypt:

    ```bash
    dokku letsencrypt:set bugsink email you@example.com
    ```

4. Enable Let's Encrypt for the app:

    ```bash
    dokku letsencrypt:enable bugsink
    ```

## Wrapping Up

Congratulations! Your Bugsink instance is now up and running. You can access it at [https://bugsink.example.com](https://bugsink.example.com).

Happy tracking!
