# Single Responsibility Principle

<div dir="rtl">

### مبدأ المسؤولية الواحدة (Single Responsibility Principle - SRP)

هو أحد المبادئ الخمسة لتصميم البرمجيات المعروفة بمبادئ
SOLID

يهدف هذا المبدأ إلى ضمان أن تكون كل وحدة
(Class, Method , Module)
في البرمجيات مسؤولة عن القيام بمهمة واحدة فقط بشكل محدد.



يعني ذلك أن كل وحدة يجب أن تؤدي وظيفة واحدة فقط أو تكون مسؤولة عن جانب واحد من النظام. هذا يساعد في تحقيق عدة أهداف : 

</div>

<div dir="rtl">

 -  تقليل التعقيد: عندما تكون الوحدة مسؤولة عن مهمة واحدة فقط، يصبح فهم الكود أسهل. كلما كانت الوحدة تقوم بعدة مهام، زاد تعقيدها وأصبح من الصعب تتبع السبب وراء أي خطأ أو مشكلة في الكود.

-  تحسين قابلية الصيانة: عندما تكون الوحدة مسؤولة عن مهمة واحدة فقط، يمكنك تعديل أو تحسين هذه المهمة دون الخوف من تأثير ذلك على أجزاء أخرى من الكود.

-  إعادة الاستخدام: الوحدات التي تقوم بمهمة واحدة تكون أكثر قابلية لإعادة الاستخدام في أجزاء أخرى من التطبيق أو حتى في تطبيقات أخرى.


**لنأخذ مثلا عن حالة إنشاء مستخدم جديد في Laravel حيث سنقوم بإنشاءمستخدم جديد ثم نرسل بريد إلكتروني ترحيبي له وبعد ذلك نسجل عملية الإنشاء في الأحداث (اللوجز) وبعد ذلك نقم بإعادة البيانات إلى المستخدم**

</div>


```php
class UserController extends Controller
{
    public function register(Request $request)
    {
        // التحقق من صحة البيانات
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|min:6|confirmed',
        ]);

        // إنشاء المستخدم
        $user = User::create([
            'name' => $request->input('name'),
            'email' => $request->input('email'),
            'password' => bcrypt($request->input('password')),
        ]);

        // إرسال بريد إلكتروني ترحيبي
        Mail::to($user->email)->send(new WelcomeEmail($user));

        // تسجيل الحدث
        Log::info('User registered: ' . $user->email);

        // إرجاع الاستجابة
        return response()->json(['message' => 'User registered successfully']);
    }
}

```


<div dir="rtl">

## تطبيق المبدأ 

</div>

<div dir="rtl">

- في البداية يجب فصل كل هذه الأمور المتشابكة عن بعضها فليست من مهام الكنترولر أن يتحقق وينشىء ويرسل بريد الكترون ويسجل حدث أو لوج

- لذلك سنبدأ بإنشاء `RegisterRequest` من أجل ان يتحقق من صحة البيانات  وبهذا نكون قد فصلنا التحقق في كلاس خاص به 
- بعد ذلك سنقوم `NotificationService` والذي سيكون مهمته إرسال الإشعارات وبهذا نكون قد فصلنا مسؤولية الاشعارات في مكان مخصص لها 
- بعد ذلك سنقوم بإنشاء `LoggingService` الخاص بتسجيل الاحداث وبالتالي نكون قد فصلنا هذه المسؤلية أيضا 
- خطوة اختيارية وهي إنشاء `UserService` من أجل وضع الكود الخاص بعملية التسجيل بالكامل فيه بدلا من وضعه بالكنترولر وهذا أيضا يطبق المبدأ بشكل أفضل وأكثر صرامة 
- في النهاية نقوم بتعديل `UserController` ليكون مهمته فقط هي النداء على السيرفس الذي يقوم بعملية التسجيل بالكامل


</div> 
<div dir="rtl">

## الخطوة الأولى 
</div> 

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
<div dir="rtl">

## الخطوة الثانية 

</div>

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

<div dir="rtl">

## الخطوة الثالثة

</div>

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

<div dir="rtl">

## الخطوة الرابعة

</div>

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
        // إنشاء المستخدم
        $user = User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => bcrypt($data['password']),
        ]);

        // إرسال البريد الإلكتروني الترحيبي
        $this->notificationService->sendWelcomeEmail($user);

        // تسجيل الحدث
        $this->loggingService->logUserRegistration($user);

        return $user;
    }
}

```

<div dir="rtl">

## الخطوة الخامسة

</div>

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