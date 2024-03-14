# Laravel Best Practices and Coding Standards For Axcertro (PVT) LTD

## Introduction

This detailed guide aims to ensure consistency and high quality in our Laravel projects by outlining specific coding standards and best practices. Focusing on naming conventions, structure, and the Repository pattern will help us maintain clean, efficient, and scalable code.

## Environment Setup

- **Laravel Homestead**: Recommended for a standardized development environment.
- **Composer**: Ensure it's updated to manage dependencies efficiently.

## Coding Standards

### PSR Compliance

- Adhere to PSR-1 for basic coding standards and PSR-4 for autoloading.

### Naming Conventions

#### Classes & Namespaces
- **Use StudlyCase.**
- Example: `UserProfileRepository`

#### Methods
- **Use camelCase.**
- Example: `public function getUserById($id)`

#### Variables
- **Use camelCase with descriptive names.**
- Example: `$userProfile`

#### Routes
- **Use kebab-case.**
- Example: Route definition for user profiles:
  ```php
  Route::get('user-profile', [UserProfileController::class, 'index']);
  ```

#### Configurations & Language Files
- **Use snake_case.**
- Example: `user_profile.php` for configuration files.

## Directory Structure

- Adhere to Laravel's conventions, placing business logic within appropriately named directories under `app/`.

## Repository Pattern
For speed development, We have a Base repository/Interface file that contains all the basic methods. You must extend all your repositories with the Base Repository files.

### Example Interface (Repository Contract)

- Located in `app/Repositories/Users/UserRepositoryInterface.php`
  ```php
  namespace App\Repositories\Users;

  use App\Repositories\Base\EloquentRepositoryInterface;
  
  interface UserRepositoryInterface extends EloquentRepositoryInterface
  {
      public function all();
      public function find($id);
      public function findByEmail($email);
  }
  ```

### Example Implementation

- Located in `app/Repositories/Users/UserRepository.php`
  ```php
  namespace App\Repositories\Users;
  
  use App\Models\User;
  use App\Repositories\Base\BaseRepository;
  use App\Repositories\Contracts\UserRepositoryInterface;
  
  class UserRepository extends BaseRepository implements UserRepositoryInterface
  {
      protected $model;
      
      public function __construct(User $model)
      {
          $this->model = $model;
      }
      
      public function all()
      {
          return $this->model->all();
      }
      
      public function find($id)
      {
          return $this->model->find($id);
      }
      
      public function findByEmail($email)
      {
          return $this->model->where('email', $email)->first();
      }
  }
  ```

### Service Provider Binding

- Bind interfaces to implementations in a service provider.
- Located in `app/Providers/RepositoryServiceProvider.php`
  ```php
  namespace App\Providers;
  
  use Illuminate\Support\ServiceProvider;
  use App\Repositories\Users\UserRepositoryInterface;
  use App\Repositories\Users\UserRepository;
  
  class RepositoryServiceProvider extends ServiceProvider
  {
      public function register()
      {
          $this->app->bind(UserRepositoryInterface::class, UserRepository::class);
      }
  }
  ```

  ### How to use the repository in a controller
  - use the construct function to import controller-level repositories
  ```php
    namespace App\Http\Controllers\Dashboard;
    
    use App\Http\Controllers\Controller;
    use App\Repositories\All\User\UserInterface;
    use Inertia\Inertia;
    use Inertia\Response;
    
    class DashboardController extends Controller
    {    
        protected $usersRepository;
  
        public function __construct(UserInterface $usersRepository){
  
            $this->usersRepository= $usersRepository;
  
        }
        /**
         * __invoke
         *
         * @return Response
         */
        public function __invoke(): Response{
            return Inertia::render('Dashboard/Index',[
                'users' => $this->usersRepository->all()
            ]); 
        }
    }
  ```
    - use app()->make() function to import method-level repositories
  ```php
    namespace App\Http\Controllers\Dashboard;
    
    use App\Http\Controllers\Controller;
    use App\Repositories\All\User\UserInterface;
    use Inertia\Inertia;
    use Inertia\Response;
    
    class DashboardController extends Controller
    {    
        /**
         * __invoke
         *
         * @return Response
         */
        public function __invoke(): Response{
            $usersRepository = app()->make(UserInterface::class);
            return Inertia::render('Dashboard/Index',[
                'users' => $usersRepository->all()
            ]); 
        }
    }
  ```

## Additional Best Practices

### Database Migrations and Seeders

- **Migrations**: Use meaningful names in snake_case. Example: `2021_01_01_000000_create_users_table.php`
- **Seeders**: Name seeders after the table they populate in StudlyCase. Example: `UsersTableSeeder.php`

### Security Practices

- **Input Sanitization**: Use Laravel's request validation.
  ```php
  $validated = $request->validate([
      'email' => 'required|email',
      'password' => 'required|min:6',
  ]);
  ```
- **SQL Injection Prevention**: Utilize Eloquent ORM or query builder, avoiding raw queries.
- **Data Encryption**: Use Laravel's encrypt/decrypt functions for sensitive data.

### Performance Optimization

- **Eager Loading**: Prevent N+1 query issues by eager loading relationships.
  ```php
  $users = User::with('posts')->get();
  ```

This comprehensive guide, with specific examples and detailed explanations, should serve as a strong foundation for coding standards and best practices within your Laravel projects at Axcertro (Pvt) Ltd. It's designed to be a living document, adaptable and updatable as new practices emerge and as your team's needs evolve.
