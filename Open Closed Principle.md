# Open/Closed Principle (OCP)

### Open/Closed Principle (OCP)

The Open/Closed Principle is another key concept within the SOLID principles of software design. It focuses on ensuring that software entities (like classes, modules, and functions) are designed in a way that they are open for extension but closed for modification.

This means that the behavior of a module or class can be extended without modifying its existing code. The goal here is to prevent code from becoming fragile or introducing bugs when new functionality is added.

**For example, let's consider a scenario where we have a class that calculates discounts based on different types of customers in an e-commerce application.**

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

## Applying the Principle

- The `DiscountCalculator` class, as shown above, is not adhering to the Open/Closed Principle. If we need to add a new customer type with a different discount, we would need to modify the `calculate` method. This approach can lead to a more complex and error-prone system as the application grows.

- To apply the Open/Closed Principle, we need to refactor the `DiscountCalculator` class to make it open for extension but closed for modification. We can do this by using polymorphism, where each customer type has its own discount logic encapsulated within a dedicated class.

## Step 1

We start by defining an interface or an abstract class that all customer types will implement.

```php
interface Discount
{
    public function calculate(): int;
}
```

## Step 2

Next, we create separate classes for each type of customer, implementing the `Discount` interface. Each class will contain its own logic for calculating the discount.

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

## Step 3

Now, we can modify the `DiscountCalculator` to use these classes without changing its core logic, making it open for extension but closed for modification.

```php
class DiscountCalculator
{
    public function calculate(Discount $discount): int
    {
        return $discount->calculate();
    }
}
```

## Step 4

Finally, when we need to calculate a discount, we simply instantiate the appropriate discount class and pass it to the `DiscountCalculator`. If a new customer type is introduced in the future, we just need to create a new class for it, without modifying any existing code.

```php
$regularCustomerDiscount = new RegularDiscount();
$vipCustomerDiscount = new VipDiscount();
$employeeCustomerDiscount = new EmployeeDiscount();

$calculator = new DiscountCalculator();

echo $calculator->calculate($regularCustomerDiscount); // Outputs: 10
echo $calculator->calculate($vipCustomerDiscount);     // Outputs: 20
echo $calculator->calculate($employeeCustomerDiscount); // Outputs: 30
```

### Benefits of the Open/Closed Principle:

- **Improved maintainability**: By avoiding modifications to existing code, you reduce the risk of introducing bugs and make the system easier to maintain.
  
- **Enhanced flexibility**: New features or behaviors can be added easily by extending the existing codebase rather than modifying it, which promotes flexibility in evolving systems.

- **Encapsulation**: Each class has a clear responsibility, and changes are isolated, which improves the code’s structure and design.

The Open/Closed Principle is crucial in creating systems that can evolve and adapt over time without becoming overly complex or unstable. By keeping modules closed for modification but open for extension, developers can ensure that their systems remain robust and flexible as new requirements emerge.
