# Laravel cheatsheet

## Using Migration to create and alter tables

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

## Using Eloquent to interact with the database

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

## Playing with the model using **Tinker** :

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
   }
```

We can chain the methods. For example if the *where* clause returns more than one record, we can chain *first()* with *where()* :


```
App\SomeTables::where('title', 'My Title')->get()->first();
```

which results in the same as the previous query as we only have one record with the *My Title* value in the *title* field.