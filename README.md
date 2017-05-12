## Shopware Mage Tasks

This package includes common tasks to deploy a shopware project using [MagePHP](http://magephp.com/).

## .mage.yml example

```yaml
magephp:
    admins: { 'el-bardan', 'emtii' }
    php_executable: /usr/bin/php # Leave this empty if you want to use the globally installed php executable.
    custom_tasks:
        - BestIt\Mage\Tasks\Deploy\ReplacePlaceHoldersTask
        - BestIt\Mage\Tasks\Deploy\SyncConfigTask
        - BestIt\Mage\Tasks\Deploy\SyncFileTask
        - BestIt\Mage\Tasks\Deploy\SyncLegacyCommunityPluginsTask
        - BestIt\Mage\Tasks\Deploy\SyncLegacyLocalPluginsTask
        - BestIt\Mage\Tasks\Deploy\SyncLegacyPluginsTask
        - BestIt\Mage\Tasks\Deploy\SyncLibrariesTask
        - BestIt\Mage\Tasks\Deploy\SyncMigrationsTask
        - BestIt\Mage\Tasks\Deploy\SyncPluginsTask
        - BestIt\Mage\Tasks\Deploy\SyncScriptsTask
        - BestIt\Mage\Tasks\Deploy\SyncThemesTask
        - BestIt\Mage\Tasks\Misc\CopyTask
        - BestIt\Mage\Tasks\Quality\LintTask
        - BestIt\Mage\Tasks\Quality\VendorBinTask
        - BestIt\Mage\Tasks\Release\PrepareTask
        - BestIt\Mage\Tasks\Shopware\ApplyMigrationsTask
        - BestIt\Mage\Tasks\Shopware\ClearCacheTask
        - BestIt\Mage\Tasks\Shopware\CommandTask
        - BestIt\Mage\Tasks\Shopware\CreateAdminUsersTask
        - BestIt\Mage\Tasks\Shopware\UpdateLegacyPluginsTask
        - BestIt\Mage\Tasks\Shopware\UpdatePluginsTask
    environments:
        prod:
            user: apache
            host_path: /var/www/html
            rsync: -rvz --delete --no-o
            releases: 4
            hosts:
                - production_server1
            pre-deploy:
                - prepare/deploy # Replaces placeholder values in the config file.
                - vendor/bin: { cmd: 'phpcs', dir: '../../custom', flags: '--standard=ruleset.xml' } # execute, for example, phpcs with given flags.
                - quality/lint: { cmd: 'stylelint', dir: '../../themes', flags: '--syntax less' } # execute stylelint with less syntax.
            on-deploy:
                - deploy/release/prepare # Creates new release directory and copies all content of current into the created directory.
                - fs/link: { from: '../../media', to: 'media' } # Creates a new symlink.
                - fs/link: { from: '../../files', to: 'files' } # Creates a new symlink.
                - deploy/config # Pushes config file to server(s).
                - deploy/scripts # Pushes scripts files to server(s).
                - deploy/libraries # Pushes library (engine/Library) to server(s).
                - deploy/legacy-plugins # Pushes legacy plugins to server(s). Use this for 5.2 systems. Basically all legacy plugins will be pushed to the local namespace.
                - deploy/legacy-community-plugins # Pushes legacy plugins in the community namespace to server(s). Use this for <5.2 systems.
                - deploy/legacy-local-plugins # Pushes legacy plugins in the local namespace to server(s). Use this for <5.2 systems.
                - deploy/plugins # Pushes plugins (>=5.2 system) to server(s).
                - deploy/migrations: { src: 'sql' } # Pushes SQL migrations to server(s). src is optional "sql" is default.
                - deploy/themes # Pushes themes to server(s).
                - deploy/file: { src: 'engine/Shopware/Core/sBasket.php', target: 'engine/Shopware/Core/', rsync_flag: 'rvz' } # Pushes one file to a directory. rsync_flags is optional, default is 'rvz'.
                - shopware/create-admins # Creates admin users from admins parameter in .yml file.
                - shopware/update-plugins # Installs/Updates all (>=5.2 system) plugins on server(s).
                - shopware/update-legacy-plugins: { sync_sources_folders: true } # Installs/Updates all (legacy) plugins on server(s). "Sources" are the Community/Local folders.
                - shopware/migrate: { table_suffix: 'bestit', migration_dir: 'sql' } # Executes all SQL migrations on server(s). Both parameters are optional.
                - shopware/clear-cache # Clears Shopware cache on server(s).
                - shopware/command: { cmd: 'sw:theme:cache:generate' } # Warms up the shopware theme cache on server(s).
```

## Installation

### Step 1: Composer

Currently the package is not registered on Packagist, so you will have to tell composer about it manually.
You can do so by putting the following content in your composer.json file:

```json
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/bestit/ShopwareMageTasks"
        }
    ]
```

Then from the command line, run:

```
composer require bestit/shopware-mage-tasks
```

### Step 2: .mage.yml

Create a .mage.yml file in your project root directory and define your desired tasks as per the example above.
For more information about what you can do check out the [documentation](http://magephp.com/).

### Step 3: That's it!

You just need to run the deployment script:

```
vendor/bin/mage deploy <environment>
```

## License

This software is open-sourced under the MIT license.