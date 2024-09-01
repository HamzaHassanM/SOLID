## What is the Liskov Substitution Principle?

The Liskov Substitution Principle states that if we have a parent class and child classes that inherit from it, we should be able to use the child class wherever the parent class is expected without causing any issues in the code.

### Why is it important?

This principle is important because it ensures that child classes are fully compatible with the parent class. If this principle is not followed, we might end up with child classes that behave in unexpected ways when substituted for the parent class, leading to errors in the system that are difficult to detect and fix.

## A Bad Example

Let's say we are designing a document management system with a parent class `Document` and two child classes: `ReadOnlyDocument` and `EditableDocument`.

The parent class `Document` contains methods to read and modify the content:

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

### The Child Class: Read-Only Document

Now, imagine we have a child class `ReadOnlyDocument`, which is a document that can only be read and should not be modified.

```php
class ReadOnlyDocument extends Document
{
    public function writeContent($content)
    {
        // Cannot modify content in a read-only document
        throw new Exception("Cannot modify read-only document");
    }
}
```

### Where is the problem?

If we apply the Liskov Substitution Principle here, we should be able to replace `Document` with `ReadOnlyDocument` anywhere in the system. However, if the code relies on the ability to modify the document, substituting `Document` with `ReadOnlyDocument` will lead to an unexpected error when attempting to modify the content.

For example:

```php
function processDocument(Document $document)
{
    // Read the content
    echo $document->readContent();

    // Attempt to modify the content
    $document->writeContent("New Content");
}

// Using the ReadOnlyDocument subclass
$doc = new ReadOnlyDocument("Initial Content");
processDocument($doc); // This will throw an exception
```

### The Importance of Following the Principle

In this example, if we don't adhere to the Liskov Substitution Principle, we might encounter unexpected errors when trying to use `ReadOnlyDocument` in places where `Document` is expected. This makes the code unpredictable and difficult to maintain.

To avoid this issue, we can redesign the structure so that `ReadOnlyDocument` is not directly substitutable for `Document` if the system expects modifiable content.

## How to Improve the Design?

Instead of allowing the `ReadOnlyDocument` subclass to inherit directly from `Document`, we can create a `ReadableDocumentInterface` and an `EditableDocumentInterface` to ensure that objects using `ReadOnlyDocument` adhere to a contract that does not include the ability to modify the content.

### The Proposed Interface:

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

### The Base Class:

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

### The Modified Child Class:

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

    // No writeContent method here since it's a read-only interface
}
```

### Now, when using objects:

We can determine whether the object we are dealing with is editable or not based on the interface it implements. This ensures that substituting objects happens correctly without any issues.

## Conclusion

The Liskov Substitution Principle is a fundamental part of software design, ensuring that child classes behave predictably when used as substitutes for parent classes. This principle helps avoid errors that may arise from incorrect object substitution, making the code more stable and maintainable.
