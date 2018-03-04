# Laravel Testing

## Quicker Tests

Add `Hash::setRounds(4)` to the *createApplication* method of your *tests/CreatesApplication.php*.

## Progress time in tests

```php
public function progressTime(int $minutes)
{
    $newNow = now()->copy()->addMinutes($minutes);
    
    Carbon::setTestNow($newNow);
    
    return $this;
}
```