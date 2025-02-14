# Docker Compose

This is the documentation for how to use Docker Compose to set up Cobrowse Enterprise on a single host.

### Dependencies

You'll need these installed on the host system before continuing:

* ****[**Docker**](https://www.docker.com)****
* ****[**Docker Compose**](https://docs.docker.com/compose/install/)****
* ****[**NodeJS**](https://nodejs.org/en/) – at least [minimum LTS version](https://nodejs.org/en/about/releases/)

### System Requirements

A single VM is all that is required to run Cobrowse. We recommend at least 2 vCPUs, 4GB memory, and 50GB SSD storage. This setup is a fast way to get started, and works well for many customers. For more information about system requirements, please see [Sizing guidelines](sizing-guidelines.md).

Cobrowse.io is deployed as containers, and we support many additional configurations to provide highly-available and large scale deployments. Please reach out below to discuss your specific requirements!

## Initial Setup

Before you're ready to run Cobrowse you'll need to configure the host system with our managed Docker Compose file. The Docker Compose files defines the services and internal configuration that is required to easily set up the different Cobrowse server components.

*   Setup access to our private Docker registry. This is where your Docker host can obtain the Cobrowse Enterprise images. You will need the Cobrowse Enterprise repo password that has been assigned to you.

```bash
docker login ghcr.io --username cobrowse-enterprise
```
* Enter your Cobrowse Enterprise repo password when prompted for a password.

### Generate the Config Directory

We have provided a small command line utility to help you get started. This utility will gather the required config for your deployment. Run the following command from your terminal:

```bash
npx cobrowse-enterprise create compose ./example
```

You can replace "./example" with the directory where you wish to save the configuration data. The directory will be created if it does not exist yet.

### Customize the Configuration

The installer above will generate a `.env` file that contains your configuration. You may edit this file to add further configuration. We support the following environment variables:

|     ENV VAR     |                   Description                   |     Example     | Required |
| :-------------: | :---------------------------------------------: | :-------------: | :------: |
|      DOMAIN     |         Domain name to run Cobrowse on.         |   mydomain.com  |    Yes   |
|     LICENSE     |   The license string you have been given by us  |                 |    Yes   |
| SSL\_GENERATION |    "automatic" (uses LetsEncrypt) or "manual"   |    automatic    |    No    |
| SSL\_VALIDATION | 0 = disable cert validation (self signed certs) |        0        |    No    |
|    SUPERUSERS   |   RegEx to specify super user email addresses   | .\*@example.com |    No    |

#### Configuring DNS

In order to be able to access Cobrowse, you'll need to set up DNS records to point to your docker host. The process will depend on which domain provider you are using. See their relevant documentation for how to set this up.

#### Configuring SSL

Our Docker images will automatically create an SSL certificate for you using [Let's Encrypt](https://letsencrypt.org/). In order for this process to work, your Docker host must be exposed to the internet and available on port 80 at the `DOMAIN` configured above. This is required in order for the domain validation to succeed before an SSL certificate is issued.

The certificates and configuration generated by Let's Encrypt will be available at `./data/certs` relative to the path of the docker-compose.yml file.

**SSL Configuration when your domain is not publicly resolvable**

If your domain is not publicly resolvable, set `SSL_GENERATION=manual` in your environment configuration. You will then have to place the following files:

* `./data/certs/fullchain.pem` – This is the certificate chain you wish to use
* `./data/certs/privkey.pem` – This is the private key for your certificate

The files are relative to the placement of the `docker-compose.yml` file.

### Starting Cobrowse

You're now ready to try starting Cobrowse. Just run these commands in the same location as the docker-compose.yml file:

```bash
docker-compose up
```

When all the docker containers have started, Cobrowse should be available on the domain you configured above. _Tip: add `--detach` to run the services in the background._

Open up a web browser and navigate to your Cobrowse domain, you should then be able to use the email registration flow to create a new account and test out your deployment!

{% content-ref url="getting-started/" %}
[getting-started](getting-started/)
{% endcontent-ref %}



### Shutting down Cobrowse

Open up a terminal to the location of the docker-compose.yml file and run:

```bash
docker-compose down
```

## Other considerations

### Managing the database

The Cobrowse docker-compose file will start a MongoDB instance for you. The database storage will be exposed on a docker volume at `./data/db` relative to the path of the docker-compose.yml file.

**Backing up MongoDB**

You are responsible for backing up the database regularly. Cobrowse will not do this automatically in any way. See the [MonogDB docs](https://docs.mongodb.com/manual/core/backups/) for recommendations on backup strategies.

### Upgrading Cobrowse

The config directory created by our command line utility is a git repo. You can update to the latest version by doing:

```bash
docker-compose down
git pull upstream stable
docker-compose up
```

_Note: We recommend making a database backup before upgrading to new versions_

### Host firewalls

If you're running additional firewalls to those that Docker sets up automatically, you need to ensure that port 80 (HTTP) and port 443 (HTTPS) are both exposed for the docker host.

{% hint style="success" %}
Any questions at all? Please email us at [hello@cobrowse.io](mailto:hello@cobrowse.io).
{% endhint %}
