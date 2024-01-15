# Debugging Laravel Sail with XDebug  

## Configure Laravel application

1. Update `.env` file to add these values

```
SAIL_XDEBUG_MODE=debug,develop
SAIL_XDEBUG_CONFIG="client_host=host.docker.internal idekey=docker"
```

2. Update `docker-compose.yml` file to add `PHP_IDE_CONFIG: 'serverName=Docker'` environment variable under `laravel.test`

![](https://github.com/dinhhuy258/notes/assets/17776979/774ca4a0-6117-49ab-8493-76455df64249) 

3. To ensure that our docker setup is configured correctly you can temporarily add a new route to your web.php files as the following

```php
Route::get('/phpinfo', function () {
    return phpinfo();
});
```

Go to `http://{your-domain}/phpinfo`

You should see similar values as of this:

![](https://github.com/dinhhuy258/notes/assets/17776979/6a7572da-28bf-46ad-817d-5e469ab02c53) 

**Great! Everything is set up**

## References

- [https://blog.stackademic.com/debugging-laravel-sail-with-xdebug-3-in-phpstorm-2023-a-detailed-guide-84a594c09586](https://blog.stackademic.com/debugging-laravel-sail-with-xdebug-3-in-phpstorm-2023-a-detailed-guide-84a594c09586)  

