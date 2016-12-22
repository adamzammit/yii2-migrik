Migration Generator
=======================================
 - generate migration files (not dumps!) with indexes, and foreign keys, for one table, comma separated list of tables,  by part of table name, for all tables by 
 - generate migrations based on table data - in two ways - as batchInsert Query or as insert via model 
 - generate migrations based on PHPDOC and model properties

###CHANGELOG
22.12.2016 - 2.2.7 version release add fields for setup default values onUpdate and onDelete in relations
17.12.2016 - 2.2.6 version release
     merge [pr](https://github.com/Insolita/yii2-migrik/pull/19) from [shirase](https://github.com/shirase) that fix issue with non-correct gii preview;
     
20.08.2016 - 2.2 version release
 - added new generator by phpdoc annotations; see [annotation syntax](#annotation-syntax)
 
15.08.2016 - 2.1 version release 
 - added ability to generate migrations in fluent interface (raw format also available)
 - improved templates; added database initializations
 - improved postresql index retrieving
 - structure improved; separate logic in external classes
 - added ability to set custom class for column generation (@see [Customizing section](#customizing))   
 
13.08.2016 - 2.0 version release with new ability - generate migrations based on table data
__Possible BC__
- class insolita\migrik\gii\Generator was changed on insolita\migrik\gii\StructureGenerator
if you made template customizations - see your gii config and replace old Generator class name

###Installation

The preferred way to install this extension is through [composer](http://getcomposer.org/download/).

Either run

```
php composer.phar require -dev --prefer-dist insolita/yii2-migration-generator:~2.2
```
or 
```
composer require -dev --prefer-dist insolita/yii2-migration-generator:~2.2
```

or add

```
"insolita/yii2-migration-generator": "~2.2"
```

to the require-dev section of your `composer.json` file.


Just install, go to gii and use (By default composer bootstrap hook)


###ANNOTATION SYNTAX

In general the syntax of column definitions is based  on style of yii-migration, only separated by "|" and provide a little more opportunities for reducing code
 - as you see in examples - empty brackets not necessary
 - also shortcut expr() will be replaced to defaultExpression() and default() to defaultValue 
 
You can add annotations in your model(not necessary AR or yii\\base\\Model or Object or stdClass)

`@db (db2)` - specify connection id required for migration 'db' - by default"

`@table ({{%my_table}})`- specify table for migration"

__Supported column annotations:__
 - As separate annotation above class  or above current variable
 
 ```php 
 /**
 * @column (name) string|notNull|default('SomeValue')
 */
 ```
 
 - As addition to @property or @var definition 
 ```
    /**
     * @var int $id @column pk()
    **/
    public $id;
    /**
     * @var string $route @column string(100)|notNull()
    **/
    public $route;
 
 ```

```
/**
 * @property integer    $id         @column pk|comment("Id")
 * @property string     $username   @column string(100)|unique|notNull|default("Vasya")
 * @property string     $email      @column string(200)|unique()|defaultValue("123@mail.ru")
 * @property string     $password   @column string(200)|notNull|expr(null)
 * @property string     $created_at @column string(200)|notNull|expr('CURRENT_TIMESTAMP')
**/
class TestModel extends ActiveRecord{
```

 
###Customizing 
##### Use Own templates
Copy default templates from folders 
   `vendor/insolita/yii2-migration-generator/gii/default_structure //schema migrations`
   `vendor/insolita/yii2-migration-generator/gii/default_data //data migrations`
to some project directory, for example 
   `@backend/gii/templates/migrator_data;`
   `@backend/gii/templates/migrator_schema;`

Change gii configuration like this
```php
$config['modules']['gii'] = [
        'class' => 'yii\gii\Module',
        'allowedIPs' => ['127.0.0.1', 'localhost', '::1'],
        'generators' => [
            'migrik'=>[
                        'class'=>\insolita\migrik\gii\StructureGenerator::class,
                        'templates'=>
                        [
                             'custom'=>'@backend/gii/templates/migrator_schema'
                        ]
            ],
            'migrikdata'=>[
                        'class'=>\insolita\migrik\gii\DataGenerator::class,
                        'templates'=>
                        [        
                            'custom'=>'@backend/gii/templates/migrator_data'
                        ]
            ],
        ]
]
```

##### Use own resolver for definition of columns 
  - create new class, inherited from \insolita\migrik\resolver\*ColumnResolver
    - override required methods, or create methods for exclusive columns based on database types - see insolita\migrik\resolver\BaseColumnResolver resolveColumn() phpdoc and realization
    
##### Use own resolver for definition of  indexes or relations 
  - create new class, inherited from \insolita\migrik\resolver\TableResolver
  - in bootsrap your apps add injection 
  
  ```\Yii::$container->set(IMigrationTableResolver::class, YourTableResolver::class);```
    