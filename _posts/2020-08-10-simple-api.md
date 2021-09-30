---
title: "Build an ASP.NET Core API in a Few Steps"
description: "We are going to rapid prototype an API."
tags: [aspnet core, API, REST]
categories: [Blogging, Tutorial]
excerpt_image: https://user-images.githubusercontent.com/19977/69273780-dfb9a000-0b8d-11ea-98e3-81a99d12f49f.jpg
---

We are going to create a simple REST API using a compiled language. In this example we will use asp.net core to create an API Server. We will cover the post, get, put and delete actions.

There are a lot of options to create an API. Python, JavaScript, Java and many others. In this example we will drink the asp.net kool aid. This ecosystem has great project generation and in the midwest it is the dominant choice for many companies.

![Image of a vanilla flower](https://user-images.githubusercontent.com/19977/69273780-dfb9a000-0b8d-11ea-98e3-81a99d12f49f.jpg "Vanilla flower")

In this post, I'll walk through building "Store API" style functionality. 

For those following along at home, all the [code is in a GitHub repository](https://github.com/cmmsolutions/SimpleStoreAPI/commits/master). Each commit in the repo corresponds to a step in this post.

## Step 1 - Generate the project
This assumes you have the .NET Core SDK or Visual Studio Community installed. If you have Visual Studio Community do the following:
Go to File > New > Project…
Select the Visual C# project category and then select ASP.NET Web Application (.NET Framework)
Name your project AspNetWebApiRest and click OK
Select the Empty project template and click OK (don’t check any boxes to add core references)


```bash
dotnet new webapi -o StoreApi
cd StoreApi
dotnet add package Microsoft.EntityFrameworkCore.InMemory
code -r ../StoreApi
```

This generates a simple ASP.NET API website. We are using an InMemory database for speedy development. This would not be a proper set up for a live site, it is not complicated to add one later.

## Step 2 - [Build out the Domain Model](https://github.com/cmmsolutions/SimpleStoreAPI/commit/b477fef3a04e8098a7df8a7da56a987b8c754162#diff-43f4297e75ad154a88b66734ee091ce8)

In this step, we build a few simple classes.

__/Models/Products.cs__

```csharp
public class Products
{
   public int Id { get; set; }
   public string Sku { get; set; }
   public string Name { get; set; }
   public string Description { get; set; }
   public decimal Price { get; set; }
   public bool IsAvailable { get; set; }

   public int CategoryId { get; set; }

   /// <summary>
   /// This is to stop a circular reference
   /// </summary>
   [JsonIgnore]
   public virtual Category Category { get; set; }
}
```

__/Models/User.cs__

```csharp
public class User
{
    public int Id { get; set; }
    public string Email { get; set; }
    public virtual List<Order> Orders { get; set; }
}
```

__/Models/Category.cs__

```csharp
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual List<Product> Products { get; set; }
}
```

__/Models/Order.cs__

```csharp
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public int UserId { get; set; }

    /// <summary>
    /// This is to stop a circular reference
    /// </summary>
    [JsonIgnore]
    public virtual User User { get; set; }
    public virtual List<Product> Products { get; set; }
}
```

## Step 3 - [Build out Entity Framework Core](https://github.com/cmmsolutions/SimpleStoreAPI/commit/b477fef3a04e8098a7df8a7da56a987b8c754162#diff-43f4297e75ad154a88b66734ee091ce8)

We will install two nuget packages to get the ball rolling. 

```bash
Install-Package Microsoft.EntityFrameworkCore.InMemory -Version 3.1.7

And

Install-Package Microsoft.EntityFrameworkCore -Version 3.1.7
```

In this section we will first create the ShopContext class. Here we will add entity framework core into the project and seed the database.

__/Models/ShopContext.cs__

```csharp
public class ShopContext : DbContext
{
    public ShopContext(DbContextOptions<ShopContext> options) : base(options)
    { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Category>().HasMany(c => c.Products).WithOne(a => a.Category).HasForeignKey(a => a.CategoryId);
        modelBuilder.Entity<Order>().HasMany(o => o.Products);
        modelBuilder.Entity<Order>().HasOne(o => o.User);
        modelBuilder.Entity<User>().HasMany(u => u.Orders).WithOne(o => o.User).HasForeignKey(o => o.UserId);

        // modelBuilder.Seed();
    }

        public DbSet<Product> Products { get; set; }
        public DbSet<Category> Categories { get; set; }
        public DbSet<Order> Orders { get; set; }
        public DbSet<User> Users { get; set; }
}
```

Let us explore what is going on in this class. We will use the documentation to configure the [DbContext](https://docs.microsoft.com/en-us/ef/core/miscellaneous/configuring-dbcontext). After that we had to tell Entity Framework the relation between those model classes. We are using the [Fluent API](https://www.entityframeworktutorial.net/efcore/fluent-api-in-entity-framework-core.aspx)  in Entity Framework Core for our mappings [mappings](https://www.entityframeworktutorial.net/efcore/fluent-api-in-entity-framework-core.aspx). You can dive deeper into how to use them here but going into detail about that concept is outside the scope of this writing. 

We will also add an extension method to seed the database with values 

[__/Models/ModelBuilderExtensions.cs__](https://github.com/cmmsolutions/SimpleStoreAPI/blob/master/SimpleStoreAPI/Models/ModelBuilderExtensions.cs)

This file is too long to give a preview for so just copy it using the link above. 

With all of that the project is now set up for us to start working on some crud operations. So let's get to it.

## Step 4 - [Build out the GET](https://github.com/cmmsolutions/SimpleStoreAPI/commit/754f50bfd664d0ef72139d83227e8cd1bc1f5aa0)

Here we are going to create the GetAllProducts and GetProduct(int id) methods. As the name implies the get all products will all of the products. The get product will use the product id to pull back that specific product. 

```csharp
public class ProductsController : ControllerBase
{
    private readonly ShopContext _context;

    public ProductsController(ShopContext context)
    {
        _context = context;

        _context.Database.EnsureCreated();
    }

    [HttpGet]
    public IActionResult GetAllProducts()
    {
        return Ok(_context.Products.ToArray());
    }

    [HttpGet("{id}")]
    public IActionResult GetProduct(int id)
    {
        var product = _context.Products.Find(id);
        return Ok(product);
    }
}
```

We have already created our Context class. The context object is how we interact with the runtime database. Using the Find and ToArray call from the context object are the backbone of our methods.  

That’s it for the GET calls!!!

## Step 5 - [Build out the POST](https://github.com/cmmsolutions/SimpleStoreAPI/commit/8c73ed2135bb2a26b00040d1d6d74c0b7dabf858)

Here we will need some extra software to implement this. I use [Insomnia](https://dev.to/kmcknight91/how-to-use-insomnia-to-test-api-endpoints-1lad) but there is nothing wrong with anything else. We will use the following json to test our new code.

```json
{
	"name": "Old Skater Jeans",
	"sku": "AWMGSYYJ",
	"price": 68.00,
	"isAavailable": true,
	"categoryId": 1
}
```

Using the api client of your choice we will put that json in the body of your PUT call. We will add the following method to now receive PUTs. 

```csharp
[HttpPost]
public ActionResult<Product> PostProduct([FromBody] Product product)
{
    _context.Products.Add(product);
    _context.SaveChanges();
        return CreatedAtAction (
            "GetProduct",
            new { id = product.Id },
            product
        );
}
```
With this in our controller we can accept a call and we can test this and see the update in the system. 
In this method we are using [EF core](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-3.1&tabs=visual-studio#the-puttodoitem-method) and [CreatedAtAction](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdataction?view=aspnetcore-3.1) call. The EF is just an add for the moment and the CreatedAtAction returns a created (201) response with a location header. In another post we will add to this but for now this is a simple example.

## Step 6 - [Build out the PUT](https://github.com/cmmsolutions/SimpleStoreAPI/commit/23dd01f5577db6abf8ff30b00f7bc187af6e27c7)

In this section we will work on the update portion of the api. We will use similar json but with some changes. 

```json
{
	"name": "Super Old Skater Jeans",
	"sku": "AWMGSYYZZ",
	"price": 168.00,
	"isAavailable": true,
	"categoryId": 1
}
```

There are just a few updates to the name, sku and price. We will have to use the correct path to trigger a put, https://localhost:{your port number}/products/{the id of the product you want to update}. Now just so we are on the same page there should not be curly braces in your actual url. That's just for us.

We will add the new put method now. 

```csharp
[HttpPut("{id}")]
public IActionResult PutProduct([FromRoute] int id, [FromBody] Product product)
{
    if (id != product.Id)
        {
            return BadRequest();
        }
    _context.Entry(product).State = EntityState.Modified;
    
    try
        {
            _context.SaveChanges();
        }
    
    catch (DbUpdateConcurrencyException)
        {
            if (_context.Products.Find(id) == null)
            {
                return NotFound();
            }
                throw;
        }
            return NoContent();
}
```

Here we are working with many of the same objects, nothing new or fancy. We are using [BadRequest](https://docs.microsoft.com/en-us/dotnet/api/system.web.http.apicontroller.badrequest?view=aspnetcore-2.2), [NotFound](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.notfound?view=aspnetcore-3.1), and [NoContent](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.nocontent?view=aspnetcore-3.1). Using [EntityState](https://docs.microsoft.com/en-us/ef/ef6/saving/change-tracking/entity-state) we have a lot of options but it is too much to go over in this small section. From the documentation it is "Modified: the entity is being tracked by the context and exists in the database, and some or all of its property values have been modified". That should be enough to keep us going. 


## Step 7 - [Build out the DELETE](https://github.com/cmmsolutions/SimpleStoreAPI/commit/d7d74a76d3ac6f2aba0f090eef1ac971684bc8cc)

Ok we are at the home stretch. The delete will act the same way as our put method. Except in our tool we will select delete instead of put to see some action. 

```csharp
[HttpDelete("{id}")]
public async Task<ActionResult<Product>> DeleteProduct(int id)
{
    var product = await _context.Products.FindAsync(id);
    if (product == null)
    {
        return NotFound();
    }

    _context.Products.Remove(product);
    await _context.SaveChangesAsync();
    return product;
}
```

This is it for our basic api!! We will add some more changes to this project in a later post. Some of the things we will cover will be adding a full database to this, paging, filtering, searching and sorting and more. Thank you for reading!!!!

And we're done. You can see all the changes in order by [looking at the commits](https://github.com/cmmsolutions/SimpleStoreAPI/commits/master) in order.

Or just see the final result at https://github.com/cmmsolutions/SimpleStoreAPI.