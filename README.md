## Laravel Coding Patterns
Below are some commonly used Laravel coding patterns with code examples to help you understand their implementation:

### 1. **MVC Pattern (Model-View-Controller)**

Laravel follows the **MVC** pattern, where:

- **Model**: Represents the database structure and logic.
- **View**: Displays data to the user.
- **Controller**: Acts as a mediator, handling the flow of data between Model and View.
## Example:

**Model (app/Models/User.php)**
```php
class User extends Model {
    protected $fillable = ['name', 'email', 'password'];
}

```
**Controller (app/Http/Controllers/UserController.php)**
```php
class UserController extends Controller {
    public function index() {
        $users = User::all();
        return view('users.index', compact('users'));
    }
}
```
**View (resources/views/users/index.blade.php)**
```php
@foreach ($users as $user)
    <p>{{ $user->name }} - {{ $user->email }}</p>
@endforeach
```

### 2. **Repository Pattern**

The Repository Pattern provides an abstraction layer for data access, separating business logic from data logic.
## Example:
**UserRepository Interface (app/Repositories/UserRepositoryInterface.php)**
```php
interface UserRepositoryInterface {
    public function all();
    public function find($id);
}
```
**UserRepository Implementation (app/Repositories/UserRepository.php)**
```php
class UserRepository implements UserRepositoryInterface {
    public function all() {
        return User::all();
    }

    public function find($id) {
        return User::find($id);
    }
}
```
**Binding in Service Provider (app/Providers/AppServiceProvider.php)**
```php
public function register() {
    $this->app->bind(UserRepositoryInterface::class, UserRepository::class);
}
```
**Controller using Repository (app/Http/Controllers/UserController.php)**
```php
class UserController extends Controller {
    protected $userRepository;

    public function __construct(UserRepositoryInterface $userRepository) {
        $this->userRepository = $userRepository;
    }

    public function index() {
        $users = $this->userRepository->all();
        return view('users.index', compact('users'));
    }
}
```

### 3. **Service Pattern**

The Service Pattern moves business logic from controllers to service classes, making the code cleaner and reusable.
## Example:

**UserService (app/Services/UserService.php)**
```php
class UserService {
    public function create(array $data) {
        return User::create($data);
    }
}
```
**Controller using Service (app/Http/Controllers/UserController.php)**
```php
class UserController extends Controller {
    protected $userService;

    public function __construct(UserService $userService) {
        $this->userService = $userService;
    }

    public function store(Request $request) {
        $data = $request->validate([
            'name' => 'required',
            'email' => 'required|email',
            'password' => 'required',
        ]);

        $this->userService->create($data);
        return redirect()->route('users.index');
    }
}
```
### 4. **Observer Pattern**

The Observer Pattern is used for monitoring events triggered by models, such as `created`, `updated`, etc.

## Example:
**UserObserver (app/Observers/UserObserver.php)**
```php
class UserObserver {
    public function created(User $user) {
        Mail::to($user->email)->send(new WelcomeMail($user));
    }
}
```
**Register Observer in AppServiceProvider (app/Providers/AppServiceProvider.php)**
```php
public function boot() {
    User::observe(UserObserver::class);
}
```
### 5. **Factory Pattern**

The Factory Pattern in Laravel is used for generating test data or seeding the database.
## Example:

**UserFactory (database/factories/UserFactory.php)**
```php
$factory->define(User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'password' => bcrypt('password'),
    ];
});
```

**Using Factory in Tests (tests/Feature/UserTest.php)**
```php
public function testUserCreation() {
    $user = factory(User::class)->create();
    $this->assertDatabaseHas('users', ['email' => $user->email]);
}
```
