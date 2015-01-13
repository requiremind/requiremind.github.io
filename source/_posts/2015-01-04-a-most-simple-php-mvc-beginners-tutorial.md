---
layout: post
title: "A Most Simple PHP MVC Beginners Tutorial"
date: 2015-01-04 21:30
comments: true
author: Neil Rosenstech
categories:
- php
- mvc
---

A few months ago, a friend of mine started taking courses about web development. Like (almost) everyone else he began learning PHP and soon enough he was asked to develop a small handmade MVC application.
So he went on the internet and started searching easily understandable tutorials, only to find out there was not that many good guides out there. Let's fix this.

<!-- more -->

Disclaimer: this tutorial is just my take on the subject. I do not guarantee that it will solve your problem. MVC is a design pattern really easy to grasp when you've been working with it, yet a bit
odd when you come across the first time. The application I will create is probably perfectly imperfect, this was originally a quick afternoon draft from a guy that has been working with Ruby on Rails
the last couple years, it goes without saying that PHP was remotely a memory. Anyways, if you find anything in this article that you think could threaten the future of the universe, please yell at me in the comments below.

Application Github repo: [https://github.com/Raindal/php_mvc](https://github.com/Raindal/php_mvc)

Let's begin with the boring part.

### MVC, what the f***?

Right now I'm guessing you have at least a basic understanding of PHP. You've probably been building small websites or applications already, to get a general idea of the possibilities.

I suppose you've been starting with putting everything in one file and you quickly moved on to using "includes" and "requires" (better even require_once) just to make your code more readable, easier
to follow for you or someone else. You see, that transition you made was for your code to be clearer, and therefore, maintainable. Maybe you even know about such things as Separation of Concerns and
DRY (Do not Repeat Yourself). If you don't know about those I suggest you take a quick look at it cause you'll come across that stuff a lot more than you'd think if you plan on following the white rabbit.

Here are some useful resources on the matter, just get a vague idea, do not get stuck on those (read the introductions at least I guess) :

- [Separation of Concerns](http://en.wikipedia.org/wiki/Separation_of_concerns)
- [DRY](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself)

Well, MVC is a design pattern that suggests you organize your code in a way concerns are properly separated, and yet interact with each other in a certain fashion.

MVC stands for Model View Controller as you may already know. That makes 3 types of concerns, I know this sounds a bit esoteric but I have faith you'll get it by reading on. Just to make things simpler,
just imagine you have an application and in that application you have at least those 3 folders : models, views and controllers. Those folders contain specific files (no shit Sherlock). Now, those files
are classified that way for a good reason: they have to carry on different tasks.

In the end: models fetch and mold data which is in turn sent to the view and therefore diplayed. The controller is managing the process.

### Building an example app

Let's get the basic stuff done: create a database **php_mvc** that contains a table **posts** with 3 columns **id** (INT PRIMARY auto_increment), **author** (VARCHAR 255, note that in a real life 
situation you could also put an index on this one as searching through posts with a sepcific author may happen quite often) and **content** (TEXT).

Let's move on to creating our first file: index.php.  
The first thing it's going to do is making sure we have something to query the database any time later in the execution.  
As this is a pretty specific thing we're gonna put it in a specific file that we're going to require in our index.php.

Note that I'm working in WAMP. My project root is at C:\wamp\www\php_mvc_blog

``` php index.php
<?php
  require_once('connection.php');
?>
```

Let's write the actual file.

``` php connection.php
<?php
  class Db {
    private static $instance = NULL;

    private function __construct() {}

    private function __clone() {}

    public static function getInstance() {
      if (!isset(self::$instance)) {
        $pdo_options[PDO::ATTR_ERRMODE] = PDO::ERRMODE_EXCEPTION;
        self::$instance = new PDO('mysql:host=localhost;dbname=php_mvc', 'root', '', $pdo_options);
      }
      return self::$instance;
    }
  }
?>
```

You may have to change the 12th line to match your database user/password. Here my user is `root` and I do not have a password.

As you may have noticed we're just creating a singleton class. This allows us to return an instance of the connection and always the same as we only need one to execute all our queries. As the class is a singleton, we make `__construct()` and `__clone()` private so that no one can call `new Db()`. It has an `$instance` variable that retains the connection object (PDO here). In the end, we have access to our "connection object" through `Db::getInstance()`. 

Note that singletons are not the only way to handle that, it's just been a common way for years but this may not be the best approach depending on your needs: 
[see this post on StackOverflow for instance](http://stackoverflow.com/questions/4595964/is-there-a-use-case-for-singletons-with-database-access-in-php).

Anyways, we can go on with our index file.

``` php index.php
<?php
  require_once('connection.php');

  if (isset($_GET['controller']) && isset($_GET['action'])) {
    $controller = $_GET['controller'];
    $action     = $_GET['action'];
  } else {
    $controller = 'pages';
    $action     = 'home';
  }

  require_once('views/layout.php');
?>
```

Here is the whole file. 
Note that this index.php file is going to receive all the requests. This means that every link would have to point to /?x=y or /index.php?x=y.

The if statement is gonna check whether we have the parameters controller and action set and store them in variables. If we do not have such parameters we just make **pages** the default controller
and **home** the default action. You may not know where we're going with this but you should get the idea by reading on. Every request when hiting our index file is going to be routed to a
controller (just a file defining a class) and an action in that controller (just a method).

Finally we require the only part of our application that does not (theoretically) change: the layout.

Let's create this file.

``` php views/layout.php
<DOCTYPE html>
<html>
  <head>
  </head>
  <body>
    <header>
      <a href='/php_mvc_blog'>Home</a>
    </header>

    <?php require_once('routes.php'); ?>

    <footer>
      Copyright
    </footer>
  <body>
<html>
```

As you can see, we put it in a new *views/* folder because it is something displayed on the users' screen, in other words it is rendered. It contains only our header with the different links you could 
find in a basic menu and a footer. Note that while I'm working on WAMP for this guide the root of my application is not / but /php_mvc_blog (the name of my folder under www/ in WAMP).

In the middle we require another file: routes.php. The only part we still need is the main area of our page. We can determine what view we need to put there depending on our previously set *$controller*
and *$action* variables. The routes.php file is gonna take care of that.

This file is not in the views folder but at the root of our application.

``` php routes.php
<?php
  function call($controller, $action) {
    // require the file that matches the controller name
    require_once('controllers/' . $controller . '_controller.php');

    // create a new instance of the needed controller
    switch($controller) {
      case 'pages':
        $controller = new PagesController();
      break;
    }

    // call the action
    $controller->{ $action }();
  }

  // just a list of the controllers we have and their actions
  // we consider those "allowed" values
  $controllers = array('pages' => ['home', 'error']);

  // check that the requested controller and action are both allowed
  // if someone tries to access something else he will be redirected to the error action of the pages controller
  if (array_key_exists($controller, $controllers)) {
    if (in_array($action, $controllers[$controller])) {
      call($controller, $action);
    } else {
      call('pages', 'error');
    }
  } else {
    call('pages', 'error');
  }
?>
```

Ok so we want our routes.php to output the html that was requested one way or another. To fetch the right view (file containing the html we need) we have 2 things: a controller name and an action name.  
We can write a function `call` that will take those 2 arguments and call the action of the controller as done in the previous code sample.

We're doing great. Let's continue and write our first controller for the call function to find the file.

``` php controllers/pages_controller.php
<?php
  class PagesController {
    public function home() {
      $first_name = 'Jon';
      $last_name  = 'Snow';
      require_once('views/pages/home.php');
    }

    public function error() {
      require_once('views/pages/error.php');
    }
  }
?>
```

The class is rather straightforward. We have 2 public functions as expected: `home()` and `error()`.  
As you can see, the first one is also defining some variables. We're doing this so we can later ouput their values in the view and not clutter our view with variables definition and other computation
not related to anything visual. Separation of concerns.

To keep things simple, we usually name the view after the action name and we store it under the controller name.

Let's create both our views.

``` php views/pages/home.php
<p>Hello there <?php echo $first_name . ' ' . $last_name; ?>!<p>

<p>You successfully landed on the home page. Congrats!</p>
```

Use those previously defined $first_name and $last_name where you see fit.

``` php views/pages/error.php
<p>Oops, this is the error page.</p>

<p>Looks like something went wrong.</p>
```

Fantastic! We have a working sample of our MVC app. Navigate to *localhost/php_mvc_blog* and you should see the following:

> Home  
Hello there Jon Snow!   
You successfully landed on the home page. Congrats!  
Copyright

### Improve the app with models and some data management

What we want now is to be able to query our database in a clean way and display the results.  
Say we want to be able to fetch a list of posts and display those, and same thing for one particular post.

Let's modify our *routes.php* file to reflect that behaviour.

Here is the final file:

``` php routes.php
<?php
  function call($controller, $action) {
    require_once('controllers/' . $controller . '_controller.php');

    switch($controller) {
      case 'pages':
        $controller = new PagesController();
      break;
      case 'posts':
        // we need the model to query the database later in the controller
        require_once('models/post.php');
        $controller = new PostsController();
      break;
    }

    $controller->{ $action }();
  }

  // we're adding an entry for the new controller and its actions
  $controllers = array('pages' => ['home', 'error'],
                       'posts' => ['index', 'show']);

  if (array_key_exists($controller, $controllers)) {
    if (in_array($action, $controllers[$controller])) {
      call($controller, $action);
    } else {
      call('pages', 'error');
    }
  } else {
    call('pages', 'error');
  }
?>
```

Just pay attention to where I add a comment, those parts where modified.  
We can create the posts controller now and we will finish with the model.

``` php controllers/posts_controller.php
<?php
  class PostsController {
    public function index() {
      // we store all the posts in a variable
      $posts = Post::all();
      require_once('views/posts/index.php');
    }

    public function show() {
      // we expect a url of form ?controller=posts&action=show&id=x
      // without an id we just redirect to the error page as we need the post id to find it in the database
      if (!isset($_GET['id']))
        return call('pages', 'error');

      // we use the given id to get the right post
      $post = Post::find($_GET['id']);
      require_once('views/posts/show.php');
    }
  }
?>
```

As you may have noticed, the pages controller is managing static pages of our application whereas the posts controller is managing what we call a resource.  
The resource here is a post. A resource can usually be listed (index action), showed (show action), created, updated and destroyed.  
We will keep the focus on the first two as building a fully functional app is not the point of this article.

`Post::all()` and `Post::find()` refer to the model Post that has not yet been written. One of the best piece of advice I was ever given was to write the final code as I want it to be. This way
I'm sure I'm writing the most beautiful code I can come up with and then I make sure it works.

So here, when I fetch the posts I want it to look like `$posts = Post::all();` as this is clear and simple.

Now we can move on to writing the model.

``` php models/post.php
<?php
  class Post {
    // we define 3 attributes
    // they are public so that we can access them using $post->author directly
    public $id;
    public $author;
    public $content;

    public function __construct($id, $author, $content) {
      $this->id      = $id;
      $this->author  = $author;
      $this->content = $content;
    }

    public static function all() {
      $list = [];
      $db = Db::getInstance();
      $req = $db->query('SELECT * FROM posts');

      // we create a list of Post objects from the database results
      foreach($req->fetchAll() as $post) {
        $list[] = new Post($post['id'], $post['author'], $post['content']);
      }

      return $list;
    }

    public static function find($id) {
      $db = Db::getInstance();
      // we make sure $id is an integer
      $id = intval($id);
      $req = $db->prepare('SELECT * FROM posts WHERE id = :id');
      // the query was prepared, now we replace :id with our actual $id value
      $req->execute(array('id' => $id));
      $post = $req->fetch();

      return new Post($post['id'], $post['author'], $post['content']);
    }
  }
?>
```

Our model is a class with 3 attributes.  
It has a constructor so we can call `$post = new Post('Neil', 'A Most Simple PHP MVC Beginners Tutorial')` if need be.  
I think this is pretty much straightforward but let me know in the comments below if something needs more explanation.

Let's modify our layout to add a link to the posts in the header.

Here is the final file.

``` php views/layout.php
<DOCTYPE html>
<html>
  <head>
  </head>
  <body>
    <header>
      <a href='/php_mvc_blog'>Home</a>
      <a href='?controller=posts&action=index'>Posts</a>
    </header>

    <?php require_once('routes.php'); ?>

    <footer>
      Copyright
    </footer>
  <body>
<html>
```

Ok now final step we have to create the views we require in the posts controller.

``` php views/posts/index.php
<p>Here is a list of all posts:</p>

<?php foreach($posts as $post) { ?>
  <p>
    <?php echo $post->author; ?>
    <a href='?controller=posts&action=show&id=<?php echo $post->id; ?>'>See content</a>
  </p>
<?php } ?>
```

And

``` php views/posts/show.php
<p>This is the requested post:</p>

<p><?php echo $post->author; ?></p>
<p><?php echo $post->content; ?></p>
```

We need to have some data in the database so add some manually as we do not have a feature for this yet.  
I'm using phpmyadmin for this matter.

I created 2 posts :

- one with author "Jon Snow" and content "The Wall will fall."
- the other with author "Khal Drogo" and content "I may be dead but I'm still the most awesome character." (and you shall not disagree on that one)

Now go ahead and test everything in your browser.

What you've seen here is a way of separating the concerns in an MVC style.  
Note that a lot of stuff is open to interpretation. This way of seeing things is mostly inspired by what I'm used to do with Rails.

If you want to go further down the road you should check out some framework like Symfony for PHP, Django for Python or Rails for Ruby.

That's it folks! Hope it helps. Until next time.
