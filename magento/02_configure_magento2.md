# Magento 2 Konfiguration

## Developer-Mode aktivieren

    bin/magento deploy:mode:set developer

## Child Theme anlegen

### 1. Theme-Ordner anlegen

```
mkdir -p app/design/frontend/Vendor/theme
cd app/design/frontend/Vendor/theme
mkdir -p web/{css,css/source,fonts,images,js}
```

### 2. theme.xml und registration.php

*app/design/frontend/Vendor/theme/theme.xml*
```
<theme xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Config/etc/theme.xsd">
     <title>My Theme Name</title>
     <parent>Magento/blank</parent>
</theme>
```
*app/design/frontend/Vendor/theme/registration.php*
```
<?php
/**
 * Copyright © 2015 Magento. All rights reserved.
 * See COPYING.txt for license details.
 */
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::THEME,
    'frontend/Vendor/theme',
    __DIR__
);
```

### 3. Less-Datei anlegen und Kompilierung aktivieren

Sämtliche CSS-Anpassungen sollten in der Datei *app/design/frontend/Vendor/theme/web/css/source/_theme.less* stattfinden. Damit diese kompiliert werden, müssen npm-Abhängigkeiten und grunt installiert werden.

```
cd <magento_root>
cp package.json{.sample,}
cp Gruntfile.js{.sample,}
npm install
# falls noch nicht global verfügbar, grunt installieren. U.U. muss dies als root geschehen
npm install -g grunt-cli
```

Das erstellte Child-Theme muss noch der Kompilierung bekannt gegeben werden. Dazu folgendes dem Config-Array hinzufügen:

> Mit **theme** ist im folgenden der Name des Themes gemeint. Der gleiche, den auch der Theme-Ordner hat.

*dev/tools/grunt/configs/themes.js*
```
theme: {
    area: 'frontend',
    name: 'Vendor/theme',
    locale: 'de_DE',
    files: [
        'css/styles-m',
        'css/styles-l'
    ],
    dsl: 'less'
}
```

Dann **einmalig** folgende Befehle ausführen:

```
grunt clean:<new_theme>
rm -r pub/static/* var/view_preprocessed/* var/cache/* var/di var/generation
bin/magento setup:di:compile
bin/magento setup:static-content:deploy de_DE
grunt exec:theme
grunt less:theme
```

Nun kann mit `grunt watch` das Theme-Less dauerhaft beobachtet werden, so dass man einfach immer weiter arbeiten kann.

## Ungenutzte Komponenten entfernen

1. Shops -> Konfiguration -> Erweitert -> Erweitert
    Hier alle Module deaktivieren, die nicht benötigt werden. Z.B: Fedex, GiftMessage, NewRelicReporting, Newsletter, Ups, Usps
2. Wishlist deaktivieren
    Shops -> Konfiguration -> Kunden -> Wishlist
3. Produktvergleich und Reviews deaktivieren:

    ```
    <page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
        <body>
            <referenceBlock name="catalog.compare.sidebar" remove="true"/>
            <referenceBlock name="view.addto.compare" remove="true" />
            <referenceBlock name="view.addto.wishlist" remove="true" />
            <referenceBlock name="reviews.tab" remove="true" />
            <referenceBlock name="product.review.form" remove="true" />
            <referenceBlock name="product.info.review" remove="true" />
        </body>
    </page>
    ```
    in */app/design/frontend/Vendor/theme/Magento_Theme/layout/default.xml* einfügen (siehe [Doku](http://devdocs.magento.com/guides/v2.0/frontend-dev-guide/layouts/xml-manage.html#layout_markup_remove_elements)).

4. Eigenen **footer_links** CMS-Block benutzen

    ```
    <referenceContainer name="footer-container">
        <container name="footer" as="footer" label="Page Footer" htmlTag="div" htmlClass="footer content">
            <block class="Magento\Store\Block\Switcher" name="store_switcher" as="store_switcher" template="switch/stores.phtml"/>
            <block class="Magento\Cms\Block\Block" name="footer_links">
                <arguments>
                    <argument name="block_id" xsi:type="string">footer_links</argument>
                </arguments>
            </block>
            <block class="Magento\Theme\Block\Html\Footer" name="copyright" template="html/copyright.phtml"/>
        </container>
    </referenceContainer>
    ```

    in den `<body>` der */app/design/frontend/Vendor/theme/Magento_Theme/layout/default.xml* einfügen. Unter Inhalt->Blöcke einen Block namens *footer_links* erstellen, der bspw. folgenden Inhalt haben könnte:
  
    ```
    <ul class="footer links">
        <li class="nav item"><a href="{{store url="agb"}}">AGB</a></li>
        <li class="nav item"><a href="{{store url="widerruf"}}">Widerruf</a></li>
        <li class="nav item"><a href="{{store url="lieferung"}}">Lieferung &amp; Versand</a></li>
        <li class="nav item"><a href="{{store url="zahlung"}}">Zahlungsarten</a></li>
        <li class="nav item"><a href="datenschutz">Datenschutz</a></li>
        <li class="nav item"><a href="bestellung">Bestellvorgang</a></li>
        <li class="nav item"><a href="{{store url="impressum"}}">Impressum</a></li>
    </ul>
    ```

## Problemlösungen

Wenn es Probleme gibt, dass die Themes nicht erkannt werden hilft es folgende Befehle auszufühen:

    rm -r pub/static/* var/view_preprocessed/* var/cache/* var/di var/generation
    bin/magento cache:clean
    bin/magento cache:flush
    bin/magento setup:upgrade
    bin/magento setup:di:compile
    bin/magento setup:static-content:deploy de_DE en_US