# General Laravel Tips

### Quickly inspect return values

```php
public function myFunction()
{
    return tap(User::admins()->get(), 'logger');
}
```