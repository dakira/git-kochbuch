# Magento 2 Installation

> Auf dem Server läuft per default PHP 5.5. Es wird aber PHP 7.0 benötigt. Alle Befehle müssen daher mit *php7* (statt *php*) bzw. composer7 ausgeführt werden!

> Es werden sowohl Accounts auf [magento.com](https://account.magento.com/customer/account/login), wie auch [github.com](https://github.com) benötigt.

```
cd /var/www/vhosts/yumdap.net/mage2.yumdap.net/
composer7 create-project --repository-url=https://repo.magento.com/ magento/project-community-edition mage2
cd mage2/
find var vendor pub/static pub/media app/etc -type f -exec chmod g+w {} \;
find var vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} \;
chmod u+x bin/magento
```

- In Plesk in den Hosting-Einstellungen das "root" auf /mage2.yumdap.net/mage2/pub setzen
- http://mage2.yumdap.net/setup aufrufen und durchführen

```
# Zur Erinnerung noch mal das aktuelle Verzeichnis anzeigen
pwd
# cronjobs erstellen
crontab -e
```

Folgende crobtab Einträge machen:

```
MAILTO=""
* * * * * /opt/plesk/php/7.0/bin/php /var/www/vhosts/yumdap.net/mage2.yumdap.net/mage2/bin/magento cron:run | grep -v "Ran jobs by schedule" >> /var/www/vhosts/yumdap.net/mage2.yumdap.net/mage2/var/log/magento.cron.log
# * * * * * /opt/plesk/php/7.0/bin/php /var/www/vhosts/yumdap.net/mage2.yumdap.net/mage2/update/cron.php >> /var/www/vhosts/yumdap.net/mage2.yumdap.net/mage2/var/log/update.cron.log
# * * * * * /opt/plesk/php/7.0/bin/php /var/www/vhosts/yumdap.net/mage2.yumdap.net/mage2/bin/magento setup:cron:run >> /var/www/vhosts/yumdap.net/mage2.yumdap.net/mage2/var/log/setup.cron.log
```

Deutsche Anpassungen installieren

```
composer7 config repositories.firegento_magesetup vcs git@github.com:firegento/firegento-magesetup2.git
composer7 require firegento/magesetup2:dev-develop
composer7 require splendidinternet/mage2-locale-de-de
rm pub/static/frontend/Magento/luma/de_DE/js-translation.json
php7 bin/magento setup:static-content:deploy de_DE
php7 bin/magento module:enable FireGento_MageSetup
php7 bin/magento setup:upgrade
php7 bin/magento magesetup:setup:run de
php7 -d memory_limit=-1 bin/magento setup:di:compile
```