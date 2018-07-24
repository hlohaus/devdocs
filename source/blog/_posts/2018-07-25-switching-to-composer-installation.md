---
title: An easy example how to switch from classic installation to composer installation
tags:
- Deployment
- Composer Installation

categories: 
- dev

authors: [tn]
github_link: /blog/_posts/2018-07-25-switching-to-composer-installation.md

---

In this blog post you will learn how to change your Shopware classic installation to the composer installation. I already mentioned it in the shopware [Livestream](https://www.youtube.com/watch?v=oUME-FnlUKE) (a new format you should know ;-) ).

The requirements are very easy to fulfill. We need Shopware in the latest Version of 5.4. Nothing more.

### clean up classic installation
So let`s start. Please go into the shopware root directory and delete some files and directories which are not longer needed from the classic installation: 
```bash 
rm -rv bin recovery vendor composer.* shopware.php
```

### create composer installation

Now we create a new composer project with the following command: 

```bash
composer create-project shopware/composer-project composer-installation \ 
    --no-interaction --stability=dev
```
You should receive an error message like the following because we didn't created a .env file: 

```bash
Could not load .env file
Script ./app/post-update.sh handling the post-update-cmd event returned with error code 1
```

### merge classic and composer installation

The command creates a new directory composer-installation witch contains all needed files for our composer installation. After that we have to move some files and directories stored in the new directory a level up with this commands:
```bash
mv composer-installation/app ./
mv composer-installation/bin ./
mv composer-installation/Plugins ./
mv composer-installation/vendor ./
mv composer-installation/composer.json ./
mv composer-installation/shopware.php ./
mv composer-installation/.env.example ./.env
```

The directories custom, files, media, themes, var and web are equal with the composer installation so we can still use them and don't lose our files, plugins and themes. If you have Plugins in engine/Shopware/Plugins/Community or engine/Shopware/Plugins/Local they must be moved to ```./Plugins```!

After that we should reduce the .env file. Please use your database credentials from your config.php file and delete it afterwards. It should look like the following: 

```
# Database credentials
DB_HOST=localhost
DB_DATABASE=shopware
DB_USERNAME=root
DB_PASSWORD=root
DB_PORT=3306

# This variable is checked by Shopware
DATABASE_URL=mysql://${DB_USERNAME}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_DATABASE}

# Environment
SHOPWARE_ENV="production"
```

The next step is to set your database credentials as an environment variable. An example virtual host configuration can be found bellow:

```
<VirtualHost *:80>
        ...
        SetEnv DB_USERNAME "root"
        SetEnv DB_PASSWORD "root"
        SetEnv DB_DATABASE "shopware"
        SetEnv DB_HOST "localhost"
        SetEnv DB_PORT "3306"
        SetEnv SHOPWARE_ENV "dev"
</VirtualHost>
```

The last step is to deactivate the SwagUpdate Plugin via the following command:

```bash
./bin/console sw:plugin:uninstall SwagUpdate
```

## Conclusion
The switch from classic installation to the new composer installation is not very difficult. Please use it if you want to achieve easier deployment and shopware updates. Our colleague Soner aka shyim develops an extension for the composer project to handle community store plugins as a composer dependency. If you are interested please have a look [here](https://github.com/shyim/store-plugin-installer).