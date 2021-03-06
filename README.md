# Eloquent-Sluggable

Easy creation of slugs for your Eloquent models in Laravel 4.

## Installation

First, you'll need to add the package to the `require` attribute of your `composer.json` file:

```json
{
    "require": {
        "cviebrock/eloquent-sluggable": "1.0.*"
    },
}
```

Aftwards, run `composer update` from your command line.

Then, add `'Cviebrock\EloquentSluggable\SluggableServiceProvider',` to the list of service providers in `app/config/app.php`
and add `'Sluggable' => 'Cviebrock\EloquentSluggable\Facades\Sluggable'` to the list of class aliases in `app/config/app.php`.

From the command line again, run `php artisan config:publish cviebrock/eloquent-sluggable`.


## Updating your Eloquent Models

Define a public property `$sluggable` with the definitions (see [#Configuration] below for details):

```php
class Post extends Eloquent
{

	public $sluggable = array(
		'build_from' => 'title',
		'save_to'    => 'slug',
	);

}
```

That's it ... your model is now "sluggable"!


## Using the Class

Saving a model is easy:

```php
$post = new Post(array(
	'title'    => 'My Awesome Blog Post'
));

$post->save();
```

And so is retrieving the slug:

```php
echo $post->slug;
```



## Configuration

Configuration was designed to be as flexible as possible.  You can set up defaults for all of your Eloquent models, and then override those settings for individual models.

By default, global configuration can be set in the `app/config/packages/cviebrock/eloquent-sluggable/config.php` file.  If a configuration isn't set, then the package defaults from `vendor/cviebrock/eloquent-sluggable/src/config/config.php` are used.  Here is an example configuration, with all the default settings shown:

```php
return array(
	'build_from' => null,
	'save_to'    => 'slug',
	'method'     => null,
	'separator'  => '-',
	'unique'     => true,
	'on_update'  => false,
);
```

`build_from` is the field or array of fields from which to build the slug. Each `$model->field` is contactenated (with space separation) to build the sluggable string.  This can be model attribues (i.e. fields in the database) or custom getters.  So, for example, this works:

```php
class Person extends Eloquent {

	public $sluggable = array(
		'build_from' => 'fullname'
	);

	public function getFullnameAttribute() {
		return $this->firstname . ' ' . $this->lastname;
	}

}
```

If `build_from` is empty, false or null, then the value of `$model->__toString()` is used.

`save_to` is the attribute field in your model where the slug is stored.  By default, this is "slug".  You need to create this column in your table when defining your schema:

```php
Schema::create('posts', function($table)
{
	$table->increments('id');
	$table->string('title');
	$table->string('body');
	$table->string('slug');
	$table->timestamps();
});
```

`method` defines the method used to turn the sluggable string into a slug.  There are three possible options for this configuration:

1. When `method` is null (the default setting), the package uses Laravel's `Str::slug()` method to create the slug.

2. When `method` is a callable, then that function or class method is used.  The function/method should expect two parameters: the string to process, and a separator string.  For example, to duplicate the default behaviour, you could do:

```php
	'method' => array('Illuminate\Support\Str','slug'),
```

3. You can also define `method` as a closure (again, expecting two parameters):

```php
	'method' => function( $string, $separator ) {
		return strtolower( preg_replace('/[^a-z]+/i', $separator, $string) );
	},
```

Any other values for `method` will throw an exception.

`separator` defines the separator used when building a slug, and is passed to the `method` defined above.  The default value is a hyphen.

`unique` is a boolean defining whether slugs should be unique among all models of the given type.  For example, if you have two blog posts and both are called "My Blog Post", then they will both sluggify to "my-blog-post" (when using Sluggable's default settings).  This could be a problem, e.g. if you use the slug in URLs.

By turning `unique` on, then the second Post model will sluggify to "my-blog-post-1".  If there is a third post with the same title, it will sluggify to "my-blog-post-2" and so on.  Each subsequent model will get an incremental value appended to the end of the slug, ensuring uniqueness.

`on_update` is a boolean.  If it is `false` (the default value), then slugs will not be updated if a model is resaved (e.g. if you change the title of your blog post, the slug will remain the same) or the slug value has already been set.  You can set it to `true` (or manually change the $model->slug value in your own code) if you want to override this behaviour.

(If you want to manually set the slug value using your model's Sluggable settings, you can run `Sluggable::make($model, true)`.  The second arguement forces Sluggable to update the slug field.)


## Bugs and Suggestions

Please use Github for bugs, comments, suggestions.  Pull requests are preferred!


## Copyright and License

Eloquent-Sluggable was written by Colin Viebrock and released under the MIT License. See the LICENSE file for details.

Copyright 2013 Colin Viebrock
