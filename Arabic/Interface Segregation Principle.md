## ما هو مبدأ Interface Segregation؟

مبدأ Interface Segregation ينص على أنه يجب عدم إجبار الكائنات على تنفيذ واجهات لا تستخدمها. بمعنى آخر، يجب تقسيم الواجهات الكبيرة والشاملة إلى واجهات أصغر وأكثر تخصصًا بحيث يحتاج كل كائن إلى تنفيذ ما يحتاجه فقط. هذا المبدأ يساعد على تجنب "تلوث" الكود ويجعل النظام أكثر مرونة وقابلية للصيانة.

### لماذا هو مهم؟

هذا المبدأ مهم لأنه يساعد في الحفاظ على تصميم نظيف ومرن. عندما تكون الواجهات أصغر وأبسط، يمكن للكائنات أن تركز على مسؤولياتها المحددة فقط دون الحاجة إلى تحمل وظائف غير ذات صلة. هذا يقلل من التعقيد ويجعل الكود أكثر وضوحًا وأسهل في الفهم والصيانة.

## مثال سيء

لنفترض أننا نصمم نظامًا للصور يتطلب من بعض الكائنات تنفيذ واجهة `ImageProcessor` التي تحتوي على عدة وظائف:

```php
interface ImageProcessor
{
    public function resize($width, $height);
    public function applyFilter($filter);
    public function save($path);
    public function print();
}
```

الآن، لدينا فئة `PhotoPrinter` التي تحتاج فقط إلى وظيفة `print()` من هذه الواجهة:

```php
class PhotoPrinter implements ImageProcessor
{
    public function resize($width, $height)
    {
        // لا حاجة لهذه الوظيفة هنا
    }

    public function applyFilter($filter)
    {
        // لا حاجة لهذه الوظيفة هنا
    }

    public function save($path)
    {
        // لا حاجة لهذه الوظيفة هنا
    }

    public function print()
    {
        // تنفيذ وظيفة الطباعة
        echo "Printing photo...";
    }
}
```

### أين تكمن المشكلة؟

الفئة `PhotoPrinter` تضطر إلى تنفيذ عدة وظائف لا تحتاجها، وهذا يتعارض مع مبدأ Interface Segregation. إذا قررت لاحقًا تغيير واجهة `ImageProcessor` بإضافة وظائف جديدة، سيتعين علينا تحديث `PhotoPrinter`، حتى لو لم تكن هذه التغييرات ذات صلة بها.

## كيف يمكن تحسين التصميم؟

بدلاً من استخدام واجهة كبيرة تحتوي على وظائف متعددة، يمكننا تقسيمها إلى واجهات أصغر وأكثر تخصصًا:

### الواجهات المقترحة:

```php
interface Resizable
{
    public function resize($width, $height);
}

interface Filterable
{
    public function applyFilter($filter);
}

interface Savable
{
    public function save($path);
}

interface Printable
{
    public function print();
}
```

### الفئة المعدلة:

```php
class PhotoPrinter implements Printable
{
    public function print()
    {
        // تنفيذ وظيفة الطباعة
        echo "Printing photo...";
    }
}
```

### الآن، عند استخدام الكائنات:

يمكننا التأكد من أن كل كائن ينفذ فقط الواجهة التي يحتاجها دون أن يتأثر بالتغييرات التي لا تهمه. هذا يجعل التصميم أكثر نظافة ويقلل من التعقيد غير الضروري.

## خلاصة

مبدأ Interface Segregation يساعد في تقسيم الواجهات الكبيرة إلى واجهات أصغر وأكثر تخصصًا، مما يسمح للكائنات بالتركيز على وظائفها الأساسية فقط. الالتزام بهذا المبدأ يجعل الكود أكثر وضوحًا وأسهل في الصيانة، ويمنع تراكم المسؤوليات غير الضرورية في الكائنات.
