# Magento 2 Konfiguration

## Ungenutzte Komponenten entfernen

1. Shop -> Konfiguration -> Erweitert -> Erweitert
    Hier alle Module deaktivieren, die nicht benötigt werden. Z.B: Fedex, GiftMessage, NewRelicReporting, Newsletter, Ups, Usps
2. Wishlist deaktivieren
    Shop -> Konfiguration -> Kunden -> Wishlist
3. Produktvergleich deaktivieren:

        <page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
            <body>
                <referenceBlock name="catalog.compare.sidebar" remove="true"/>
                <referenceBlock name="view.addto.compare" remove="true" />
                <referenceBlock name="view.addto.wishlist" remove="true" />
            </body>
        </page>


  in `/app/design/frontend/Vendor/theme/Magento_Theme/layout/default.xml` einfügen (siehe [Doku](http://devdocs.magento.com/guides/v2.0/frontend-dev-guide/layouts/xml-manage.html#layout_markup_remove_elements)).

## Child Theme anlegen