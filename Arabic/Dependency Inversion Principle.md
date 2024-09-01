## ما هو مبدأ Dependency Inversion؟

مبدأ Dependency Inversion (DIP) ينص على أنه يجب ألا تعتمد المكونات عالية المستوى على المكونات منخفضة المستوى. بدلاً من ذلك، يجب أن تعتمد كلاهما على التجريدات (abstractions). علاوة على ذلك، يجب ألا تعتمد التجريدات على التفاصيل، بل يجب أن تعتمد التفاصيل على التجريدات. يساعد هذا المبدأ في تقليل الترابط بين أجزاء النظام المختلفة ويعزز من مرونة وصيانة الكود.

### لماذا هو مهم؟

هذا المبدأ مهم لأنه يضمن أن المكونات عالية المستوى (التي تحتوي على المنطق الأساسي للتطبيق) ليست مرتبطة بشكل مباشر بالمكونات منخفضة المستوى (التي تتعامل مع تنفيذات محددة). من خلال الاعتماد على التجريدات بدلاً من التنفيذات الملموسة، يمكنك بسهولة تبديل أو تعديل المكونات منخفضة المستوى دون التأثير على المنطق العالي المستوى. وهذا يؤدي إلى كود أكثر modular وقابلية للتكيف.

## مثال سيء

لنقل أنك تمتلك فئة `OrderService` تعتمد على تنفيذ محدد لفئة `PaymentProcessor`:

```php
class PaymentProcessor
{
    public function processPayment($amount)
    {
        // منطق معالجة الدفع
    }
}

class OrderService
{
    protected $paymentProcessor;

    public function __construct(PaymentProcessor $paymentProcessor)
    {
        $this->paymentProcessor = $paymentProcessor;
    }

    public function placeOrder($amount)
    {
        // منطق وضع الطلب
        $this->paymentProcessor->processPayment($amount);
    }
}
```

### أين تكمن المشكلة؟

في هذا المثال، تعتمد `OrderService` مباشرة على فئة `PaymentProcessor`. إذا كنت بحاجة إلى تغيير منطق معالجة الدفع أو التبديل إلى معالج دفع مختلف، ستحتاج إلى تعديل فئة `OrderService`. هذا يخلق ترابطاً قوياً بين `OrderService` و`PaymentProcessor`.

## كيف يمكن تحسين التصميم؟

لتطبيق مبدأ Dependency Inversion، يجب عليك استخدام واجهات أو فئات تجريدية لفصل المكونات عالية المستوى عن المكونات منخفضة المستوى:

### الواجهات المقترحة:

```php
interface PaymentProcessorInterface
{
    public function processPayment($amount);
}
```

### تنفيذ المكونات منخفضة المستوى:

```php
class PaymentProcessor implements PaymentProcessorInterface
{
    public function processPayment($amount)
    {
        // منطق معالجة الدفع
    }
}
```

### المكون عالي المستوى:

```php
class OrderService
{
    protected $paymentProcessor;

    public function __construct(PaymentProcessorInterface $paymentProcessor)
    {
        $this->paymentProcessor = $paymentProcessor;
    }

    public function placeOrder($amount)
    {
        // منطق وضع الطلب
        $this->paymentProcessor->processPayment($amount);
    }
}
```

### الآن، عند استخدام الكائنات:

يمكنك بسهولة استبدال `PaymentProcessor` بتنفيذ مختلف ينفذ أيضاً `PaymentProcessorInterface` دون الحاجة لتعديل `OrderService`. على سبيل المثال:

```php
class AnotherPaymentProcessor implements PaymentProcessorInterface
{
    public function processPayment($amount)
    {
        // منطق معالجة دفع مختلف
    }
}

// الاستخدام
$paymentProcessor = new PaymentProcessor(); // أو AnotherPaymentProcessor
$orderService = new OrderService($paymentProcessor);
$orderService->placeOrder(100);
```

## خلاصة

مبدأ Dependency Inversion يساعد في فصل المكونات عالية المستوى عن المكونات منخفضة المستوى من خلال الاعتماد على التجريدات بدلاً من التنفيذات الملموسة. هذا يجعل الكود أكثر مرونة وأسهل في الصيانة، حيث يمكنك تعديل أو استبدال المكونات منخفضة المستوى دون التأثير على المنطق العالي المستوى.
