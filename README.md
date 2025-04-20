# phpBB Docker files - for Archipelago unofficial forum 
Docker Compose and build files used to host the short-lived Archipelago unofficial forum board,
which was hosted at `https://apforum.flitpix.net` before being shut down on April 26, 2025.

This does not include any assets not owned by me, including anything derived from the Archipelago asset pack.

## Prerequisites

The host system must have the following software installed to proceed:

- [Git](https://git-scm.com/)
- [Docker Engine](https://docs.docker.com/engine/install/)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Setup process

All instructions here assume that...

- You are on a Linux server, though this should work fine on a Windows server with Docker as well.
- You are familiar with [Docker Engine](https://docs.docker.com/engine/) and [Docker Compose](https://docs.docker.com/compose/).
- You are familiar with the basics of hosting a website using [Apache httpd](https://httpd.apache.org/).

### Preparation

Clone this repository.

Open `db_password.txt` (phpBB database secret), `db_root_password.txt` (root database secret), and `build/forum/phpbb-install.yml` (phpBB admin account username/password; and board, database, and email settings), and set their respective options.

Open `docker-compose.yml`, and follow the commented instructions.

### Installation

Ensure you're in the same directory as `docker-compose.yml`, then run the Docker Compose script to start the containers:

```
docker-compose up -d
```

Building the forum container may take some time.

Install phpBB using the phpBB CLI (any `phpBB Debug` errors can be safely ignored):

```
docker exec -it --user www-data apforum php /var/www/html/board/install/phpbbcli.php install /var/www/html/board/phpbb-install.yml
docker exec -it apforum rm -r /var/www/html/board/install/ /var/www/html/board/docs/ phpbb-install.yml
```

The first command installs phpBB.
The second command deletes the `install` and `docs` directories, as well as `phpbb-install.yml`.
`install` must be deleted to complete setup.
It is highly recommended to delete `phpbb-install.yml` as it may contain your admin user password if set.
If you aren't sure about any of this, just run the above commands.

You can now go to `https://domain.tld/board` (or `http://ipaddress/board`) to access phpBB!
Sign into the admin account, go into the ACP, and set up your extensions, styles, users, groups, and forums.
To learn how, see phpBB's
[User Guide](https://www.phpbb.com/support/docs/en/3.3/ug/adminguide/) and
[Knowledge Base](https://www.phpbb.com/support/docs/en/3.3/kb/).

This can also be used locally to develop and test extensions and styles for phpBB. I highly encourage this!

## Installing additional extensions and/or styles

After installing your board, you may later find additional extensions and/or styles you might want to install.

To download them, use `curl -fSL <url> -o <filename>.zip`,
where `<url>` is the URL copied from the green download button on the phpbb.com extension page
([here's an example](https://www.phpbb.com/customise/db/extension/webpushnotifications/)).

Then, `unzip` it, and
[copy](https://docs.docker.com/reference/cli/docker/container/cp/) the resulting directory into the Docker container.
Extensions go into `/var/www/html/board/ext/`, while styles go into `/var/www/html/board/styles/`.

If you have issues afterwards, you may then need to go into the Docker container with `docker exec -it apforum bash`,
and use `chown -R www-data:www-data <directory>` on the `ext` and/or `styles` directories.
Past that, it might be a problem with the extension and you'll need to do independent troubleshooting from there.

## Other considerations/notes

### Search backend

Consider using [Manticore](https://manticoresearch.com/) as your search backend instead of native or database fulltext,
as it scales better on larger boards.
It's a drop-in replacement for Sphinx; the former is open source and still maintained,
while the latter is closed-source and no longer maintained,
so the choice is fairly obvious imo.

Untested, yet promising setup instructions can be found [here](https://www.phpbb.com/community/viewtopic.php?t=2607821).
Adjust them for your setup.

### Reverse proxy

I use [Traefik](https://traefik.io/traefik/) as my reverse proxy as it is very simple to set up and use,
and this is reflected in `docker-compose.yml`.

This should also work with other reverse proxies, like nginx.

If you continue with using a reverse proxy, and you configure it to use SSL (you should),
the Browser Push Notifications extension will complain about the server not being configured for SSL.
You can safely ignore this, it will allow you to continue and will work.
