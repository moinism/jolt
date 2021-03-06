Jolt
====

Jolt is a PHP micro framework that helps you quickly write simple yet powerful web applications and APIs.

Jolt takes some inspiration from ExpressJS.

Jolt is not a full featured MVC framework, it is built to be a micro framework that handles routing and carries some basic template rendering. Feel free to use your own template engine such as Twig instead.

You can see more info at the [Jolt home page](http://joltframework.com/):

For database, I recommend [Idiorm and Paris](http://j4mie.github.com/idiormandparis/)

### Requirements
* PHP 5.3

### Quick and Basic
A typical PHP app using Jolt will look like this.

```php
<?php
// include the library
include 'jolt.php';

$app = new Jolt();

// define your routes
$app->get('/greet', function () use ($app){
	// render a view
	$app->render( 'page', array(
		"pageTitle"=>"Greetings",
		"body"=>"Greetings world!"
	));
});
$app->post('/greet', function () use ($app){
	// render a view
	$app->render( 'page', array(
		"pageTitle"=>"Greetings",
		"body"=>"Greetings world!"
	));
});

$app->get('/hello/:name', function ($name){
	$app = Jolt::getInstance();
	// render a view
	$app->render( 'page', array(
		"pageTitle"=>"Hello",
		"body"=>"Hello {$name}!",
	));
});

$app->get('/', function()  use ($app) {
	$app->render('home');
});

$app->listen();

?>
```

### Optional route parameters

You may also have optional route parameters. These are ideal for using one route for a blog archive. To declare optional route parameters, specify your route pattern like this:

```php
<?php
$app->get('/archives(/:yyyy(/:mm(/:dd)))', function($yyyy='',$mm='',$dd='') use ($app) {
	echo $yyyy.' - '.$mm.' - '.$dd;
});

$app->get('/test(/:param1(/:param2(/:param3(/:param4))))', function () use ($app) {
	$args  = func_get_args();
	print_r($args);
});

?>
```

Each subsequent route segment is optional. This route will accept HTTP requests for:

- /archives
- /archives/2013
- /archivs/2013/04
- /archivs/2013/04/05

If an optional route segment is omitted from the HTTP request, the default values in the callback signature are used instead.


### Route conditions

Jolt lets you assign conditions to route parameters. If the specified conditions are not met, the route is not run. For example, if you need a route with a second segment that must be a valid 4-digit year, you could enforce this condition like this:

```php
<?php
$app->get('/archives(/:yyyy(/:mm(/:dd)))', function($yyyy='',$mm='',$dd='') use ($app) {
	echo $yyyy.' - '.$mm.' - '.$dd;
},  array(
		'yyyy' => '(19|20)\d\d'
		,'mm'=>'(0[1-9]|1[0-2])'
		,'dd'=>'(0[1-9]|[1-2][0-9]|3[0-1])'
	)
);

?>
```

This example, will only let the $yyyy variable match a 4 digit year, $mm match a 2 digit month, and $dd match a 2 digit day.


### Route Symbol Filters
This is taken from ExpressJS. Route filters let you map functions against symbols in your routes. These functions then get executed when those symbols are matched.

```php
<?php
// preload blog entry whenever a matching route has :blog_id in it
$app->filter('blog_id', function ($blog_id) use ($app) {
	$blog = Blog::findOne($blog_id);
	// store() lets you store stuff for later use (NOT a cache)
	$app->store('blog', $blog);
});

// here, we have :blog_id in the route, so our preloader gets run
$app->get('/blogs/:blog_id', function ($blog_id) use ($app)  {
	// pick up what we got from the stash
	$blog = $app->store('blog');
	$app->render('single', array('blog' => $blog);
});
?>
```

### Using classes in Routes

You can also make your routes a bit smarter by including classes, this helps move the code into smaller pieces:

```php
<?php
//	instead of a function, we can also define a controller and action and have it called that way as well...
$app->route('/greet2(/:name)', array("controller"=>'Greetings',"action"=>'my_name') );
//	we can also define the class and action as a string.. Class#Action
$app->route('/greet3(/:name)', 'Greetings#my_name' );

class Greetings extends Jolt_Controller{
    public function my_name($name = 'default'){
		$this->app->render( 'page', array(
			"pageTitle"=>"Greetings ".$this->sanitize($name)."!",
			'title'=>'123',
			"body"=>"Greetings ".$this->sanitize($name)."!"
		));

    }
}
?>
```

The classes extend our abstract Jolt_Controller class and already grab the $app variable as a class member, you can then store your code in controller classes seperately from the main index.php file, this can help make larger applications cleaner and neater.

Using a controller class, the function that gets called would still contain the variables that you pass, so in the above example, we passed $name as a variable, and the my_name function inside the Greetings class received it.

### Middleware
Helper function called during routing, handy for taking care of database connections, etc.

```php
<?php

$app->middleware(function () use ($app){
	$db = create_connection();
	$app->store('db', $db);
});
?>
```


### Conditions
Conditions are basically helper functions.

```php
<?php
// require that users are signed in
$app->condition('signed_in', function () use ($app) {
	$app->redirect( '/403-forbidden',!$app->store('user'));
});

// require a valid token when accessing a page
$app->get('/admin', function () use ($app)  {
  $app->condition('signed_in');
  $app->render('admin');
});

?>
```
*NOTE:* Because of the way conditions are defined, conditions can't have anonymous functions as their first parameter.

### Caching

```php
<?php
$data = $app->cache('users', function () {
	return array('bill', 'ted', 'elmo');
}, 60);

$app->cache_invalidate('users');
?>
```


### Configurations
You can make use of ini files for configuration by doing something like `option('source', 'myconfig.ini')`.
This lets you put configuration settings in ini files instead of making `option()` calls in your code.

```php
<?php
// load a config.ini file
$app->option('source', 'my-settings.ini');

// set a different folder for the views
$app->option('views', __DIR__.'/myviews');

// get the encryption secret
$secret = $app->option('secret');
?>
```

### Utility Functions
There are several utility routines in the library:

```php
<?php
//set a route that doesn't care about GET or POST
$app->route('/rule',function() use ($app){
	
});
//set a route that only works during GET queries
$app->get('/rule',function() use ($app){
	
});
//set a route that only works during POST queries
$app->post('/rule',function() use ($app){
	
});
$app->put('/rule',function() use ($app){
	
});
$app->patch('/rule',function() use ($app){
	
});
$app->delete('/rule',function() use ($app){
	
});

//render html with no variables, and use layout.html
$app->render('view');

//render html with variables, stay with standard layout
$app->render('view',array("var1"=>"val1"));

//render html with no variables and a different layout (layout2.html)
$app->render('view',null,"layout2");

//render html with variables and a different layout (layout2.html)
$app->render('view',array("var1"=>"val1"),"layout2");

// store a setting and get it
$app->option('views', './views');
$app->option('views'); // returns './views'

// store a variable and get it (useful for moving stuff between scopes)
$app->store('user', $user);
$app->store('user'); // returns stored $user var

// redirect with a status code
$app->redirect(302, '/index');

// redirect if a condition is met
$app->redirect(403, '/users', !$authenticated);

// redirect only if func is satisfied
$app->redirect('/admin', function () use ($auth) { return !!$auth; });

// redirect only if func is satisfied, and with a diff code
$app->redirect(301, '/admin', function () use ($auth) { return !!$auth; });

// send a http error code and print out a message
$app->error(403, 'Forbidden');

// get the current HTTP method or check the current method
$app->method(); // GET, POST, PUT, DELETE
$app->method('POST'); // true if POST request, false otherwise

// client's IP
$app->client_ip();

// get something or a hash from a hash
$name = $app->from($_POST, 'name');
$user = $app->from($_POST, array('username', 'email', 'password'));

// return URI stuff:
$app->getBaseUri();	//	returns the current root uri.. for example /site/ if the app is installed in the /site/ folder or blank if it's at the document root..

$app->getUri();	//	returns the current uri for example, /login if you are on the login page, or / if you are on the homepage..

// load a partial using some file and locals
$html = $app->partial('users/profile', array('user' => $user));
?>
```