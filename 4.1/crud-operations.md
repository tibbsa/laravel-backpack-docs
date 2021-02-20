# Operations

---

When creating a CRUD Panel, your ```EntityCrudController``` (where Entity = your model name) is extending ```CrudController```. **By default, no operations are enabled.** No routes are registered.

To use an operation, you need to use the operation trait on your controller. For example, to enable the List operation:

```php
<?php

namespace App\Http\Controllers\Admin;

use Backpack\CRUD\app\Http\Controllers\CrudController;

class ProductCrudController extends CrudController
{
    use \Backpack\CRUD\app\Http\Controllers\Operations\ListOperation;
}
```

**Operations are traits that add functionality to that controller**. Most operations will have:
- routes inside a ```setupOperationNameRoutes()```; this gets called in your ```routes/backpack/custom.php``` by the ```Route::crud('product', 'ProductCrudController``` macro, which determines which routes to register for that CrudController;
- default setup inside a ```setupOperationNameDefaults()``` method, that gets called automatically by CrudController when you use that operation on a controller;
- methods that return views, or perform certain operations;

Of course, you can easily [add custom operations](/#creating-a-custom-operation).

<a name="supported-operations"></a>
## Standard Operations

No operations are enabled by default.

But Backpack does provide the logic for the most common operations admins perform on Eloquent model. You just need to use it (and maybe configure it) in your controller.

Operations provided by Backpack:
- [List](/docs/{{version}}/crud-operation-list-entries) - allows the admin to see all entries for an Eloquent model, with pagination, search, filters;
- [Create](/docs/{{version}}/crud-operation-create) - allows the admin to add a new entry;
- [Update](/docs/{{version}}/crud-operation-update) - allows the admin to edit an existing entry;
- [Show](/docs/{{version}}/crud-operation-show) - allows the admin to preview an entry;
- [Delete](/docs/{{version}}/crud-operation-delete) - allows the admin to remove and entry;
- [BulkDelete](/docs/{{version}}/crud-operation-delete) - allows the admin to remove multiple entries in one go;
- [Clone](/docs/{{version}}/crud-operation-clone) - allows the admin to make a copy of a database entry;
- [BulkClone](/docs/{{version}}/crud-operation-clone) - allows the admin to make a copy of multiple database entries in one go;
- [Reorder](/docs/{{version}}/crud-operation-reorder) - allows the admin to reorder & nest all entries of a model, in a hierarchy tree;
- [Revisions](/docs/{{version}}/crud-operation-revisions) - shows an audit log of all changes to an entry, and allows the admin to undo modifications;


<a name="operation-actions"></a>
### Operation Actions

Each Operation is actually _a trait_, which can be used on CrudControllers. This trait can contain one or more methods (or functions). Since Laravel calls each Controller method an _action_, that means each _Operation_ can have one or many _actions_. For example, we have the ```create``` operation and two actions: ```create()``` and ```store()```.

```php
trait CreateOperation
{
    public function create()
    {
        // ...
    }

    public function store()
    {
        // ...
    }
}
```

An action can do something with AJAX and return true/false, it can return a view, or whatever else you can do inside a controller method. Notice that it's a ```public``` method - which is a Laravel requirement, in order to point a route to it.

You can check which action is currently being performed using the [standard Laravel Route API](https://laravel.com/api/8.x/Illuminate/Routing/Route.html):

- ```\Route::getCurrentRoute()->getAction()``` or ```$this->crud->getRequest()->route()->getAction()```:
```
array:8 [▼
  "middleware" => array:2 [▼
    0 => "web"
    1 => "admin"
  ]
  "as" => "crud.monster.index"
  "uses" => "App\Http\Controllers\Admin\MonsterCrudController@index"
  "operation" => "list"
  "controller" => "App\Http\Controllers\Admin\MonsterCrudController@index"
  "namespace" => "App\Http\Controllers\Admin"
  "prefix" => "admin"
  "where" => []
]
```
- ```\Route::getCurrentRoute()->getActionName()``` or ```$this->crud->getRequest()->route()->getActionName()```:
```
App\Http\Controllers\Admin\MonsterCrudController@index
```
- ```\Route::getCurrentRoute()->getActionMethod()``` or ```$this->crud->getRequest()->route()->getActionMethod()```:
```
index
```

You can also use the shortcuts on the CrudPanel object:
```php
$this->crud->getActionMethod(); // returns the method on the controller that was called by the route; ex: create(), update(), edit() etc;
$this->crud->actionIs('create'); // checks if the controller method given is the one called by the route
```

<a name="titles-heading-and-subheadings"></a>
### Titles, Headings and Subheadings

For standard CRUD operations, each _action_ that shows an interface uses some texts to show the user what page, operation or action he is currently performing:
- **Title** - page title, shown in the browser's title bar;
- **Heading** - biggest heading on page;
- **Subheading** - short description of the current page, sits beside the heading;

![CRUD Operation Headings](https://backpackforlaravel.com/uploads/docs-4-0/operations/headings.png)

You can get and set the above using:
```php
// Getters
$this->crud->getTitle('create'); // get the Title for the create action
$this->crud->getHeading('create'); // get the Heading for the create action
$this->crud->getSubheading('create'); // get the Subheading for the create action

// Setters
$this->crud->setTitle('some string', 'create'); // set the Title for the create action
$this->crud->setHeading('some string', 'create'); // set the Heading for the create action
$this->crud->setSubheading('some string', 'create'); // set the Subheading for the create action
```

These methods are usually useful inside actions, not in ```setup()```. Since action methods are called _after_ ```setup()```, any call to these getters and setters in ```setup()``` would get overwritten by the call in the action.

<a name="access-to-operations"></a>
### Handling Access to Operations

Admins are allowed to do an operation or not using a very simple system: ```$crud->settings['operation_name']['access']``` will either be ```true``` or ```false```. When you enable a stock Backpack operation by doing ```use SomeOperation;``` on your controller, all operations will run ```$this->crud->allowAccess('operation_name');```, which will toggle that variable to ```true```.

You can easily add or remove elements to this access array in your ```setup()``` method, or your custom methods, using:
```php
$this->crud->allowAccess('operation_name');
$this->crud->allowAccess(['list', 'update', 'delete']);
$this->crud->denyAccess('operation');
$this->crud->denyAccess(['update', 'create', 'delete']);

$this->crud->hasAccess('operation_name'); // returns true/false
$this->crud->hasAccessOrFail('create'); // throws 403 error
$this->crud->hasAccessToAll(['create', 'update']); // returns true/false
$this->crud->hasAccessToAny(['create', 'update']); // returns true/false
```

<a name="operation-routes"></a>
### Operation Routes

Starting with Backpack 4.0, routes can be defined in the CrudController. Your ```routes/backpack/custom.php``` file will have calls like ```Route::crud('product', 'ProductCrudController');```. This ```Route::crud()``` is a macro that will go to that controller and run all the methods that look like ```setupXxxRoutes()```. That means each operation can have its own method to define the routes it needs. And they do - if you check out the code of any operation, you'll see every one of them has a ```setupOperationNameRoutes()``` method. 

If you want to add a new route to your controller, there are two ways to do it:
1. Add a route in your ```routes/backpack/custom.php```;
2. Add a method following the ```setupXxxRoutes()``` convention to your controller;

Inside a ```setupOperationNameRoutes()```, you'll notice that's also where we define the operation name:

```php
    protected function setupShowRoutes($segment, $routeName, $controller)
    {
        Route::get($segment.'/{id}/show', [
            'as'        => $routeName.'.show',
            'uses'      => $controller.'@show',
            'operation' => 'show',
        ]);
    }
```

<a name="getting-and-setting-operation"></a>
### Getting an Operation Name

Once an operation name has been set using that route, you can do ```$crud->getOperation()``` inside your views and do things according to this.


<a name="creating-a-custom-operation"></a>
## Creating a Custom Operation

<a name="create-operation-command"></a>
### Command-line Tool

If you've installed ```backpack/generators```, you can do ```php artisan backpack:crud-operation {OperationName}``` to generate an empty operation trait, that you can edit and use on your CrudControllers. For example:

```bash
php artisan backpack:crud-operation Comment
```

Will generate ```app/Http/Controllers/Admin/Operations/CommentOperation``` with the following contents:

```php
<?php

namespace App\Http\Controllers\Admin\Operations;

use Illuminate\Support\Facades\Route;

trait CommentOperation
{
    /**
     * Define which routes are needed for this operation.
     *
     * @param string $segment    Name of the current entity (singular). Used as the first URL segment.
     * @param string $routeName  Prefix of the route name.
     * @param string $controller Name of the current CrudController.
     */
    protected function setupCommentRoutes($segment, $routeName, $controller)
    {
        Route::get($segment.'/comment', [
            'as'        => $routeName.'.comment',
            'uses'      => $controller.'@comment',
            'operation' => 'comment',
        ]);
    }

    /**
     * Add the default settings, buttons, etc that this operation needs.
     */
    protected function setupCommentDefaults()
    {
        $this->crud->allowAccess('comment');

        $this->crud->operation('comment', function () {
            $this->crud->loadDefaultOperationSettingsFromConfig();
        });

        $this->crud->operation('list', function () {
            // $this->crud->addButton('top', 'comment', 'view', 'crud::buttons.comment');
            // $this->crud->addButton('line', 'comment', 'view', 'crud::buttons.comment');
        });
    }

    /**
     * Show the view for performing the operation.
     *
     * @return Response
     */
    public function comment()
    {
        $this->crud->hasAccessOrFail('comment');

        // prepare the fields you need to show
        $this->data['crud'] = $this->crud;
        $this->data['title'] = $this->crud->getTitle() ?? 'comment '.$this->crud->entity_name;

        // load the view
        return view("crud::operations.comment", $this->data);
    }
}
```

You'll notice the generated operation has:
- a GET route (inside ```setupCommentRoutes()```);
- a method that sets the defaults for this operation (```setupCommentDefaults()```);
- a method to perform the operation, or show an interface (```comment()```);

You can customize these to fit the operation you have in mind, then ```use \App\Http\Controllers\Admin\Operations\CommentOperation;``` inside the CrudControllers where you want the operation.

<a name="contents-of-a-custom-operation"></a>
### Contents of a Custom Operation

Thanks to [Backpack's simple architecture](/docs/{{version}}/crud-basics#architecture), each CRUD panel uses a controller and a route, that are placed inside your project. That means you hold the keys to how this controller works.

To add an operation to an ```EntityCrudController```, you can:
- decide on your operation name; for example... "publish";
- create a method that loads the routes inside your controller:
```php
    protected function setupPublishRoutes($segment, $routeName, $controller)
    {
        Route::get($segment.'/{id}/publish', [
            'as'        => $routeName.'.publish',
            'uses'      => $controller.'@publish',
            'operation' => 'publish',
        ]);
    }
```
- create a method that performs the operation you want:
```php
    public function publish()
    {
        // do something
        // return something
    }
```
- [add a new button for this operation to the List view](/docs/{{version}}/crud-buttons#creating-a-custom-button), or enable access to it, inside a ```setupPublishDefaults()``` method:
```php
    protected function setupPublishDefaults()
    {
        $this->crud->allowAccess('publish');

        $this->crud->operation('list', function () {
            $this->crud->addButton('line', 'publish', 'view', 'buttons.publish', 'beginning');
        });
    }
```

Take a look at the examples below for a better picture and code examples.

If you intend to reuse this operation across multiple controllers, you can group all the methods above in a trait, say ```PublishOperation.php``` and then just use that trait on the controllers where you need the operation:

```php
<?php

namespace App\Http\Controllers\Admin\CustomOperations;

use Illuminate\Support\Facades\Route;

trait PublishOperation
{
    protected function setupPublishRoutes($segment, $routeName, $controller)
    {
        Route::get($segment.'/{id}/publish', [
            'as'        => $routeName.'.publish',
            'uses'      => $controller.'@publish',
            'operation' => 'publish',
        ]);
    }

    protected function setupPublishDefaults()
    {
        $this->crud->allowAccess('publish');

        $this->crud->operation('list', function () {
            $this->crud->addButton('line', 'publish', 'view', 'buttons.publish', 'beginning');
        });
    }

    public function publish()
    {
        // do something
        // return something
    }
}
```

In the example above, you could just do ```use \App\Http\Controllers\Admin\CustomOperations\PublishOperation;``` on any EntityCrudController, and your operation will be added - complete with routes, buttons, access, actions, everything.

<a name="access-to-custom-operations"></a>
### Access to Custom Operations

Since you're creating a new operation, in terms of restricting access you can:
1. allow access to this new operation depending on access to a default operation (usually if the admin can ```update```, he's ok to perform custom operations);
2. customize access to this particular operation, by just using a different key than the default ones; for example, you can allow access by using ```$this->crud->allowAccess('publish')``` in your ```setup()``` method, then check for access to that operation using ```$this->crud->hasAccess('publish')```;

<a name="operation-features"></a>
### Adding Settings to the CrudPanel object

#### Using the Settings API

Anything an operation does to configure itself, or process information, should be stored inside ```$this->crud->settings``` . It's an associative array, and you can add/change things using the Settings API:

```php
// for the operation that is currently being performed
$this->crud->setOperationSetting('show_title', true);
$this->crud->getOperationSetting('show_title');
$this->crud->hasOperationSetting('show_title');

// for a particular operation, pass the operation name as a last parameter 
$this->crud->setOperationSetting('show_title', true, 'create');
$this->crud->getOperationSetting('show_title', 'create');
$this->crud->hasOperationSetting('show_title', 'create');

// alternatively, you could use the direct methods with no fallback to the current operation
$this->crud->set('create.show_title', false);
$this->crud->get('create.show_title');
$this->crud->has('create.show_title');
```

#### Using the crud config file

Additionally, operations can load default settings from the config file. You'll notice the ```config/backpack/crud.php``` file contains an array of operations, each with various settings. Those settings there are loaded by the operation as defaults, to allow users to change one setting in the config, and have that default changed across ALL of their CRUDs. If you take a look at the List operation you'll notice this:

```php
    /**
     * Add the default settings, buttons, etc that this operation needs.
     */
    protected function setupListDefaults()
    {
        $this->crud->allowAccess('list');

        $this->crud->operation('list', function () {
            $this->crud->loadDefaultOperationSettingsFromConfig();
        });
    }
```

You can do the same in custom operations. Because this call happens in setupListDefaults(), inside an operation closure, the settings will only be added when that operation is being performed.


<a name="adding-methods-to-the-crud-panel-object"></a>
### Adding Methods to the CrudPanel Object

You can add static methods to this ```$this->crud``` (which is a CrudPanel object) object with ```$this->crud->macro()```, because the object is [macroable](https://unnikked.ga/understanding-the-laravel-macroable-trait-dab051f09172). So you can do:

```php
class MonsterCrudController extends CrudController 
{
   public function setup() 
   {
       $this->crud->macro('doStuff', function($something) {
            echo '<pre>'; var_dump($something); echo '</pre>';
            dd($this);
        });
       $this->crud->macro('getColumnsInTheFormatIWant', function() {
            $columns = $this->columns();
            // ... do something to $columns;
            return $columns;
        });

       // bla-bla-bla the actual setup code
   }
   public function sendEmail() 
   {
      // ...
      $this->crud->doStuff();
      dd($this->crud->getColumnsInTheFormatIWant());
      // ...
   }
   public function markPending() 
   {
      // ...
      $this->crud->doStuff();
      dd($this->crud->getColumnsInTheFormatIWant());
      // ...
   }
}
```

So if you define a custom operation that needs some static methods added to the ```CrudPanel``` object, you can add them. The best place to register your macros in a custom operation would probably be inside your ```setupXxxDefaults()``` method, inside an operation closure. That way, the static methods you add are only added when that operation is being performed. For example:

```php
protected function setupPrintDefaults()
{
    $this->crud->allowAccess('print');
    
    $this->crud->operation('print', function() {
       $this->crud->macro('getColumnsInTheFormatIWant', function() {
            $columns = $this->columns();
            // ... do something to $columns;
            return $columns;
        });
    });
}
```

With the example above, you'll be able to use ```$this->crud->getColumnsInTheFormatIWant()``` inside your operation actions.

<a name="using-a-feature-from-another-operation"></a>
### Using a feature from another operation

Anything an operation does to configure itself, or process information, is stored on the ```$this->crud->settings``` property. Operation features (ex: fields, columns, buttons, filters, etc) are created in such a way that all they do is add an entry in settings, for an operation, and manipulate it. That means there is nothing stopping you from using a feature from one operation in a different operation.

If you create a Print operation, and want to use the ```columns``` feature that List and Show use, you can just go ahead and do ```$this->crud->addColumn()``` calls inside your operation. You'll notice the columns are stored inside ```$this->crud->settings['print.columns']```, so they're completely different from the ones in the List or Show operation. You'll need to actually do something with the columns you added, inside your operation methods or views - of course.


<a name="examples"></a>
### Examples

<a name="creatig-a-new-operation-with-no-interface"></a>
#### Creating a New Operation With No Interface

Let's say we have a ```UserCrudController``` and we want to create a simple ```Clone``` operation, which would create another entry with the same info. So very similar to ```Delete```. What we need to do is:

1. Create a route for this operation - as we've learned above we can do that in a ```setupXxxRoutes()``` method:

```php
    protected function setupPublishRoutes($segment, $routeName, $controller)
    {
        Route::get($segment.'/{id}/clone', [
            'as'        => $routeName.'.clone',
            'uses'      => $controller.'@clone',
            'operation' => 'clone',
        ]);
    }
```

2. Add the method inside ```UserCrudController```:

```php
public function clone($id) 
{
    $this->crud->hasAccessOrFail('create');
    $this->crud->setOperation('Clone');

    $clonedEntry = $this->crud->model->findOrFail($id)->replicate();

    return (string) $clonedEntry->push();
}
```

3. Create a button for this method. Since our operation is similar to "Delete", lets start from that one and customize what we need. The button should clone the entry using an AJAX call. No need to load another page for an operation this simple. We'll create a ```resources\views\vendor\backpack\crud\buttons\clone.blade.php``` file:

```php
@if ($crud->hasAccess('create'))
	<a href="javascript:void(0)" onclick="cloneEntry(this)" data-route="{{ url($crud->route.'/'.$entry->getKey().'/clone') }}" class="btn btn-xs btn-default" data-button-type="clone"><i class="fa fa-clone"></i> Clone</a>
@endif

{{-- Button Javascript --}}
{{-- - used right away in AJAX operations (ex: List) --}}
{{-- - pushed to the end of the page, after jQuery is loaded, for non-AJAX operations (ex: Show) --}}
@push('after_scripts') @if (request()->ajax()) @endpush @endif
<script>
	if (typeof cloneEntry != 'function') {
	  $("[data-button-type=clone]").unbind('click');

	  function cloneEntry(button) {
	      // ask for confirmation before deleting an item
	      // e.preventDefault();
	      var button = $(button);
	      var route = button.attr('data-route');

          $.ajax({
              url: route,
              type: 'POST',
              success: function(result) {
                  // Show an alert with the result
                  new Noty({
                    type: "success",
                    text: "<strong>Entry cloned</strong><br>A new entry has been added, with the same information as this one."
                  }).show();

                  // Hide the modal, if any
                  $('.modal').modal('hide');

                  if (typeof crud !== 'undefined') {
                    crud.table.ajax.reload();
                  }
              },
              error: function(result) {
                  // Show an alert with the result
                  new Noty({
                    type: "warning",
                    text: "<strong>Cloning failed</strong><br>The new entry could not be created. Please try again."
                  }).show();
              }
          });
      }
	}

	// make it so that the function above is run after each DataTable draw event
	// crud.addFunctionToDataTablesDrawEventQueue('cloneEntry');
</script>
@if (!request()->ajax()) @endpush @endif
```

4. We can now actually add this button to our ```UserCrudController::setupCloneOperation()``` method, or our ```setupCloneDefaults()``` method:

```php
protected function setupCloneDefaults() {
  $this->crud->allowAccess('clone');

  $this->crud->operation(['list', 'show'], function () {
    $this->crud->addButtonFromView('line', 'clone', 'clone', 'beginning');
  });
}
```

>Of course, **if you plan to re-use this operation on another EntityCrudController**, it's a good idea to isolate the method inside a trait, then use that trait on each EntityCrudController where you want the operation to work.

<a name="creating-a-new-operation-with-an-interface"></a>
#### Creating a New Operation With An Interface

Let's say we have a ```UserCrudController``` and we want to create a simple ```Moderate``` operation, where we show a form where the admin can add his observations and what not. In this respect, it should be similar to ```Update``` - the button should lead to a separate form, then that form will probably have a Save button. So when creating the methods, we should look at ```CrudController::edit()``` and ```CrudController::updateCrud()``` for working examples.

What we need to do is:

1. Create routes for this operation - we can do that using the ```setupOperationNameRoutes()``` convention inside a ```UserCrudController```:

```php
    protected function setupModerateRoutes($segment, $routeName, $controller)
    {
        Route::get($segment.'/{id}/moderate', [
            'as'        => $routeName.'.getModerate',
            'uses'      => $controller.'@getModerateForm',
            'operation' => 'moderate',
        ]);
        Route::post($segment.'/{id}/moderate', [
            'as'        => $routeName.'.postModerate',
            'uses'      => $controller.'@postModerateForm',
            'operation' => 'moderate',
        ]);
    }
```

2. Add the methods inside ```UserCrudController```:

```php
public function getModerateForm($id) 
{
    $this->crud->hasAccessOrFail('update');
    $this->crud->setOperation('Moderate');

    // get the info for that entry
    $this->data['entry'] = $this->crud->getEntry($id);
    $this->data['crud'] = $this->crud;
    $this->data['title'] = 'Moderate '.$this->crud->entity_name;

    return view('vendor.backpack.crud.moderate', $this->data);
}

public function postModerateForm(Request $request = null)
{
    $this->crud->hasAccessOrFail('update');

    // TODO: do whatever logic you need here
    // ...
    // You can use 
    // - $this->crud
    // - $this->crud->getEntry($id)
    // - $request
    // ...

    // show a success message
    \Alert::success('Moderation saved for this entry.')->flash();

    return \Redirect::to($this->crud->route);
}
```

3. Create the ```/resources/views/vendor/backpack/crud/moderate.php``` blade file, which shows the moderate form and what not. Best to start from the ```edit.blade.php``` file and customize:

```html
@extends(backpack_view('layouts.top_left'))

@php
  $defaultBreadcrumbs = [
    trans('backpack::crud.admin') => backpack_url('dashboard'),
    $crud->entity_name_plural => url($crud->route),
    'Moderate' => false,
  ];

  // if breadcrumbs aren't defined in the CrudController, use the default breadcrumbs
  $breadcrumbs = $breadcrumbs ?? $defaultBreadcrumbs;
@endphp

@section('header')
  <section class="container-fluid">
    <h2>
        <span class="text-capitalize">{!! $crud->getHeading() ?? $crud->entity_name_plural !!}</span>
        <small>{!! $crud->getSubheading() ?? 'Moderate '.$crud->entity_name !!}.</small>

        @if ($crud->hasAccess('list'))
          <small><a href="{{ url($crud->route) }}" class="hidden-print font-sm"><i class="fa fa-angle-double-left"></i> {{ trans('backpack::crud.back_to_all') }} <span>{{ $crud->entity_name_plural }}</span></a></small>
        @endif
    </h2>
  </section>
@endsection

@section('content')
<div class="row">
    <div class="col-md-8 col-md-offset-2">
          <div class="card">
            <div class="card-header">
                <h3 class="card-title">Moderate</h3>
            </div>
            <div class="card-body row">
              Something in the card body
            </div><!-- /.card-body -->

            <div class="card-footer">
                Something in the box footer
            </div><!-- /.card-footer-->
          </div><!-- /.card -->
          </form>
    </div>
</div>
@endsection

```

4. Create a button for this operation. Since our operation is similar to "Update", lets start from that one and customize what we need. The button should just take the admin to the route that shows the Moderate form. Nothing fancy. We'll create a ```resources\views\vendor\backpack\crud\buttons\moderate.blade.php``` file:

```php
@if ($crud->hasAccess('moderate'))
    <a href="{{ url($crud->route.'/'.$entry->getKey().'/moderate') }}" class="btn btn-sm btn-link"><i class="fa fa-list"></i> Moderate</a>
@endif
```

4. We can now actually add this button to our ```UserCrudController::setup()```, to register that button inside the List operation:

```php
$this->crud->operation('list', function() {
  $this->crud->addButtonFromView('line', 'moderate', 'moderate', 'beginning');  
});
```

Or better yet, we can do this inside a ```setupModerateDefaults()``` method, which gets called automatically by CrudController when the ```moderate``` operation is being performed (thanks to the operation name set on the routes):

```php
protected function setupModerateDefaults()
{
  $this->crud->allowAccess('moderate');

  $this->crud->operation('list', function() {
    $this->crud->addButtonFromView('line', 'moderate', 'moderate', 'beginning');  
  });
}
```

>Of course, **if you plan to re-use this operation on another EntityCrudController**, it's a good idea to isolate the method inside a trait, then use that trait on each EntityCrudController where you want the operation to be enabled.

```php
<?php

namespace App\Http\Controllers\Admin\CustomOperations;

use Illuminate\Support\Facades\Route;

trait ModerateOperation
{
    protected function setupModerateRoutes($segment, $routeName, $controller)
    {
        Route::get($segment.'/{id}/moderate', [
            'as'        => $routeName.'.getModerate',
            'uses'      => $controller.'@getModerateForm',
            'operation' => 'moderate',
        ]);
        Route::post($segment.'/{id}/moderate', [
            'as'        => $routeName.'.postModerate',
            'uses'      => $controller.'@postModerateForm',
            'operation' => 'moderate',
        ]);
    }

    protected function setupmoderateDefaults()
    {
        $this->crud->allowAccess('moderate');

        $this->crud->operation('list', function() {
          $this->crud->addButtonFromView('line', 'moderate', 'moderate', 'beginning');  
        });
    }

    public function getModerateForm($id) 
    {
        $this->crud->hasAccessOrFail('update');
        $this->crud->setOperation('Moderate');

        // get the info for that entry
        $this->data['entry'] = $this->crud->getEntry($id);
        $this->data['crud'] = $this->crud;
        $this->data['title'] = 'Moderate '.$this->crud->entity_name;

        return view('vendor.backpack.crud.moderate', $this->data);
    }

    public function postModerateForm(Request $request = null)
    {
        $this->crud->hasAccessOrFail('update');

        // TODO: do whatever logic you need here
        // ...
        // You can use 
        // - $this->crud
        // - $this->crud->getEntry($id)
        // - $request
        // ...

        // show a success message
        \Alert::success('Moderation saved for this entry.')->flash();

        return \Redirect::to($this->crud->route);
    }
}
```

<a name="creating-a-new-operation-with-bulk-action"></a>
#### Creating a New Operation With a Bulk Action (No Interface)

Say we want to create a ```BulkClone``` operation, with a button which clones multiple entries at the same time. So very similar to our ```BulkDelete```. What we need to do is:

1. Create a new button:

```html
@if ($crud->hasAccess('bulkClone') && $crud->get('list.bulkActions'))
  <a href="javascript:void(0)" onclick="bulkCloneEntries(this)" class="btn btn-sm btn-secondary bulk-button"><i class="fa fa-clone"></i> Clone</a>
@endif

@push('after_scripts')
<script>
  if (typeof bulkCloneEntries != 'function') {
    function bulkCloneEntries(button) {

        if (typeof crud.checkedItems === 'undefined' || crud.checkedItems.length == 0)
        {
            new Noty({
            type: "warning",
            text: "<strong>{{ trans('backpack::crud.bulk_no_entries_selected_title') }}</strong><br>{{ trans('backpack::crud.bulk_no_entries_selected_message') }}"
          }).show();

          return;
        }

        var message = "Are you sure you want to clone these :number entries?";
        message = message.replace(":number", crud.checkedItems.length);

        // show confirm message
        swal({
        title: "{{ trans('backpack::base.warning') }}",
        text: message,
        icon: "warning",
        buttons: {
          cancel: {
          text: "{{ trans('backpack::crud.cancel') }}",
          value: null,
          visible: true,
          className: "bg-secondary",
          closeModal: true,
        },
          delete: {
          text: "Clone",
          value: true,
          visible: true,
          className: "bg-primary",
        }
        },
      }).then((value) => {
        if (value) {
          var ajax_calls = [];
              var clone_route = "{{ url($crud->route) }}/bulk-clone";

          // submit an AJAX delete call
          $.ajax({
            url: clone_route,
            type: 'POST',
            data: { entries: crud.checkedItems },
            success: function(result) {
              // Show an alert with the result
                    new Noty({
                    type: "success",
                    text: "<strong>Entries cloned</strong><br>"+crud.checkedItems.length+" new entries have been added."
                  }).show();

              crud.checkedItems = [];
              crud.table.ajax.reload();
            },
            error: function(result) {
              // Show an alert with the result
                    new Noty({
                    type: "danger",
                    text: "<strong>Cloning failed</strong><br>One or more entries could not be created. Please try again."
                  }).show();
            }
          });
        }
      });
      }
  }
</script>
@endpush
```

2. Create a method in your EntityCrudController (or in a trait, if you want to re-use it for multiple CRUDs):

```php
    public function bulkClone()
    {
        $this->crud->hasAccessOrFail('create');

        $entries = $this->crud->getRequest()->input('entries');
        $clonedEntries = [];

        foreach ($entries as $key => $id) {
            if ($entry = $this->crud->model->find($id)) {
                $clonedEntries[] = $entry->replicate()->push();
            }
        }

        return $clonedEntries;
    }
```

3. Add a route to point to this new method:

```php
protected function setupBulkCloneRoutes($segment, $routeName, $controller)
{
    Route::post($segment.'/bulk-clone', [
        'as'        => $routeName.'.bulkClone',
        'uses'      => $controller.'@bulkClone',
        'operation' => 'bulkClone',
    ]);
}
```

4. Setup the default features we need for the operation to work:

```php
protected function setupBulkCloneDefaults()
{
    $this->crud->allowAccess('bulkClone');

    $this->crud->operation('list', function () {
        $this->crud->enableBulkActions();
        $this->crud->addButton('bottom', 'bulk_clone', 'view', 'bulk_clone', 'beginning');
    });
}
```


Now there's a Clone button on our List bottom stack, that works as expected for multiple entries.

The button makes one call for all entries, and only triggers one notification. If you would rather make a call for each entry, you can use something like below:

```html
@if ($crud->hasAccess('create') && $crud->bulk_actions)
  <a href="javascript:void(0)" onclick="bulkCloneEntries(this)" class="btn btn-sm btn-secondary bulk-button"><i class="fa fa-clone"></i> Clone</a>
@endif

@push('after_scripts')
<script>
  if (typeof bulkCloneEntries != 'function') {
    function bulkCloneEntries(button) {

        if (typeof crud.checkedItems === 'undefined' || crud.checkedItems.length == 0)
        {
          new PNotify({
                title: "{{ trans('backpack::crud.bulk_no_entries_selected_title') }}",
                text: "{{ trans('backpack::crud.bulk_no_entries_selected_message') }}",
                type: "warning"
            });

          return;
        }

        var message = "Are you sure you want to clone these :number entries?";
        message = message.replace(":number", crud.checkedItems.length);

        // show confirm message
        if (confirm(message) == true) {
            var ajax_calls = [];

            // for each crud.checkedItems
            crud.checkedItems.forEach(function(item) {
              var clone_route = "{{ url($crud->route) }}/"+item+"/clone";

              // submit an AJAX delete call
              ajax_calls.push($.ajax({
                  url: clone_route,
                  type: 'POST',
                  success: function(result) {
                      // Show an alert with the result
                      new Noty({
                        type: "success",
                        text: "<strong>Entries cloned</strong><br>"+crud.checkedItems.length+" new entries have been added."
                      }).show();
                  },
                  error: function(result) {
                      // Show an alert with the result
                      new Noty({
                        type: "danger",
                        text: "<strong>Cloning failed</strong><br>One or more entries could not be created. Please try again."
                      }).show();
                  }
              }));

          });

          $.when.apply(this, ajax_calls).then(function ( ajax_calls ) {
              crud.checkedItems = [];
              crud.table.ajax.reload();
        });
        }
      }
  }
</script>
@endpush
```
