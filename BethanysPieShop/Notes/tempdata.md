### 🎯 What is TempData?

In ASP.NET Core, **TempData** is a mechanism used to store data for a very short time. Its specific purpose is to pass data from one HTTP request to the **very next** HTTP request.

The most common use case is for **redirects**. Imagine a user submits a form. You process the data in a POST request, save it to the database, and then redirect the user back to a list or a "success" page (a GET request). If you want to show a message like "Successfully created new product\!" on that success page, `TempData` is the perfect tool.

The data in `TempData` is "temporary" because once it's read in the second request, it's automatically marked for deletion.

-----

### ⚙️ How It Works

`TempData` is a dictionary object (specifically, a `TempDataDictionary`) that you can access in your controllers, views, and Razor Pages. Behind the scenes, ASP.NET Core typically uses **Session** state or **cookies** to store the data between the two requests.

  * **Request 1 (e.g., POST):** You set a value in `TempData`.
  * **Redirect:** The server sends a redirect (HTTP 302) to the browser.
  * **Request 2 (e.g., GET):** The browser makes a new request. You can now read the value from `TempData`. After you read it, the value is automatically removed.

-----

### ⌨️ How to Use TempData

You can set and get `TempData` values in your controller actions using a string key, just like a dictionary.

#### Example: The Post-Redirect-Get Pattern

This is the classic use case.

**1. Set the data in the POST action:**
In your controller, after the user creates something, you set a `TempData` message before redirecting.

```csharp
[HttpPost]
public IActionResult Create(Product product)
{
    if (ModelState.IsValid)
    {
        // ... save the product to the database ...
        
        // Set the success message in TempData
        TempData["Message"] = "Your new product was created successfully!";
        
        // Redirect to the Index page
        return RedirectToAction("Index");
    }

    return View(product);
}
```

**2. Read the data in the GET action's view:**
In your `Index.cshtml` view, you can check if the `TempData` key exists and then display the message.

```html
@* At the top of your Index.cshtml file *@

@if (TempData["Message"] != null)
{
    <div class="alert alert-success" role="alert">
        @TempData["Message"]
    </div>
}

<h2>Product List</h2>
```

When the `Index` page loads after the redirect, the message will be displayed. If the user then refreshes the `Index` page, the message will be gone because `TempData` was cleared after being read.

-----

### ✨ Other Useful `TempData` Features

#### The `[TempData]` Attribute

For a cleaner, strongly-typed approach, you can use the `[TempData]` attribute on a property in your controller or Razor Page. The framework will automatically populate this property from `TempData` and persist its value.

```csharp
public class HomeController : Controller
{
    // The framework will manage this property's value in TempData
    [TempData]
    public string Message { get; set; }

    [HttpPost]
    public IActionResult Create(Product product)
    {
        // ... save logic ...
        
        // Just set the property. It's now in TempData.
        Message = "Your new product was created successfully!";
        
        return RedirectToAction("Index");
    }
    
    public IActionResult Index()
    {
        // The 'Message' property will be automatically
        // populated from TempData if a value exists.
        // You can then pass it to the view or use it directly.
        
        // No need to even use the property here, 
        // you can access it in the view.
        return View();
    }
}
```

And in your `Index.cshtml` view, you can access it via the `Model` (if you pass it) or directly from `TempData`:

```html
@if (TempData["Message"] != null)
{
    <div class="alert alert-success" role="alert">
        @TempData["Message"]
    </div>
}
```

#### Peeking vs. Keeping

  * **`TempData.Peek(key)`:** This allows you to **read** the value **without** marking it for deletion. It will still be available for the next request.
  * **`TempData.Keep(key)`:** If you've already read a value (e.g., `var msg = TempData["Message"]`) and you decide you need it to persist for *another* request, you can call `TempData.Keep("Message")`.

-----

### 📊 TempData vs. ViewData vs. Session

This is a common point of confusion. Here's the simple difference:

| Storage | Lifespan | Typical Use Case |
| :--- | :--- | :--- |
| **ViewData/ViewBag** | **Current request only.** | Passing data from a **controller to its view**. |
| **TempData** | **Current and next request.** | Passing data between **two separate requests**, usually a redirect (e.g., status messages). |
| **Session** | **Entire user session.** | Storing data that needs to persist for a user's whole visit (e.g., shopping cart, user ID). |

Would you like to see how to use `ViewData` or `Session` next?