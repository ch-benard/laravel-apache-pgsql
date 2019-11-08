# LARAVEL CHEATSHEET

## USING MIGRATION TO CREATE AND ALTER TABLES

You can use Laravel migration tool to create a table :

```bash
docker-compose exec -u devuser php php artisan make:migration create_sometables_table
```

The command *php artisan make:migration* creates a script into the *database/migrations* folder. Executing this php script results in the creation of a table named *sometables* into the database defined in the *.env* file.

This command uses a convention : by using *create_sometables_table*, you tell Laravel artisan to **create** a **table** named **sometable**.

To execute the script, use the command below :

```bash
docker-compose exec -u devuser php php artisan migrate
```

You can check your database to verify the existence of the **sometables** table.

If you're not satisfied with this tablke, you can **rollback** using the command below :

```bash
docker-compose exec -u devuser php php artisan migrate:rollback
```

This command suppresses the **sometables** table.

In fact, executing the *php artisan make:migration create_sometables_table* command creates 2 functions :

- a **up** function
- a **down** function

The **up** function creates the table when invoking the *php artisan migrate* command.

The **down** function "undoes" what the **up** function made previously.

**Alternative** :

You can also use the command below to create the *sometables* table :

```bash
docker-compose exec -u devuser php php artisan make:migration create_some_tables --create=sometables
```

in which *create_some_tables* correspond to **the class name** : createSomeTables

and *--create=sometables* tells Laravel artisan to create a table named **sometables**

**Adding columns to a table**

In order to alter the structure of a table, you can use the command below : 

```bash
docker-compose exec -u devuser php php artisan make:migration alter_structure_to_sometables_table
```

Or, if you don't specify the **to_sometables_table** keywords, you can spécify the name of the table using *--table=sometables* at the end of the command.

Once the **AlterStructureToSometablesTable** class is generated, modify it to add columns :

```php
public function up()
{
    Schema::table('sometables', function(Blueprint $table ) {
        $table->string('title');
        $table->text('content');
    })
}
```

*string* method adds a varchar(255) column to the table.

Since **content** can be more than 255 characters, which is default for varchar, we would use *text*.

In the *down* method of the **AlterStructureToSometablesTable** class, let's specify that we would like to drop specific columns :

```php
    public function down()
    {
        Schema::table('sometables', function (Blueprint $table) {
            $table->dropColumn('title');
            $table->dropColumn('content');
        });
    }
```

or 

```php
    public function down()
    {
        Schema::table('sometables', function (Blueprint $table) {
            $table->dropColumn(['title', 'content']);
        });
    }
```

since the *dropColumn* method accepts an array as a parameter.

To execute the script, use the command below :

```bash
docker-compose exec -u devuser php php artisan migrate
```

## USING ELOQUENT TO INTERACT WITH THE DATABASE

Eloquent is Laravel ORM (Object Relationship Mapping), which makes a PHP class correspond to a table.

Exemple : PHP class = SomeTable <=> Table Name = some_table

The PHP class SomeModel is called a *model* and is used to store the data and fetch the data of the *some_table* table.

To create the corresponding model to the *some_table* table, let's use the LAravel artisan command :

```bash
docker-compose exec -u devuser php php artisan make:model SomeTables
```

The *php artisan make:model* command creates a model stored inside the **app** folder.

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class SomeTables extends Model
{
    //
}<?php
```

As we said previously, Laravel uses conventions. One of the conventions is the name of the Model class and the corresponding table. In our case, the model named *SomeTables* should correspond to the *some_tables* table. But our table is named *sometables*. Let's correct this using *migration* tool.

We can create a migration script to modify the name of the *sometable* table :

```bash
docker-compose exec -u devuser php php artisan make:migration change_sometables_table_name
```

which creates the script for renaming our table :

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class ChangeSometablesTableName extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        //
    }
...
    public function down()
    {
        //
    }
}
```

We can now create the method to rename the *sometables* table :

```php
    public function up()
    {
        Schema::rename('sometables', 'some_tables');
    }
...
    public function down()
    {
        Schema::rename('some_tables', 'sometables');
    }
```

The *down* method is used to rollback the changes, so iot just does the opposite of the *up* method.

Without renaming the *sometables table, it would have been necessary to force the connection between the *SomeTables*  model and the *sometables* table by precising it into the model class :

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class SomeTables extends Model
{
    protected $table = 'sometables';
}
```

But let's respect the convention and use the *some_tables* as table name. So let's comment out the  

```php
protected $table = 'sometables';
```

line :

```php
// protected $table = 'sometables';
```

Don't forget to apply the changes by executing the *artisan migrate* command :

```bash
docker-compose exec -u devuser php php artisan migrate
```

## PLAYING WITH THE MODEL USING **TINKER** :

### Reading all the records of a table using Tinker

```bash
docker-compose exec -u devuser php php artisan tinker
```

To read all the records from the *some_tables* table :

```
>>> $allrec = App\SomeTables::all();
```

The result is an empty array.

### Adding a record using Tinker

Let's populate the table using tinker :

```
>>> $rec = new App\SomeTables();
```

Let's give the values to the fields :

```
$rec->title='My Title';
```

then :

```
$rec->content='My Content';
```

and let's save this into the database :

```
$rec->save();
```

### Fetching a record using Tinker

Now, if we try to get the records from the table :

```
>>> $allrec = App\SomeTables::all();
```

we get a result :

```
=> Illuminate\Database\Eloquent\Collection {#3012
     all: [
       App\SomeTables {#3011
         id: 1,
         created_at: "2019-11-05 12:23:00",
         updated_at: "2019-11-05 12:23:00",
         title: "My Title",
         content: "My Content",
       },
     ],
   }
```

We can also fetch one single record from the table using the *find()* method, and giving it the *id* of the record :

```
$rec2 = App\SomeTables::find(1);
```

Result : 

```
=> App\SomeTables {#2992
     id: 1,
     created_at: "2019-11-05 12:23:00",
     updated_at: "2019-11-05 12:23:00",
     title: "My Title",
     content: "My Content",
   }
```

We can now read the content of our variable typing its name into tinker :

```
$rec2
```

result :

```
=> App\SomeTables {#3000
     title: "My Title",
     content: "My Content",
     updated_at: "2019-11-05 12:23:00",
     created_at: "2019-11-05 12:23:00",
     id: 1,
   }
```

### Modifying a record using Tinker

We can also modify the value of the fierld of an existing record. Let's do it for our **rec2** :

```
$rec2->title = "Another title";
```

Now we can save the change :


```
$rec2->save;
```

Let's verify if everything is ok :

```
$rec2;
```

Result :

```
=> App\SomeTables {#2992
     id: 1,
     created_at: "2019-11-05 12:23:00",
     updated_at: "2019-11-05 12:23:00",
     title: "Another title",
     content: "My Content",
   }
```

### More actions using Tinker

Now, we have 2 records in our table. We can fetch a record from a table as we fetch an element from an array, using an index :

Let's initialize *$allrec* with all the records from the *SomeTables* table :

```
$allrec = App\SomeTables::all();
```

Result : 

```
=> Illuminate\Database\Eloquent\Collection {#3010
     all: [
       App\SomeTables {#3009
         id: 1,
         created_at: "2019-11-05 12:23:00",
         updated_at: "2019-11-05 12:23:00",
         title: "My Title",
         content: "My Content",
       },
       App\SomeTables {#3016
         id: 2,
         created_at: "2019-11-05 16:14:39",
         updated_at: "2019-11-05 16:14:39",
         title: "Third Title",
         content: "Another Content",
       },
     ],
   }
```

We can fetch the first record this way :

```
$allrec[0];
```

Result : 

```
=> App\SomeTables {#3009
     id: 1,
     created_at: "2019-11-05 12:23:00",
     updated_at: "2019-11-05 12:23:00",
     title: "My Title",
     content: "My Content",
   }
```

If we want to fetch he second record :

```
$allrec[1];
```

Result : 

```
=> App\SomeTables {#3016
     id: 2,
     created_at: "2019-11-05 16:14:39",
     updated_at: "2019-11-05 16:14:39",
     title: "Third Title",
     content: "Another Content",
   }
```

We can also iterate over the collection as we would do for an array :

```
foreach ($allrec as $rec) { echo "{$rec->title}\n"; }
```

Result :

```
My Title
Third Title
```

As *$allrec* is a PHP Object, it has its own method which we can use :

```
$allrec->first() ;
```

Which returns the first record of the table.

```
$allrec->last() ;
```

returns the last record of the table.

### Querying with Tinker

We can write a *where* clause using the *model* and *Tinker* :

```
App\SomeTables::where('title', 'My Title')->get();
```

Result :

```
=> Illuminate\Database\Eloquent\Collection {#3013
     all: [
       App\SomeTables {#3006
         id: 1,
         created_at: "2019-11-05 12:23:00",
         updated_at: "2019-11-05 12:23:00",
         title: "My Title",
         content: "My Content",
       },
     ],
   }👏
```

We c👏here* clause returns more than one record, we can chain *first()* with *where()* :


```
App\SomeTables::where('title', 'My Title')->get()->first();
```

which results in the same as the previous query as we only have one record with the *My Title* value in the *title*.

## USING LARAVEL RESOURCE CONTROLLER

When you want to publish a list on a page and permit the user to click on a line of this list to display the detail of an item for exemple, you can use a table to store the items and then use Laravel Resource Controller to help you generate the links to the detail pages.

Let's take the example of a shop : you can manage a table which contains all the items to sell. It could be an idea to display the list on a page and each line could be a link to the detail page of an item.

Let's build the Laravel Resource Controller to do this. We will create a single controller to handle all the possible routes we might be using for :

- displaying all the items
- adding an item
- storing an item
- displaying the detail of an item
- editing an iem
- modifying an existing item and saving the changes
- deleting an item

### Creating a LAravel Resource Controller

To generate such controller, Laravel provides a command :

```bash
php artisan make:controller NameOfTheController --resource
```

This will generate a controller class named *NameOfTheController* containing all the required methods to do the above listed actions.

Let's create an *ItemController* to use the item contained into ouor *some_tables* table.

```bash
docker-compose exec -u devuser php php artisan make:controller ItemController --resource
```

This results in the creation of the Controller Class, containing the following methods :

- index
- create
- store
- show
- edit
- update
- destroy

### Adding the routes into the web.php route controller

Just add the line above to the web.php file :

```php
Route::resource('/item', 'ItemController');
```

Let's check the routes list :

```bash
$ docker-compose exec -u devuser php php artisan route:list
+--------+-----------+-------------------+---------------+---------------------------------------------+--------------+
| Domain | Method    | URI               | Name          | Action                                      | Middleware   |
+--------+-----------+-------------------+---------------+---------------------------------------------+--------------+
|        | GET|HEAD  | /                 | home          | App\Http\Controllers\HomeController@home    | web          |
|        | GET|HEAD  | api/user          |               | Closure                                     | api,auth:api |
|        | GET|HEAD  | contact           | contact       | App\Http\Controllers\HomeController@contact | web          |
|        | GET|HEAD  | posts             | posts.index   | App\Http\Controllers\PostController@index   | web          |
|        | POST      | posts             | posts.store   | App\Http\Controllers\PostController@store   | web          |
|        | GET|HEAD  | posts/create      | posts.create  | App\Http\Controllers\PostController@create  | web          |
|        | GET|HEAD  | posts/{post}      | posts.show    | App\Http\Controllers\PostController@show    | web          |
|        | PUT|PATCH | posts/{post}      | posts.update  | App\Http\Controllers\PostController@update  | web          |
|        | DELETE    | posts/{post}      | posts.destroy | App\Http\Controllers\PostController@destroy | web          |
|        | GET|HEAD  | posts/{post}/edit | posts.edit    | App\Http\Controllers\PostController@edit    | web          |
+--------+-----------+-------------------+---------------+---------------------------------------------+--------------+
```

- The *index* will display the list of items.
- The *store* method will save the informatiopn about the item in the table
- The *create* method will display a form in order to create a new item
- The *show* method will display an individual item
- The *update* method will save the changes
- The *destroy* will delete the item from the table
- The *edit* will display a form to edit an existing item

**Urls** and **actions** are generated out of the box in the controller class.

### How to limit the methods of a Resource Controller

If you don't want to use all the methods generated, you can limit their use by using the *only()* method in the *web.php* route class. The authorized methods are specified in an array as a parameter :

```pĥp
Route::resource('/item', 'ItemController')->only(['index', 'show']);
```

The *route:list* Laravel artisan command now returns :

```bash
$ docker-compose exec -u devuser php php artisan route:list
+--------+----------+-------------+------------+---------------------------------------------+--------------+
| Domain | Method   | URI         | Name       | Action                                      | Middleware   |
+--------+----------+-------------+------------+---------------------------------------------+--------------+
|        | GET|HEAD | /           | home       | App\Http\Controllers\HomeController@home    | web          |
|        | GET|HEAD | api/user    |            | Closure                                     | api,auth:api |
|        | GET|HEAD | contact     | contact    | App\Http\Controllers\HomeController@contact | web          |
|        | GET|HEAD | item        | item.index | App\Http\Controllers\ItemController@index   | web          |
|        | GET|HEAD | item/{item} | item.show  | App\Http\Controllers\ItemController@show    | web          |
+--------+----------+-------------+------------+---------------------------------------------+--------------+
```

We can now implement the *index()* method to display the item list using the *SomeTable Model*:

```php
    public function index()
    {
       dd(\App\SomeTables::all());
    }
```

As we don't have a view to handle the list yet, we just use the **dd** (dump & die) method that just displays the content passed to it (the items) and die.

If we connect to the application url specifying the controller name, we get :

```json
http://127.0.0.1:8080/item

Collection {#259 ▼
  #items: array:2 [▼
    0 => SomeTables {#260 ▼
      #connection: "pgsql"
      #table: "some_tables"
      #primaryKey: "id"
      #keyType: "int"
      +incrementing: true
      #with: []
      #withCount: []
      #perPage: 15
      +exists: true
      +wasRecentlyCreated: false
      #attributes: array:5 [▼
        "id" => 1
        "created_at" => "2019-11-05 12:23:00"
        "updated_at" => "2019-11-05 12:23:00"
        "title" => "My Title"
        "content" => "My Content"
      ]
      #original: array:5 [▶]
      #changes: []
      #casts: []
      #dates: []
      #dateFormat: null
      #appends: []
      #dispatchesEvents: []
      #observables: []
      #relations: []
      #touches: []
      +timestamps: true
      #hidden: []
      #visible: []
      #fillable: []
      #guarded: array:1 [▼
        0 => "*"
      ]
    }
    1 => SomeTables {#261 ▼
      #connection: "pgsql"
      #table: "some_tables"
      #primaryKey: "id"
      #keyType: "int"
      +incrementing: true
      #with: []
      #withCount: []
      #perPage: 15
      +exists: true
      +wasRecentlyCreated: false
      #attributes: array:5 [▶]
      #original: array:5 [▼
        "id" => 2
        "created_at" => "2019-11-05 16:14:39"
        "updated_at" => "2019-11-05 16:14:39"
        "title" => "Third Title"
        "content" => "Another Content"
      ]
      #changes: []
      #casts: []
      #dates: []
      #dateFormat: null
      #appends: []
      #dispatchesEvents: []
      #observables: []
      #relations: []
      #touches: []
      +timestamps: true
      #hidden: []
      #visible: []
      #fillable: []
      #guarded: array:1 [▶]
    }
  ]
}
```
The 2 records of the *some_table* table are shown in each *attributes* section of the resulting page.

Let's now implement the *show()* method of the controller :

```php
    public function show($id)
    {
        dd(\App\SomeTables::find($id));
    }
```

If we now call the application url specifying the controller name and the id of a record (1 for example), we get :

```json
SomeTables {#259 ▼
  #connection: "pgsql"
  #table: "some_tables"
  #primaryKey: "id"
  #keyType: "int"
  +incrementing: true
  #with: []
  #withCount: []
  #perPage: 15
  +exists: true
  +wasRecentlyCreated: false
  #attributes: array:5 [▼
    "id" => 1
    "created_at" => "2019-11-05 12:23:00"
    "updated_at" => "2019-11-05 12:23:00"
    "title" => "My Title"
    "content" => "My Content"
  ]
  #original: array:5 [▶]
  #changes: []
  #casts: []
  #dates: []
  #dateFormat: null
  #appends: []
  #dispatchesEvents: []
  #observables: []
  #relations: []
  #touches: []
  +timestamps: true
  #hidden: []
  #visible: []
  #fillable: []
  #guarded: array:1 [▶]
}
```

The record is detailed in the *attributes* section of the resulting page.

We have to write the namespace *App* each time we want to call the *SomeTables*, which is not really convenient. It could be cumbersome if the namespace is longer. To avoid having to type the namespace, we can import the class using the *use* statement just after the namespace:

```php
namespace App\Http\Controllers;
 
use App\SomeTables
```

We can now type the class name without the namespace:

```php
    public function index()
    {
        dd(SomeTables::all());
    }
```

...

```php
    public function show($id)
    {
        dd(SomeTables::find($id));
    }
```