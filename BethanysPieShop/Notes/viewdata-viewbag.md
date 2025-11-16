In ASP.NET, both `ViewData` and `ViewBag` are used to pass small amounts of temporary data from a controller to a view. The most important thing to know is that **`ViewBag` is simply a dynamic wrapper around `ViewData`**. They both point to the same underlying data collection.

This means that setting a value using one method makes it available through the other:

```csharp
// In your Controller
ViewData["Title"] = "My Page Title";

// This will also work
string title = ViewBag.Title; // title is "My Page Title"

// And vice-versa
ViewBag.Message = "Hello World";

// This will also work
string message = (string)ViewData["Message"]; // message is "Hello World"
```

Here’s a breakdown of their differences and when you might use them.

-----

### `ViewData`

`ViewData` is a **dictionary** object (`ViewDataDictionary`). You access its data using string keys.

  * **Syntax (Controller):** `ViewData["KeyName"] = "My Value";`
  * **Syntax (View):** `@ViewData["KeyName"]`
  * **Typing:** It's weakly typed. When you retrieve a value in the view, you must **cast** it to its original type to use its specific properties or methods.
  * **Null Checks:** You must check for nulls before casting to avoid errors.

**Example:**

```csharp
// Controller
ViewData["PageTitle"] = "User Dashboard";
ViewData["ItemsCount"] = 10;
```

```html
<h2>@ViewData["PageTitle"]</h2>
<p>You have @((int)ViewData["ItemsCount"]) items in your cart.</p>
```

> **Note:** You must cast `ViewData["ItemsCount"]` to `(int)` to treat it as a number.

-----

### `ViewBag`

`ViewBag` is a **dynamic** property. It uses the `dynamic` feature of C\# to allow you to add properties to it at runtime.

  * **Syntax (Controller):** `ViewBag.PropertyName = "My Value";`
  * **Syntax (View):** `@ViewBag.PropertyName`
  * **Typing:** It's dynamic. You do **not** need to cast the value in the view.
  * **Null Checks:** The syntax can be cleaner, especially with the null-conditional operator (`?.`).

**Example:**

```csharp
// Controller
ViewBag.PageTitle = "User Dashboard";
ViewBag.ItemsCount = 10;
```

```html
<h2>@ViewBag.PageTitle</h2>
<p>You have @ViewBag.ItemsCount items in your cart.</p>
```

> **Note:** No casting is needed for `ViewBag.ItemsCount`.

-----

### 📊 Quick Comparison

| Feature | `ViewData` (Dictionary) | `ViewBag` (Dynamic) |
| :--- | :--- | :--- |
| **How it works** | A dictionary of objects. | A dynamic wrapper around `ViewData`. |
| **Access** | `ViewData["MyKey"]` | `ViewBag.MyKey` |
| **Type Casting** | **Required** in the view. | **Not required**. |
| **Performance** | Slightly faster (negligible). | Slightly slower (due to dynamic resolution). |
| **Key Names** | Allows spaces and non-standard C\# names (e.g., `ViewData["My-Key"]`). | Must be valid C\# identifiers (e.g., `ViewBag.MyKey`). |
| **Error Checking** | Runtime errors if a key doesn't exist or cast fails. | No compile-time checking. Runtime errors if a property is misspelled. |

-----

### 🚀 Recommendation: When to Use What

1.  **For most data, use a ViewModel.**
    This is the modern, recommended "best practice." A ViewModel is a strongly-typed class you create specifically for your view. It provides **compile-time type checking**, which prevents common runtime errors (like typos) and gives you **IntelliSense** (autocomplete) in your view.

    > **When to use:** For 95% of cases. Any data that is central to the view (a user, a list of products, form data) should be in a ViewModel.

2.  **For trivial data, use `ViewBag` or `ViewData`.**
    If you only need to pass a single, simple piece of data that isn't part of your main model (like a page title or a temporary success message), either `ViewBag` or `ViewData` is fine.

      * **Choose `ViewBag`** if you prefer the cleaner syntax and don't want to cast.
      * **Choose `ViewData`** if you need to use a key with a space (e.g., for compatibility with a library) or if you're on a team that prefers the dictionary syntax.

The choice between `ViewBag` and `ViewData` is mostly a matter of personal or team preference. The most important rule is to prefer **ViewModels** for everything else.

-----

This video helps explain the differences between `ViewData` and `ViewBag` in ASP.NET MVC.

  * [ViewData vs ViewBag in ASP.NET MVC](https://www.youtube.com/watch?v=B1VIMNIeLGQ)

http://googleusercontent.com/youtube_content/0