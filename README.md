To run the blog with modifications from git :

```bash
#Make sure sqlite is enabled in PHP (check with php -m, should have sqlite3 and pdo_sqlite)

#cloning the repo
mkdir StarbugStoneDir
cd StarbugStoneDir
git clone https://github.com/Starbugstone/zozor-blog.git
cd zozor-blog

#Installing the dependancies
composer install
#not required for prod
npm install

# create the database
mkdir var\data
php bin/console doctrine:database:create
php bin/console doctrine:schema:create

#Load demo info into database
php bin/console doctrine:fixtures:load

#copy to test unit
cp var\data\blog.sqlite var\data\blog_test.sqlite

#run test
vendor\bin\simple-phpunit.bat

#run the server to test yourself
php bin/console server:run
```

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

Also need to check if the extension=intl is active in the php.ini file for all to work. Also check if pdo_sqlite and sqlite3 extensions are active

Code
------------

**Blog Controller Modifications**
```php
// BlogController

// injecting the objectmanager and userinterface in the constructor

/**
 * @var ObjectManager
 */
private $manager;
/**
 * @var TokenStorageInterface
 */
private $tokenStorage;


public function __construct(ObjectManager $manager, TokenStorageInterface $tokenStorage)
{
    $this->manager = $manager;

    $this->tokenStorage = $tokenStorage;
}

 public function new(Request $request): Response
    {
        // @done: manage the form and the post.
        $post = new Post();
        $form = $this->createForm(PostType::class, $post);

        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $post->setAuthor($this->tokenStorage->getToken()->getUser());
            $this->manager->persist($post);
            $this->manager->flush();
            $this->addFlash('success', 'flash_success_new_post');
            return $this->redirectToRoute('admin_index');
        }

        return $this->render('admin/blog/new.html.twig', [
            'post' => $post,
            'form' => $form->createView(),
        ]);
    }

    /**
     * Finds and displays a Post entity.
     *
     * @Route("/{id<\d+>}", methods={"GET"}, name="admin_post_show")
     */
    public function show(Post $post): Response
    {
        // @done: render the template with the post
        return $this->render('admin/blog/show.html.twig', [
            'post' => $post
        ]);
    }

    /**
     * Displays a form to edit an existing Post entity.
     *
     * @Route("/{id<\d+>}/edit",methods={"GET", "POST"}, name="admin_post_edit")
     * @IsGranted("edit", subject="post", message="Posts can only be edited by their authors.")
     */
    public function edit(Request $request, Post $post): Response
    {
        $form = $this->createForm(PostType::class, $post);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            //$post->setSlug(Slugger::slugify($post->getTitle())); //No longer needed with Gedmo
            // @done: persist the update
            $this->manager->flush();

            $this->addFlash('success', 'post.updated_successfully');

            return $this->redirectToRoute('admin_post_edit', ['id' => $post->getId()]);
        }

        // @done rendrer the post and form
        return $this->render('admin/blog/edit.html.twig', [
            'post' => $post,
            'form' => $form->createView(),
        ]);
    }

    /**
     * Deletes a Post entity.
     *
     * @Route("/{id}/delete", methods={"POST"}, name="admin_post_delete")
     * @IsGranted("delete", subject="post")
     */
    public function delete(Request $request, Post $post): Response
    {
        if (!$this->isCsrfTokenValid('delete', $request->request->get('token'))) {
            return $this->redirectToRoute('admin_post_index');
        }

        // Delete the tags associated with this blog post. This is done automatically
        // by Doctrine, except for SQLite (the database used in this application)
        // because foreign key support is not enabled by default in SQLite
        $post->getTags()->clear();

        // @done: delete the post
        $this->manager->remove($post);
        $this->manager->flush();

        $this->addFlash('success', 'post.deleted_successfully');

        return $this->redirectToRoute('admin_post_index');
    }
```
Also updated Admin/Blog/index.html.twig to include flash messages.


Using 
composer require stof/doctrine-extensions-bundle for the slug generation.
```php
// src/Entity/Post
// Constructing the slug from the title
//comment for the slug variable.
/**
 * @var string
 *
 * @Gedmo\Slug(fields={"title"})
 * @ORM\Column(type="string")
 */
private $slug;
```
