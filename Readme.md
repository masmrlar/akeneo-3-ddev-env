# Ddev env for Akeneo 3.x

A  lightweight/simple ddev env for Akeneo 3.x development.

## Setup

* install [ddev](https://ddev.readthedocs.io/en/stable/)
* startup the ddev env
* checkout our akeneo project into `pim-project` - the ddev config is set up to serve from `/var/www/html/pim-project/web`
  * run you composer or bin/console commands using `ddev exec` or better `ddev ssh` (don't forget `ddev auth ssh` prior to ddev ssh)

## Details on the Configuration

### Additional Elasticsearch Instance

Akneo 3.x uses Elasticsearch 3.8 as storage for product data - thus an instance is fired up.

#### Elasticsearch Access / Credentials (within ddev)

| Key     | Value           |
|---------|-----------------|
| `host`  | `elasticsearch` |
| `ports` | `9200,9300`     |

#### Elasticsearch Access from outside

| Key     | Value                            |
|---------|----------------------------------|
| `host`  | `elasticsearch.${DDEV_SITENAME}` |
| `ports` | `9200`                           |

### Additional Mysql Instance

A separate mysql >5.7.22 < 8 instance is used as ddev provides only mariadb which does not contain `JSON_ARRAYAGG` used by Akeneo 3.x.

As a separat db instance is used, the usual ddv db dump/restore features won't work, but the instance used a volume - configuration is not lost on ddev restarts.

#### MySQL Access / Credentials (within ddev)

| Key        | Value   |
|------------|---------|
| `host`     | `mysql` |
| `port`     | `3306`  |
| `user`     | `db`    |
| `password` | `db`    |

#### MySQL Access from outside

| Key        | Value                    |
|------------|--------------------------|
| `host`     | `mysql.${DDEV_SITENAME}` |
| `port`     | `3306`                   |
| `user`     | `db`                     |
| `password` | `db`                     |

## Example Setup - Plain Akeneo Install

Setting up a pim project to get a plain Akeneo 3.1 running in ddev (see also [https://docs.akeneo.com/master/install_pim/manual/index.html](https://docs.akeneo.com/master/install_pim/manual/index.html))

* fire up ddev env
  
  ```bash
  ddev start
  ddev auth ssh
  ```

* create the pim project
  
  ```bash
  ddev ssh
  composer create-project --prefer-dist akeneo/pim-community-standard . pim-project "3.1.*@stable"
  ```

  * On the interactive configuration use
  
    | Key                                    | Value                                                         |
    |----------------------------------------|---------------------------------------------------------------|
    | `database_driver`                      | `pdo_mysql`                                                   |
    | `database_host`                        | `mysql`                                                       |
    | `database_port`                        | `null` or `3306`                                              |
    | `database_name`                        | `db`                                                          |
    | `database_user`                        | `db`                                                          |
    | `database_password`                    | `db`                                                          |
    | `locale`                               | `en` or whatever locale you want to have                      |
    | `secret`                               | As this is just an example - whatever you like or the default |
    | `product_index_name`                   | default value                                                 |
    | `product_model_index_name`             | default value                                                 |
    | `product_and_product_model_index_name` | default value                                                 |
    | `index_host`                           | `elasticsearch: 9200`                                         |

* run the installer
  
  just in case you left the ssh shell above:
  
  ```bash
  ddev ssh
  ```

  now install it

  ```bash
  cd pim-project
  bin/console cache:clear --no-warmup --env=prod
  yarn install
  bin/console pim:installer:assets --symlink --clean --env=prod
  bin/console pim:install --force --symlink --clean --env=prod
  yarn run webpack
  ```

  exit ddev ssh by
  
  ```bash
  exit
  ```

* Now surf to [https://akeneo-3.1.ddev.site/](https://akeneo-3.1.ddev.site/) and login using `admin` / `admin`

To also see running jobs, dont forget to run 

```bash
ddev exec /var/www/html/pim-project/bin/console akeneo:batch:job-queue-consumer-daemon --env=prod
```
