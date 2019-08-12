# Misc

## Fix long login times:

In

lib/internal/Magento/Framework/Session/SessionManager.php

replace the regenerateId() function with this:

    public function regenerateId()
    {
        if (headers_sent()) {
            return $this;
        }
        if ($this->isSessionExists()) {
            session_regenerate_id(true);
        } else {
            session_start();
        }
        $this->storage->init(isset($_SESSION) ? $_SESSION : []);
        if ($this->sessionConfig->getUseCookies()) {
            $this->clearSubDomainSessionCookie();
        }
        return $this;
    }

See [this issue](https://github.com/magento/magento2/commit/aaa60b1b72bdc189b38492bd50b0ffb23101173e?diff=split) for reference.

## Fix state defaulting to Niedersachsen

1. In General Settings > State settings of the store, make the state mandatory for Germany
2. flush the config cache
3. Go to to Sales > Tax settings
    Now (and only now), you see a default state and default zip code. Switch default state to asterisk (* / any) and empty default zip code
4. Go back to General Settings > State settings, make state no longer mandatory (revert to what you had before)
5. Flush the config cache once more.
    Now, orders should no longer contain a state. It's not obvious (and again: I'm not sure about the culprit), but I hope this helps...

## Upgrade Magento

https://devdocs.magento.com/guides/v2.3/comp-mgr/cli/cli-upgrade.html

```
bin/magento maintenance:enable
composer require magento/product-community-edition 2.3.2 --no-update;
composer update;
rm -rf pub/static/* var/view_preprocessed/* var/cache/* var/di var/generation
redis-cli -n <db_number> FLUSHDB # as defined in env.php
bin/magento cache:clean
bin/magento cache:flush
bin/magento setup:upgrade
bin/magento setup:di:compile
bin/magento indexer:reindex
bin/magento setup:static-content:deploy de_DE en_US -f
bin/magento maintenance:disable
```