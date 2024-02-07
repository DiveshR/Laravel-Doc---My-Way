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
```
 php artisan make:model Flight --policy
 ```

### Generate a model and a migration, factory, seeder, and controller...
```
php artisan make:model Flight -mfsc
```

### Shortcut to generate a model, migration, factory, seeder, policy, controller, and form requests...
```
php artisan make:model Flight --all
```



### Generate a pivot model...
```
php artisan make:model Member --pivot
php artisan make:model Member -p
```

### Inspecting Models
Sometimes it can be difficult to determine all of a model's available attributes and relationships just by skimming its code. Instead, try the ```model:show``` Artisan command, which provides a convenient overview of all the model's attributes and relations:

```
php artisan model:show Flight
```

### Eloquent Model Conventions
Table Names : By convention, the "snake case", plural name of the class will be used as the table name unless another name is explicitly specified.
```
/**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'my_flights';
```

#### Primary Keys
```
    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'flight_id';
```

If you wish to use a non-incrementing or a non-numeric primary key you must define a public $incrementing property on your model that is set to false:
```
    /**
     * Indicates if the model's ID is auto-incrementing.
     *
     * @var bool
     */
    public $incrementing = false;
}
```

If your model's primary key is not an integer, you should define a protected $keyType property on your model. This property should have a value of string:
```
    /**
     * The data type of the auto-incrementing ID.
     *
     * @var string
     */
    protected $keyType = 'string';
```

### "Composite" Primary Keys

"Composite" primary keys are not supported by Eloquent models. However, you are free to add additional multi-column, unique indexes to your database tables in addition to the table's uniquely identifying primary key.

### UUID and ULID Keys

UUIDs are universally unique alpha-numeric identifiers that are ```36 characters``` long.

If you would like a model to use a UUID key instead of an auto-incrementing integer key, you may use the ```Illuminate\Database\Eloquent\Concerns\HasUuids``` trait on the model. 

```
use Illuminate\Database\Eloquent\Concerns\HasUuids;

class Article extends Model
{
    use HasUuids;
 
    // ...
}

$article = Article::create(['title' => 'Traveling to Europe']);
 
$article->id; // "8f8e8478-9035-4d23-b9a7-62f4d2612ce5"
```

By default, The HasUuids trait will generate "ordered" UUIDs for your models.

### Timestamps
By default, Eloquent expects ```created_at``` and ```updated_at``` columns to exist on your model's corresponding database table.

If you do not want these columns to be automatically managed by Eloquent,
```
class Flight extends Model
{
    /**
     * Indicates if the model should be timestamped.
     *
     * @var bool
     */
    public $timestamps = false;
}
```

If you need to customize the format of your model's timestamps, set the ```$dateFormat``` property on your model. 
```
    /**
     * The storage format of the model's date columns.
     *
     * @var string
     */
    protected $dateFormat = 'U';
```

If you need to customize the names of the columns used to store the timestamps, you may define ```CREATED_AT``` and ```UPDATED_AT``` constants on your model:
```
class Flight extends Model
{
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'updated_date';
}
```

If you would like to perform model operations without the model having its ```updated_at``` timestamp modified, you may operate on the model within a closure given to the withoutTimestamps method:
```
Model::withoutTimestamps(fn () => $post->increment(['reads']));
```


### Database Connections
```
    /**
     * The database connection that should be used by the model.
     *
     * @var string
     */
    protected $connection = 'sqlite';
```
### Default Attribute Values

By default, a newly instantiated model instance will not contain any attribute values. If you would like to define the default values for some of your model's attributes, you may define an $attributes property on your model. 
```
    /**
     * The model's default values for attributes.
     *
     * @var array
     */
    protected $attributes = [
        'options' => '[]',
        'delayed' => false,
    ];
```

### Retrieving Models
1. ```Flight::all()```

2. Building Queries: 
```
$flights = Flight::where('active', 1)
               ->orderBy('name')
               ->take(10)
               ->get();
```

### Refreshing Models

The fresh method will re-retrieve the model from the database. 
```
$flight = Flight::where('number', 'FR 900')->first(); 
$freshFlight = $flight->fresh();
```

The refresh method will re-hydrate the existing model using fresh data from the database. In addition, all of its loaded relationships will be refreshed as well:

```
$flight = Flight::where('number', 'FR 900')->first();
 
$flight->number = 'FR 456';
 
$flight->refresh();
 
$flight->number; // "FR 900"
```


### Collections
As we have seen, Eloquent methods like ```all()``` and ```get()``` retrieve multiple records from the database. However, these methods don't return a plain PHP array. Instead, an instance of ```Illuminate\Database\Eloquent\Collection``` is returned.

```
$flights = Flight::where('destination', 'Paris')->get();
 
$flights = $flights->reject(function (Flight $flight) {
    return $flight->cancelled;

});
```

### Chunking Results

Your application may run out of memory if you attempt to "load tens of thousands" of Eloquent records via the all or get methods.

The chunk method will retrieve a subset of Eloquent models, passing them to a closure for processing. Since only the current chunk of Eloquent models is retrieved at a time, the chunk method will provide significantly reduced memory usage when working with a large number of models:
```
use App\Models\Flight;
use Illuminate\Database\Eloquent\Collection;
 
Flight::chunk(200, function (Collection $flights) {
    foreach ($flights as $flight) {
        // ...
    }
});
```

### Advanced Subqueries
#### Subquery Selects

Eloquent also offers advanced subquery support, which allows you to pull information from related tables in a single query. 

For example, 
let's imagine that we have a table of flight ```destinations``` and a table of ```flights``` to destinations. The flights table contains an ```arrived_at``` column which indicates when the flight arrived at the destination.

Using the subquery functionality available to the query builder's ```select()``` and ```addSelect()``` methods, we can select all of the destinations and the name of the flight that most recently arrived at that destination using a single query:

```
use App\Models\Destination;
use App\Models\Flight;
 
return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderByDesc('arrived_at')
    ->limit(1)
])->get();
```
#### Subquery Ordering

In addition, the query builder's ```orderBy``` function supports subqueries. Continuing to use our flight example, we may use this functionality to sort all destinations based on when the last flight arrived at that destination. Again, this may be done while executing a single database query:

```
return Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderByDesc('arrived_at')
        ->limit(1)
)->get();
```

### Retrieving Single Models / Aggregates
You can retrieve single records using the ```find()```, ```first()```, or ```firstWhere()``` methods.

 Instead of returning a collection of models, these methods return a single model instance:

```
use App\Models\Flight;

// Retrieve a model by its primary key...
$flight = Flight::find(1);

// Retrieve the first model matching the query constraints...
$flight = Flight::where('active', 1)->first();

// Alternative to retrieving the first model matching the query constraints...
$flight = Flight::firstWhere('active', 1);
```

Sometimes you may wish to perform some other action if no results are found. The ```findOr()``` and ```firstOr()``` methods will return a single model instance or, if no results are found, execute the given closure. 

```
$flight = Flight::findOr(1, function () {
    // ...
});

```
```
$flight = Flight::where('legs', '>', 3)->firstOr(function () {
    // ...
});
```
#### Not Found Exceptions
Sometimes you may wish to throw an exception if a model is not found. This is particularly useful in routes or controllers. The ```findOrFail()``` and ```firstOrFail()``` methods will retrieve the first result of the query; however, if no result is found, an ```Illuminate\Database\Eloquent\ModelNotFoundException``` will be thrown:

```
$flight = Flight::findOrFail(1);
$flight = Flight::where('legs', '>', 3)->firstOrFail();
```

If the ModelNotFoundException is not caught, a 404 HTTP response is automatically sent back to the client:
```
use App\Models\Flight;
 
Route::get('/api/flights/{id}', function (string $id) {
    return Flight::findOrFail($id);
});
```
#### Retrieving or Creating Models
The ```firstOrCreate()``` method will attempt to locate a database record using the given column / value pairs. If the model can not be found in the database, a record will be inserted with the attributes resulting from merging the first array argument with the optional second array argument:

```
use App\Models\Flight;
 
// Retrieve flight by name or create it if it doesn't exist...
$flight = Flight::firstOrCreate([
    'name' => 'London to Paris'
]);
```
```
// Retrieve flight by name or create it with the name, delayed, and arrival_time attributes...
$flight = Flight::firstOrCreate(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```
```
// Retrieve flight by name or instantiate a new Flight instance...
$flight = Flight::firstOrNew([
    'name' => 'London to Paris'
]);
```
```
// Retrieve flight by name or instantiate with the name, delayed, and arrival_time attributes...
$flight = Flight::firstOrNew(
    ['name' => 'Tokyo to Sydney'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```

#### More Coming Soon..
