# Nextcloud on Dokku
> This is a complete overhaul of the initial fork from @dionysio that has been tested, confirmed working and used in production for Nextcloud 18.

With recent versions of Nextcloud, it became super easy to run `nextcloud` under dokku:

```sh
dokku postgres:create nextcloud
dokku apps:create nextcloud
dokku postgres:link nextcloud nextcloud
```

Choose which version of Nextcloud you want to deploy from [Docker Hub](https://hub.docker.com/_/nextcloud).

```sh
dokku docker-options:add nextcloud build "--build-arg NEXTCLOUD_VERSION={{put your version here}}"
```

Then, from your machine, after cloning this repo and adding `dokku` as a remote:

```sh
git push dokku
```

Now back to your dokku server!

```sh
dokku domains:add nextcloud <your domain>
dokku letsencrypt nextcloud
# Replace this path with where you want to store user files (can be a network disk).
mkdir -p /var/lib/dokku/data/storage/nextcloud/data
chown www-data:www-data /var/lib/dokku/data/storage/nextcloud/data

# Replace this path with where you want to store Nextcloud config (can be a network disk, but this'll make serving webpages much slower)
mkdir -p /var/lib/dokku/data/storage/nextcloud/config
chown www-data:www-data /var/lib/dokku/data/storage/nextcloud/config

dokku storage:mount nextcloud /var/lib/dokku/data/storage/nextcloud/data:/var/www/html/data
dokku storage:mount nextcloud /var/lib/dokku/data/storage/nextcloud/config:/var/www/html/config
# required to create the mount files (we pushed earlier)
dokku ps:restart nextcloud

```
You're nearly done! Visit `<your domain>` to finish configuration.

Set up an admin account.
Click on "Storage & database".
Leave the data folder unchanged (`/var/www/html/data`)
For the database, **select Postgres**; you'll find the database credentials by running `dokku config nextcloud` and looking for DATABASE_URL: it'll look like postgres://USER:PASSWORD@HOST:PORT/DBNAME, just fill-in the fields on the webpage.

For instance, with `postgres://postgres:ae9e02101f9977e1fabb19f09605e486@dokku-postgres-nextcloud:5432/nextcloud`:

* Database user: postgres
* Database password: ae9e02101f9977e1fabb19f09605e486
* Database name: nextcloud
* Database host: dokku-postgres-nextcloud:5432

Click on finish setup.


## Tweaks
If you're planning to use desktop client,  because you're running with Dokku as a reverse proxy, you'll need to change your config.php file under `/var/lib/dokku/data/storage/nextcloud/config` to add this:

```
'overwriteprotocol' => 'https'
```

Make sure the value for `'overwrite.cli.url'` starts with https.

You'll also need to allow large uploads to your server:

```sh
mkdir /home/dokku/nextcloud/nginx.conf.d/
echo 'client_max_body_size 50000m;' > /home/dokku/nextcloud/nginx.conf.d/upload.conf
echo 'proxy_read_timeout 600s;' >> /home/dokku/nextcloud/nginx.conf.d/upload.conf
chown dokku:dokku /home/dokku/nextcloud/nginx.conf.d/upload.conf
service nginx reload
```

## Uninstalling
If you're unhappy with your setup, this'll remove everything *forever*:
```
dokku apps:destroy nextcloud
dokku postgres:destroy nextcloud
rm -rf  /var/lib/dokku/data/storage/nextcloud
```
