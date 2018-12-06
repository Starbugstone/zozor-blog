Zozor's Blog
============

The Zozor's Blog is an application used a learning support for the Symfony 4 course available in OpenClassRooms platform.
The entire application is based on the []Symfony Demo Application](https://github.com/symfony/demo), which is a reference application created to show how
to develop Symfony applications following the recommended best practices.

Requirements
------------

  * PHP 7.1.3 or higher;
  * PDO-SQLite PHP extension enabled;
  * and the [usual Symfony application requirements][1].

Installation
------------

Execute this command to install the project:

```bash
$ composer create-project mickaelandrieu/learn-symfony
```

Usage
-----

There's no need to configure anything to run the application. Just execute this
command to run the built-in web server and access the application in your
browser at <http://localhost:8000>:

```bash
$ cd learn-symfony/
$ php bin/console server:run
```

Alternatively, you can [configure a fully-featured web server][2] like Nginx
or Apache to run the application.

Tests
-----

Execute this command to run tests:

```bash
$ cd symfony-demo/
$ ./vendor/bin/simple-phpunit
```

[1]: https://symfony.com/doc/current/reference/requirements.html
[2]: https://symfony.com/doc/current/cookbook/configuration/web_server_configuration.html

My Modifications
------------

The create project doesn't work, componants have security issues and composer won't update nicely.

First 
```bash
$ composer update symfony/flex --no-plugins
```

then
```bash
$ composer update
```

Also need to take care of node and npm files. Somthing seams to be corrups with the install so delete, 
reinstall and fix security issues

```bash
$ rm -rf node_modules\
$ npm install
$ npm audit fix
```

Also need to check if the extension=intl is active in the php.ini file for all to work.

Code
------------
**Showing Post**
```php
Public Function Show(post $post){
    return $this->render('admin/blog/show.html.twig', ['post' => $post]);
}
```

**New Post**
```php
// BlogController
public function new(Request $request){
    $post = new Post();
        $form = $this->createFormBuilder($post)
            ->getForm();
}
```

```php
// src/form/PostType
// Adding the save button
$builder->add('save', SubmitType::class, [
    'label' => 'label.save',
    'attr' => array('class' => 'save')
])
```

```xml
// translations/messages.fr.xlf
<trans-unit id="label.save">
    <source>label.save</source>
    <target>Sauvegarder</target>
</trans-unit>
```

