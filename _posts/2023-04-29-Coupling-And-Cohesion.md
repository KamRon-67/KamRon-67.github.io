---
title: "Coupling and Cohesion"
description: "A simple coupling and cohesion walkthrough"
tags: [Coupling, Cohesion, C#,]
categories: [Blogging, Coding Examples]
---

Software design principles like coupling and cohesion play a significant role in determining the quality and maintainability of software systems. When high coupling and low cohesion exist, the software system can become challenging to maintain and modify. On the other hand, low coupling and high cohesion can make a system more adaptable, easier to comprehend, and maintain.

Coupling measures how tightly connected or dependent two or more software modules or components are. High coupling between components can create rigidity in a system, where modifications in one component necessitate changes in others. Conversely, low coupling means the system components are independent, and changes to one component are less likely to impact others. To make a software system more flexible and adaptable, it is generally advisable to minimize coupling between components.

Cohesion, on the other hand, measures how well the various parts of a module work together towards a common goal. High cohesion implies that all module parts serve the same purpose and have a well-defined function. Conversely, low cohesion can result in modules that perform multiple, unrelated tasks, making the code challenging to understand and maintain. It is therefore essential to ensure that each module or component in a software system has a well-defined purpose to maximize cohesion.

Example: 

```csharp
public class Customer
{
    private string name;
    private string email;
    private string address;
    private string phone;

    public Customer(string name, string email, string address, string phone)
    {
        this.name = name;
        this.email = email;
        this.address = address;
        this.phone = phone;
    }

    public string GetName()
    {
        return name;
    }

    public string GetEmail()
    {
        return email;
    }

    public string GetAddress()
    {
        return address;
    }

    public string GetPhone()
    {
        return phone;
    }
}
```

This class has low coupling because it is self-contained and does not depend on any other classes. It has high cohesion because all its parts are related to representing and managing a customer's information.
On the other hand, consider a class that represents a shopping cart in the same application. 

```csharp
public class ShoppingCart
{
    private Customer customer;
    private List<Product> products;
    private decimal total;

    public ShoppingCart(Customer customer)
    {
        this.customer = customer;
        this.products = new List<Product>();
        this.total = 0;
    }

    public void AddProduct(Product product)
    {
        products.Add(product);
        total += product.GetPrice();
    }

    public void RemoveProduct(Product product)
    {
        products.Remove(product);
        total -= product.GetPrice();
    }

    public decimal GetTotal()
    {
        return total;
    }
}
```

In the C# example given, the Customer class demonstrates low coupling and high cohesion. It is self-contained, does not rely on other classes, and all its parts are related to representing and managing customer information. On the other hand, the ShoppingCart class displays high coupling and high cohesion, as it depends on both the Customer class and the Product class, and all its parts are related to representing and managing a shopping cart.

Designing software systems with low coupling and high cohesion is generally regarded as a best practice. Adhering to these design principles can make the system more flexible, easier to understand and maintain, and less prone to errors. Low coupling and high cohesion help developers create software that is easier to change, adapt, and extend, enabling them to add new features and functionality without disrupting existing code.

