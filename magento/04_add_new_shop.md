# Neuen Shop hinzufügen

## Plesk Einstellungen

1. Eine neue Subdomain erstellen (z.B. 4promille.hamburgrecords.com) und als Dokumentenstamm `mage2.hamburgrecords.com/magento` hinterlegen. In den PHP-Einstellungen für die neue Subdomain die Version 7.0 festlegen.
2. In den *Apache und Nginx Einstellungen* für die Domain muss noch der Store-Code hinterlegt werden. Das muss man für http und https hinterlegen:
    ```
    SetEnv MAGE_RUN_CODE "vier_promille_de"
    SetEnv MAGE_RUN_TYPE "store"
    ```
3. Unter *Tools und Einstellungen -> Service-Verwaltung* die Dienste Nginx und Apache neu starten.
4. Nach ca. 10 Minuten Wartezeit dürfte die Domain erreichbar sein. Nun kann man in den Einstellungen der Subdomain ein **SSL-Zertifikat** bei Lets Encrypt beantragen.

## Im Terminal auf dem Server

> **WICHTIG**: Im folgenden wird immer vom Ordner der Hauptinstallation ausgegangen. Zum Testen und neu bauen sollte immer die Testinstallation unter `~/test.hamburgrecords.com/magento` genommen werden.

    ssh hhrec@hamburgrecords.com
    cd ~/mage2.hamburgrecords.com/magento/app/design/frontend/hhrec
    cp -ar feuerschwanz <new_theme> && sed -i 's/feuerschwanz/<new_theme>/g' <new_theme>/registration.php && sed -i 's/Feuerschwanz/New Theme/g' <new_theme>/theme.xml

Mit dem letzten Befehl wird der Theme-Ordner erstellt. Für 4Promille würde das z.B. so aussehen:

    cp -ar feuerschwanz vier_promille && sed -i 's/feuerschwanz/vier_promille/g' vier_promille/registration.php && sed -i 's/Feuerschwanz/4Promille/g' vier_promille/theme.xml
    
Nun muss das Theme aktiviert werden

    cd ~/mage2.hamburgrecords.com/magento
    micro dev/tools/grunt/configs/themes.js

Dort das Theme so hinterlegen, wie die anderen bereits hinterlegt sind, dann speichern (ctrl+s) und beenden (ctrl+q).

    grunt clean:<new_theme>
    grunt exec:<new_theme>
    grunt less:<new_theme>

Nun ist das neue Theme einsatzbereit und kann bspw. über die Datei `app/design/frontend/hhrec/vier_promille/web/css/source/_theme.less` bearbeitet werden. Damit die Updates live übernommen werden muss noch der *Watcher* gestartet werden. Dazu folgenden Befehl ausführen:

    grunt watch:<new_theme>

Für Details hierzu siehe auch [Magento Konfiguration](02_configure_magento2.md). Eigentlich muss für Änderungen nun nur noch der watch-Befehl ausgeführt werden. Wenn das mal nicht klappt (also Änderungen nicht erkannt werden) hilft es die drei Befehle von oben noch mal zu starten (clean, exec, less).
    

## Im Magento Backend

1. **Im Backend** unter *Produkte -> Kategorien* Hauptkategorie anlegen. Unter *Shops -> Alle Stores* eine Website, einen Shop und ein oder mehrere StoreViews anlegen. Dabei den Website-Code kleinschreiben (z.B. `vier_promille`), beim Shop die Hauptkategorie zuweisen und die StoreView nach der jeweiligen Sprache benennen (z.B. *Deutsch* oder *English*) und einen entspr. Code wählen (z.B. `vier_promille_de`).
2. Auf *Shops -> Konfiguration -> Allgemein -> Web* gehen und dann als StoreView die Website auswählen (in unserem Fall also *4Promille*). Dort Basis- und Secure-URLs so anpassen, dass es zur erstellten Subdomain passt. Hier können unter *Allgemein* noch die Shopinformationen und das Impressum für den aktuellen Store angepasst werden. Unter *Store-Email-Adressen* kann man noch die entspr. Kontakte hinterlegen.
3. Unter *Inhalt -> Design -> Konfiguration* die Website wählen und rechts auf bearbeiten drücken. Dort das Theme auswählen und alle weiteren Einstellungen machen (Logo, Header, Footer, usw.).