## What is the Interface Segregation Principle?

The Interface Segregation Principle (ISP) states that objects should not be forced to implement interfaces they do not use. In other words, large and general interfaces should be broken down into smaller, more specific ones so that each object only needs to implement the methods it actually requires. This principle helps avoid "polluting" the code and makes the system more flexible and maintainable.

### Why is it Important?

This principle is important because it helps maintain a clean and flexible design. When interfaces are smaller and simpler, objects can focus solely on their specific responsibilities without being burdened by unrelated functions. This reduces complexity and makes the code more understandable and easier to maintain.

## Bad Example

Let's consider a system for handling images where some objects are required to implement an `ImageProcessor` interface with several functions:

```php
interface ImageProcessor
{
    public function resize($width, $height);
    public function applyFilter($filter);
    public function save($path);
    public function print();
}
```

Now, suppose we have a `PhotoPrinter` class that only needs the `print()` function from this interface:

```php
class PhotoPrinter implements ImageProcessor
{
    public function resize($width, $height)
    {
        // This function is not needed here
    }

    public function applyFilter($filter)
    {
        // This function is not needed here
    }

    public function save($path)
    {
        // This function is not needed here
    }

    public function print()
    {
        // Implement print functionality
        echo "Printing photo...";
    }
}
```

### What's the Problem?

The `PhotoPrinter` class is forced to implement several functions it doesn't need, which violates the Interface Segregation Principle. If you later decide to modify the `ImageProcessor` interface by adding new functions, you'll have to update `PhotoPrinter` even if these changes are irrelevant to it.

## How to Improve the Design?

Instead of using a large interface with multiple functions, you can break it down into smaller, more specialized interfaces:

### Suggested Interfaces:

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

### Modified Class:

```php
class PhotoPrinter implements Printable
{
    public function print()
    {
        // Implement print functionality
        echo "Printing photo...";
    }
}
```

### Now, when using objects:

You can ensure that each object implements only the interface it needs, without being affected by changes that do not concern it. This makes the design cleaner and reduces unnecessary complexity.

## Summary

The Interface Segregation Principle helps to break down large interfaces into smaller, more specialized ones, allowing objects to focus solely on their core functionalities. Adhering to this principle makes the code clearer and easier to maintain, and prevents unnecessary responsibilities from accumulating in objects.

---
