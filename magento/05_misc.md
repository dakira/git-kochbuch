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