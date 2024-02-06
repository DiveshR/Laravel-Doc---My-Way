# Eloquent: Getting Started


An object-relational mapper (ORM) that makes it enjoyable to interact with your database.
Before getting started, be sure to configure a database connection in your application's 

config/database.php

### Generating Model Classes:
```
php artisan make:model Flight -f
php artisan make:model Flight -s
php artisan make:model Flight -c
php artisan make:model Flight --controller --resource --requests
php artisan make:model Flight -crR
```
### Generate a model and a FlightPolicy class...
``` php artisan make:model Flight --policy```

### Generate a model and a migration, factory, seeder, and controller...
```php artisan make:model Flight -mfsc```

### Shortcut to generate a model, migration, factory, seeder, policy, controller, and form requests...
```php artisan make:model Flight --all```
