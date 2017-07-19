# Magento 2 Konfiguration

## Ungenutzte Komponenten entfernen

1. Shop -> Konfiguration -> Erweitert -> Erweitert
    Hier alle Module deaktivieren, die nicht benötigt werden. Z.B: Fedex, GiftMessage, NewRelicReporting, Newsletter, Ups, Usps
2. Wishlist deaktivieren
    Shop -> Konfiguration -> Kunden -> Wishlist
3. Produktvergleich deaktivieren:

        <referenceBlock name="catalog.compare.sidebar" remove="true"/>
        <referenceBlock name="view.addto.compare" remove="true" />
        <referenceBlock name="view.addto.wishlist" remove="true" />

  in `/app/design/frontend/Vendor/theme/Magento_Theme/layout/default.xml` einfügen.

## Child Theme anlegen