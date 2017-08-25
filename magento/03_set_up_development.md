# Magento Dev Umgebung

1. Nötige PHP-Pakete

    ```
    sudo apt install php-cli php-curl php-gd php-intl php-json php-mbstring php-mcrypt php-readline php-soap php-xml php-zip
    ```

2. Composer

    ```
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php composer-setup.php
    php -r "unlink('composer-setup.php');"
    sudo mv composer.phar /usr/local/bin/composer
    ```
    
3. Valet

    ```
    composer global require cpriego/valet-linux
    valet install
    mkdir ~/Code && cd ~/Code
    valet park
    ```
    
4. Clone und install Repo

    ```
    git clone git@example.com:vendor/project.git myproject
    cd myproject
    composer install
    ```

5. Installiere dev Magento und richte DB ein
    Hierzu muss bereits eine Datenbank vorhanden sein.
    
    ```
    magento setup:install --base-url=http://myproject.dev/ \
    --db-host=localhost --db-name=magento --db-user=magento --db-password='magento' \
    --admin-firstname=Magento --admin-lastname=User --admin-email=user@example.com \
    --admin-user=admin --admin-password='admin123' --language=de_DE \
    --backend-frontname=admin --session-save=files --cleanup-database
    --currency=EUR --timezone=Europe/Berlin --use-rewrites=1
    ```

6. Magento in Developer-Mode setzen

    `bin/magento deploy:mode:set developer`