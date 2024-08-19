# Single Responsibility Principle

### Single Responsibility Principle (SRP)

The Single Responsibility Principle is one of the five SOLID principles of software design.

This principle aims to ensure that each unit (Class, Method, Module) in software is responsible for only one specific task.

This means that each unit should perform only one function or be responsible for one aspect of the system. This helps achieve several goals:

- **Reducing complexity**: When a unit is responsible for only one task, the code becomes easier to understand. The more tasks a unit handles, the more complex it becomes, making it harder to track down the cause of any errors or issues in the code.

- **Improving maintainability**: When a unit is responsible for only one task, you can modify or improve that task without worrying about its impact on other parts of the code.

- **Reusability**: Units that perform a single task are more likely to be reusable in other parts of the application or even in other applications.

**For example, let’s consider creating a new user in Laravel, where we will create a new user, send them a welcome email, log the creation process, and then return the data to the user.**

```php
class UserController extends Controller
{
    public function register(Request $request)
    {
        // Validate the request data
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|min:6|confirmed',
        ]);

        // Create the user
        $user = User::create([
            'name' => $request->input('name'),
            'email' => $request->input('email'),
            'password' => bcrypt($request->input('password')),
        ]);

        // Send a welcome email
        Mail::to($user->email)->send(new WelcomeEmail($user));

        // Log the event
        Log::info('User registered: ' . $user->email);

        // Return the response
        return response()->json(['message' => 'User registered successfully']);
    }
}
```

## Applying the Principle

- First, each of these intertwined tasks should be separated. It is not the responsibility of the controller to validate, create, send an email, and log an event.

- So, we start by creating a `RegisterRequest` to validate the data, thus separating the validation into its own class.
- Next, we create a `NotificationService`, which will handle sending notifications, thereby separating the responsibility for notifications.
- After that, we create a `LoggingService` to handle event logging, thus isolating this responsibility as well.
- An optional step is to create a `UserService` to encapsulate the entire registration process, rather than placing it in the controller, which further adheres to the principle more strictly.
- Finally, we modify the `UserController` so that its only task is to call the service that handles the registration process.

## Step 1

```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class RegisterRequest extends FormRequest
{
    public function rules()
    {
        return [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|min:6|confirmed',
        ];
    }
}
```

## Step 2

```php
namespace App\Services;

use App\Models\User;
use App\Mail\WelcomeEmail;
use Illuminate\Support\Facades\Mail;

class NotificationService
{
    public function sendWelcomeEmail(User $user)
    {
        Mail::to($user->email)->send(new WelcomeEmail($user));
    }
}
```

## Step 3

```php
namespace App\Services;

use App\Models\User;
use Illuminate\Support\Facades\Log;

class LoggingService
{
    public function logUserRegistration(User $user)
    {
        Log::info('User registered: ' . $user->email);
    }
}
```

## Step 4

```php
namespace App\Services;

use App\Models\User;
use App\Services\NotificationService;
use App\Services\LoggingService;

class UserService
{
    protected $notificationService;
    protected $loggingService;

    public function __construct(NotificationService $notificationService, LoggingService $loggingService)
    {
        $this->notificationService = $notificationService;
        $this->loggingService = $loggingService;
    }

    public function registerUser(array $data)
    {
        // Create the user
        $user = User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => bcrypt($data['password']),
        ]);

        // Send the welcome email
        $this->notificationService->sendWelcomeEmail($user);

        // Log the event
        $this->loggingService->logUserRegistration($user);

        return $user;
    }
}
```

## Step 5

```php
namespace App\Http\Controllers;

use App\Http\Requests\RegisterRequest;
use App\Services\UserService;

class UserController extends Controller
{
    protected $userService;

    public function __construct(UserService $userService)
    {
        $this->userService = $userService;
    }

    public function register(RegisterRequest $request)
    {
        $user = $this->userService->registerUser($request->validated());

        return response()->json(['message' => 'User registered successfully', 'user' => $user]);
    }
}
```
```