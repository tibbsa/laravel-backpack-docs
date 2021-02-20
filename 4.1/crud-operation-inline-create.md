# InlineCreate Operation

---

<a name="about"></a>
## About

This operation allows your admins to add new entries to a database table on-the-fly, from a modal. 

For example:
- if you have an ```ArticleCrudController``` where your user can also select ```Categories```;
- this operation adds the ability to create ```Categories``` right inside the ```ArticleCrudController```'s Create form; 
    - the admin needs to click an Add button
    - a modal will show the form from ```CategoryCrudController```'s Create operation;

![Backpack InlineCreate Operation](https://backpackforlaravel.com/uploads/docs-4-1/release_notes/inline_create_small.gif)


<a name="requirements"></a>
## Requirements

- a working Create operation;
- correctly defined Eloquent relationships on both the primary Model, and the secondary Model;
- a working Fetch operation to retrieve the secondary Model from the primary Model;
- an understanding of what we call "_main_" and "_secondary_" in this case; using the same example as above, where you want to be able to add ```Categories``` in a modal, inside ```ArticleCrudController```'s create&update forms:
    - the _main entity_ would be Article (big form); 
    - the _secondary entity_ would be Category (small form, in a modal);

<a name="how-to-use"></a>
## How to Use

To use the Create operation, you must:

**Step 1. Use the operation trait on your secondary entity's CrudController** (aka. the entity that will gain the ability to be created inline, in our example CategoryCrudController). Make sure you use `InlineCreateOperation` *after* `CreateOperation`:

```php
<?php

namespace App\Http\Controllers\Admin;

use Backpack\CRUD\app\Http\Controllers\CrudController;

class TagCrudController extends CrudController
{
    use \Backpack\CRUD\app\Http\Controllers\Operations\CreateOperation;
    use \Backpack\CRUD\app\Http\Controllers\Operations\InlineCreateOperation;
    
    // notice InlineCreateOperation is used AFTER CreateOperation
    // that's required in order for InlineCreate to re-use whatever
    // CreateOperation has already setup
    
    // OPTIONAL
    // only if you want to make the InlineCreateOperation behave differently 
    // from the CreateOperation, otherwise you can just skip the setup method entirely
    // 
    // protected function setupInlineCreateOperation()
	// {
		// $this->crud->setValidation(StoreRequest::class);
		// $this->crud->addField($field_definition_array);
	// }
}
```

**Step 2. Use [the relationship field](/docs/{{version}}/crud-fields#relationship) inside the ```setupCreateOperation()``` or ```setupUpdateOperation()``` of the main entity** (where you'd like to be able to click a button and a modal shows up, in our example ArticleCrudController), and define ```inline_create``` on it:

```php
// for 1-n relationships (ex: category) - inside ArticleCrudController
[
    'type' => "relationship",
    'name' => 'category', // the method on your model that defines the relationship
    'ajax' => true,
    'inline_create' => true, // assumes the URL will be "/admin/category/inline/create"
]

// for n-n relationships (ex: tags) - inside ArticleCrudController
[
    'type' => "relationship",
    'name' => 'tags', // the method on your model that defines the relationship
    'ajax' => true,
    'inline_create' => [ 'entity' => 'tag' ] // specify the entity in singular
    // that way the assumed URL will be "/admin/tag/inline/create"
]

// OPTIONALS - to customize behaviour
[
    'type' => "relationship",
    'name' => 'tags', // the method on your model that defines the relationship
    'ajax' => true,
    'inline_create' => [ // specify the entity in singular
        'entity' => 'tag', // the entity in singular
        // OPTIONALS
        'force_select' => true, // should the inline-created entry be immediately selected?
        'modal_class' => 'modal-dialog modal-xl', // use modal-sm, modal-lg to change width
        'modal_route' => route('tag-inline-create'), // InlineCreate::getInlineCreateModal()
        'create_route' =>  route('tag-inline-create-save'), // InlineCreate::storeInlineCreate()
        'include_main_form_fields' => ['field1', 'field2'], // pass certain fields from the main form to the modal
    ]
```


**Step 3. OPTIONAL - You can create a ```setupInlineCreateOperation()``` method in the EntityCrudController**, to make the InlineCreateOperation different to the CreateOperation, for example have more/less fields, or different fields. Check out the [Fields API](/docs/{{version}}/crud-fields#fields-api) for a reference of all you can do with them.

<a name="how-it-works"></a>
## How It Works

The ```CreateInline``` operation uses two routes:
- POST to ```/entity-name/inline/create/modal``` -  ```getInlineCreateModal()``` which returns the contents of the Create form, according to how it's been defined by the CreateOperation (in ```setupCreateOperation()```, then overwritten by the InlineCreateOperation (in ```setupInlineCreateOperation()```);
- POST to ```/entity-name/inline/create``` - points to ```storeInlineCreate()``` which does the actual saving in the database by calling the ```store()``` method from the CreateOperation;

Since this operation is just a way to allow access to the Create operation from a modal, the ```getInlineCreateModal()``` method will show all the fields you've defined for this operation using the [Fields API](/docs/{{version}}/crud-fields#fields-api), then upon Save the ```store()``` method will first check the validation from the FormRequest you've specified, then create the entry using the Eloquent model. Only attributes that are specified as fields, and are ```$fillable``` on the model will actually be stored in the database.
