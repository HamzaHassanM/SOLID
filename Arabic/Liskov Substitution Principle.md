
## ما هو مبدأ Liskov Substitution؟

مبدأ Liskov Substitution ينص على أنه إذا كانت لدينا فئة أساسية (`Parent` class) وفئات فرعية (`Child` classes) ترث منها، فيجب أن يكون بإمكاننا استخدام الفئة الفرعية في أي مكان نتوقع فيه استخدام الفئة الأساسية دون أن يتسبب ذلك في أي مشاكل في الكود.

### لماذا هو مهم؟

المبدأ مهم لأنه يضمن أن الفئات الفرعية متوافقة تمامًا مع الفئة الأساسية. إذا لم يتم الالتزام بهذا المبدأ، فقد ينتهي بنا الأمر بفئات فرعية تتصرف بطرق غير متوقعة عندما نستبدلها بالفئة الأساسية، مما يؤدي إلى أخطاء في النظام يصعب اكتشافها وإصلاحها.

## مثال سيء

لنفترض أننا نصمم نظامًا لإدارة المستندات يحتوي على فئة أساسية `Document`، وفئتين فرعيتين `ReadOnlyDocument` و`EditableDocument`. 

الفئة الأساسية `Document` تحتوي على طرق (methods) لقراءة وتعديل المحتوى:

```php
class Document
{
    protected $content;

    public function __construct($content)
    {
        $this->content = $content;
    }

    public function readContent()
    {
        return $this->content;
    }

    public function writeContent($content)
    {
        $this->content = $content;
    }
}
```

### الفئة الفرعية: مستند للقراءة فقط

الآن، تخيل أن لدينا فئة فرعية `ReadOnlyDocument`، وهي مستند يمكن قراءته فقط ولا يجب تعديله.

```php
class ReadOnlyDocument extends Document
{
    public function writeContent($content)
    {
        // لا يمكن تعديل المحتوى في المستند للقراءة فقط
        throw new Exception("Cannot modify read-only document");
    }
}
```

### أين تكمن المشكلة؟

إذا استخدمنا مبدأ Liskov Substitution هنا، يجب أن نكون قادرين على استبدال `Document` بـ `ReadOnlyDocument` في أي مكان في النظام. ولكن إذا كان الكود يعتمد على إمكانية تعديل المستند، فإن استبدال `Document` بـ `ReadOnlyDocument` سيؤدي إلى خطأ غير متوقع عند محاولة تعديل المحتوى.

على سبيل المثال:

```php
function processDocument(Document $document)
{
    // قراءة المحتوى
    echo $document->readContent();

    // محاولة تعديل المحتوى
    $document->writeContent("New Content");
}

// استخدام الفئة الفرعية ReadOnlyDocument
$doc = new ReadOnlyDocument("Initial Content");
processDocument($doc); // هذا سيؤدي إلى استثناء
```

### أهمية الالتزام بالمبدأ

في هذا المثال، إذا لم نلتزم بمبدأ Liskov Substitution، فإننا قد نواجه أخطاء غير متوقعة عندما نحاول استخدام `ReadOnlyDocument` في أماكن نتوقع فيها استخدام `Document`. هذا يجعل الكود غير متوقع ويصعب الحفاظ عليه.

لتجنب هذه المشكلة، يمكننا إعادة تصميم الهيكل بحيث لا يسمح `ReadOnlyDocument` بالاستبدال المباشر بـ `Document` إذا كان النظام يتوقع إمكانية التعديل.

## كيف يمكن تحسين التصميم؟

بدلاً من السماح للفئة الفرعية `ReadOnlyDocument` بأن ترث من `Document` مباشرةً، يمكننا إنشاء واجهة `ReadableDocumentInterface` وأخرى `EditableDocumentInterface` لضمان أن الكائنات التي تستخدم `ReadOnlyDocument` تلتزم بعقد لا يتضمن القدرة على التعديل.

### الواجهة المقترحة:

```php
interface ReadableDocumentInterface
{
    public function readContent();
}

interface EditableDocumentInterface extends ReadableDocumentInterface
{
    public function writeContent($content);
}
```

### الفئة الأساسية:

```php
class Document implements EditableDocumentInterface
{
    protected $content;

    public function __construct($content)
    {
        $this->content = $content;
    }

    public function readContent()
    {
        return $this->content;
    }

    public function writeContent($content)
    {
        $this->content = $content;
    }
}
```

### الفئة الفرعية المعدلة:

```php
class ReadOnlyDocument implements ReadableDocumentInterface
{
    protected $content;

    public function __construct($content)
    {
        $this->content = $content;
    }

    public function readContent()
    {
        return $this->content;
    }

    // لا يوجد هنا writeContent لأنها واجهة قراءة فقط
}
```

### الآن، عند استخدام الكائنات:

يمكننا تحديد ما إذا كان الكائن الذي نتعامل معه قابلاً للتعديل أو لا بناءً على الواجهة التي يستخدمها. وهذا يضمن أن استبدال الكائنات يتم بشكل صحيح دون أي مشاكل.

## خلاصة

مبدأ Liskov Substitution هو جزء أساسي من تصميم البرمجيات، حيث يضمن أن الفئات الفرعية تتصرف بطريقة متوقعة عند استخدامها كبديل للفئة الأساسية. هذا المبدأ يساعد في تجنب الأخطاء التي قد تنشأ من استبدال الكائنات بطريقة غير صحيحة، ويضمن أن الكود أكثر استقرارًا وقابلية للصيانة.
