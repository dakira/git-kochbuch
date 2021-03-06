# Magento 2 Installation

> Es werden sowohl Accounts auf [magento.com](https://account.magento.com/customer/account/login), wie auch [github.com](https://github.com) benötigt.

```
cd /var/www/vhosts/yumdap.net/mage2.yumdap.net/
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition mage2
cd mage2/
find var vendor pub/static pub/media app/etc -type f -exec chmod g+w {} \;
find var vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} \;
chmod u+x bin/magento
```

- In den Hosting-Einstellungen das *docroot* auf /mage2.yumdap.net/mage2 setzen
- http://mage2.yumdap.net/setup aufrufen und durchführen
- Nach Abschluss des Setups das *docroot* auf /mage2.yumdap.net/mage2/pub setzen.

```
# Zur Erinnerung noch mal das aktuelle Verzeichnis anzeigen
pwd
# cronjobs erstellen
crontab -e
```

Folgende crobtab Einträge machen:

```
MAILTO=""
* * * * * <path to php binary> <magento install dir>/bin/magento cron:run | grep -v "Ran jobs by schedule" >> <magento install dir>/var/log/magento.cron.log
* * * * * <path to php binary> <magento install dir>/update/cron.php >> <magento install dir>/var/log/update.cron.log
* * * * * <path to php binary> <magento install dir>/bin/magento setup:cron:run >> <magento install dir>/var/log/setup.cron.log
```

Testweise ins Backend einloggen, dann deutsche Anpassungen installieren.

```
composer config repositories.firegento_magesetup vcs git@github.com:firegento/firegento-magesetup2.git
composer require firegento/magesetup2:dev-develop
bin/magento module:enable FireGento_MageSetup
bin/magento setup:upgrade
bin/magento magesetup:setup:run de
composer require splendidinternet/mage2-locale-de-de
rm pub/static/frontend/Magento/luma/de_DE/js-translation.json
bin/magento setup:static-content:deploy de_DE
php -d memory_limit=-1 bin/magento setup:di:compile
```
Dem Backend-User kann nun Deutsch als Sprache zugewiesen werden.