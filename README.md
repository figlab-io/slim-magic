# Slim-Magic Blog

A very minimalistic, sample blog application [Slim Framework][SLIM].

The purpose of this application is to test if the [magic][M] Dependency Container is working with [Slim Framework][SLIM] and also to check the design decisions. 
So, instead of completing the blog centric features, I’ve focused on demonstrating Dependency Injection Container usages and design layers.

![Blog Screenshot](/resources/screenshot.jpg)

## Selection of components
- **Framework**: Used Slim Framework for a few reasons:
    - Simplicity. For a minimalistic approach, a microframework like Slim is more suitable. 
    - Slim uses PSR-11: Container interface compatible Dependency Injection Container and allows using any third party container that implements PSR-11. 
    - Not opinioned. We can design application layers ourselves.
- **Data Accessors**: [Doctrine DBAL][DD] - The Doctrine DataBase Abstraction Layer (DBAL) offers an object-oriented API and a lot of additional.
- **Dependency Injection Container**: [Magic][M] The DI we are testing in this project
- **View Rendering**: Using Twig or any other view library for this simple app seems overkill. So, I have written a simple class `App\Service\Template` to render plain PHP templates with placeholder variables.
- **WYSIWYG Editor**: [SimpleMDE][MDE] - a simple, embeddable, and beautiful JS markdown editor.
- **CSS Framework**: [Skeleton][SK] - this dead simple framework only styles a handful of standard HTML elements and includes a grid.

## Design Overview

### Directory Structure and Logical layers

Slim does not expose any specific directory structure. So it’s easy to organize the files that represent the system's logical layers.

| Directory      | Purpose                                                                                                                                                |
|----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| public         | Contains the FrontController (inde.php) and static assets                                                                                              |
| config         | All kind of configurations - from Service definitions to Routs                                                                                         |
| src/Action     | The Controller classes that handle and respond to http Requests. They use task performers under src/Domain. Do not use the Data Access layer directly. |
| src/Exception  | Purpose specific custom Exception classes.                                                                                                             |
| src/Repository | The Data Access Layer - interacts with data sources.                                                                                                   |
| src/Domain     | The Domain Logic remains here. Uses Data Access Layer when required.                                                                                   |
| src/Services   | Arbitrary functionalities and Utility classes. Can be used across the layers. (e,g, from Actions, Domain etc.)                                         |
| src/Traits     | Common functionalities that can be used by selective Actions.                                                                                          |
| templates      | Plain php template/view files. The templates are Rendered using App\Service\Template Service.                                                          |
| vendor         | Contains dependencies. Auto generated by composer.                                                                                                     |

### The Request Lifecycle

1. All HTTP Requests are received by `public/index.php`. It loads the bootstrapped application from config/bootstrap.php and runs the application.
2. The `config/bootstrap.php` loads and prepares the application. Primarily it performs -
   - Create and configure the Dependency Container using `config/container.php`
   - Loads the defined routes from `config/routes.php`
   - Registers required middlewares using `config/middleware.php`
3. Slim matches the appropriate route based on URL and invokes configured Action.
4. The required Services and domain classes are injected to Action class via constructor with proper type hinting and naming.
5. The DC instantiates the Action class by supplying asked dependencies. Then calls the  `__invoke()` method.
6. The  `__invoke()` method handles the Request and prepares the Response by rendering view file(s). It used the injected Services where needed.
7. The request may be passed through middlewares before and after generating the Response. Middlewares may add/modify the request outcome. For example - generating and validating CSRF protection data.

## How to test

### Testing with Docker

#### 1. Prerequisites
- You should have `composer` installed. [Download composer](https://getcomposer.org/download/) if required.
- You should have `docker` and `docker-compose` installed. [Check instructions](https://docs.docker.com/compose/install/) otherwise. 

#### 2. Run the application
Execute the following 3 commands from project root directory sequentially:
```shell
git clone https://github.com/ajaxray/slim-magic.git blog
cd blog
cp .env.dist .env
# Set DB connection options in the .env file at this point
docker-compose up --build -d
docker-compose exec app composer install --prefer-dist --no-interaction
```

#### 3. Prepare Database
Go to <http://localhost:8081>, Adminer (a tiny alternative to phpMyAdmin) should be running here.  
Login with host: db and user-password  mentioned in `.env` file.
Then select the database “blog”. Import (at left-top corner of page) the database dump `resources/db_dump.sql` file. 
It should create the posts table with sample posts.

#### 4. Go live!
Go to <http://localhost:8080>, You should see the blog home page with a list of recent posts.  
Login (user: admin, password: 123123) to add new posts and edit posts.

#### 5. Exit plan
Run this command to stop docker containers
```shell
docker-compose down
```

### Notes:

1. If you are testing manually,
    - Use the sql dump at `resources/db_dump.sql` to create the posts table.
    - Set database connection by updating “dsn” in `config/container.php`. 
    - Install dependencies with composer
    ```shell
    composer install --prefer-dist
    ```
2. The default user-pass for logging in is admin/123123. You can change it in “auth” values in `config/container.php`.

## Compromises considering “minimalistic”
- Not implemented User management. A fixed login credential (in config/settings.php) is being used for login instead.
- Not implemented commenting, Category and Tagging features.




[M]: https://github.com/ajaxray/magic
[DD]: https://www.doctrine-project.org/projects/doctrine-dbal/en/latest/index.html
[SK]: http://getskeleton.com
[MDE]: https://simplemde.com
[SLIM]: https://www.slimframework.com
[PSR11]: http://www.php-fig.org/psr/psr-11/
