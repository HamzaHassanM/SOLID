# مبدأ المفتوح/المغلق (Open/Closed Principle)

### مبدأ Open/Closed

مبدأ Open/Closed هو أحد مبادئ SOLID في تصميم البرمجيات. ينص هذا المبدأ على أن الوحدات البرمجية (مثل الكلاسات والدوال) يجب أن تكون **مفتوحة للتوسيع** ولكن **مغلقة للتعديل**.

هذا يعني أنه يمكن إضافة وظائف جديدة للوحدات البرمجية بدون الحاجة لتعديل الكود الأصلي. الهدف من هذا المبدأ هو تقليل تأثير التغييرات على النظام، مما يحمي الكود من إدخال الأخطاء ويجعل النظام أكثر مرونة واستقرارًا.

**على سبيل المثال، لنفترض أن لدينا كلاس لحساب الخصومات لأنواع مختلفة من العملاء في تطبيق تجارة إلكترونية.**

```php
class DiscountCalculator
{
    public function calculate($customer)
    {
        if ($customer->type == 'regular') {
            return 10;
        } elseif ($customer->type == 'vip') {
            return 20;
        } elseif ($customer->type == 'employee') {
            return 30;
        }
    }
}
```

## تطبيق المبدأ

- الكلاس `DiscountCalculator` في المثال أعلاه لا يتبع مبدأ Open/Closed. إذا أردنا إضافة نوع جديد من العملاء بخصم مختلف، سيتعين علينا تعديل دالة `calculate`. هذا يجعل الكود أقل مرونة وأكثر عرضة للأخطاء.

- لتطبيق مبدأ Open/Closed، يمكننا إعادة تصميم الكود بحيث نتمكن من إضافة أنواع جديدة من الخصومات دون تعديل الكود الموجود. يمكننا القيام بذلك عن طريق استخدام التعددية الشكلية (Polymorphism)، بحيث يكون لكل نوع من العملاء كلاس خاص به يحتوي على منطق الخصم الخاص به.

## الخطوة الأولى

نبدأ بتعريف واجهة أو كلاس تجريدي يقوم كل نوع من العملاء بتنفيذه.

```php
interface Discount
{
    public function calculate(): int;
}
```

## الخطوة الثانية

ننشئ كلاسات منفصلة لكل نوع من العملاء، حيث تقوم هذه الكلاسات بتنفيذ واجهة `Discount`. يحتوي كل كلاس على منطق حساب الخصم الخاص به.

```php
class RegularDiscount implements Discount
{
    public function calculate(): int
    {
        return 10;
    }
}

class VipDiscount implements Discount
{
    public function calculate(): int
    {
        return 20;
    }
}

class EmployeeDiscount implements Discount
{
    public function calculate(): int
    {
        return 30;
    }
}
```

## الخطوة الثالثة

نعدل الكلاس `DiscountCalculator` ليستخدم هذه الكلاسات بدون تغيير المنطق الأساسي، مما يجعله مفتوحًا للتوسيع ولكنه مغلقًا للتعديل.

```php
class DiscountCalculator
{
    public function calculate(Discount $discount): int
    {
        return $discount->calculate();
    }
}
```

## الخطوة الرابعة

عندما نحتاج إلى حساب الخصم، نقوم ببساطة بإنشاء كائن من الكلاس المناسب وتمريره إلى `DiscountCalculator`. إذا احتجنا لإضافة نوع جديد من العملاء في المستقبل، يمكننا إنشاء كلاس جديد بدون تعديل أي كود موجود.

```php
$regularCustomerDiscount = new RegularDiscount();
$vipCustomerDiscount = new VipDiscount();
$employeeCustomerDiscount = new EmployeeDiscount();

$calculator = new DiscountCalculator();

echo $calculator->calculate($regularCustomerDiscount); // الناتج: 10
echo $calculator->calculate($vipCustomerDiscount);     // الناتج: 20
echo $calculator->calculate($employeeCustomerDiscount); // الناتج: 30
```

### فوائد مبدأ Open/Closed:

- **تحسين الصيانة**: بتجنب تعديل الكود الموجود، يقلل من خطر إدخال الأخطاء ويجعل النظام أسهل في الصيانة.

- **زيادة المرونة**: يمكن إضافة ميزات أو سلوكيات جديدة بسهولة عن طريق توسيع الكود بدلاً من تعديله، مما يعزز مرونة النظام.

- **التغليف**: كل كلاس له مسؤولية واضحة، ويتم عزل التغييرات، مما يحسن تصميم الكود وبنيته.

مبدأ Open/Closed هو أداة مهمة في تطوير أنظمة مرنة وقابلة للتطوير، حيث يمكن للنظام التكيف مع التغييرات المستقبلية دون أن يصبح معقدًا أو غير مستقر.
