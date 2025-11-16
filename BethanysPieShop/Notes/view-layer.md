Here is a summary article explaining the concepts of the ASP.NET View layer, designed to be archived as part of your learning notes.

-----

## 📄 Article: Understanding the ASP.NET View Layer

In ASP.NET, the **View** is the component responsible for rendering the user interface (UI), typically as HTML. The goal is to keep your UI code separate from your business logic (which lives in the Controller and services). To make this efficient and clean, ASP.NET uses a set of special files and folders based on "convention over configuration."

This guide covers how to pass data to views, the purpose of the layout files, and how the folder structure organizes everything.

### 1\. Passing Data to Views (ViewData vs. ViewBag)

You often need to send small, temporary pieces of data from your Controller to your View (like a page title or a success message). `ViewData` and `ViewBag` are two mechanisms for this.

  * **`ViewData`**: A dictionary object (`ViewDataDictionary`).

      * **Syntax (Controller):** `ViewData["MyKey"] = "My Value";`
      * **Syntax (View):** `@ViewData["MyKey"]`
      * **Key Detail:** It's weakly typed. You must **cast** the data in the view to use it (e.g., `@((int)ViewData["ItemsCount"])`).

  * **`ViewBag`**: A dynamic object.

      * **Syntax (Controller):** `ViewBag.MyKey = "My Value";`
      * **Syntax (View):** `@ViewBag.MyKey`
      * **Key Detail:** It's dynamic, so **no casting** is required in the view.

> **Key Concept:** `ViewBag` is just a dynamic wrapper around `ViewData`. They both point to the *same* underlying data. Setting `ViewData["Title"] = "Home"` makes `@ViewBag.Title` also equal "Home".

**Best Practice:** For all primary view data (like a user's profile, a list of products, etc.), **do not use `ViewData` or `ViewBag`**. Instead, use a strongly-typed **ViewModel**. A ViewModel is a C\# class you create to perfectly shape the data your view needs.

-----

### 2\. View Folder Structure & Conventions

ASP.NET automatically finds views based on a folder structure that matches your controllers.

  * **`Views/ControllerName/`**: This folder holds views for a *specific* controller.

      * **Example:** When `HomeController`'s `Index` action returns `View()`, ASP.NET looks for `/Views/Home/Index.cshtml`.

  * **`Views/Shared/`**: This is the **fallback folder**.

      * **Example:** If ASP.NET *cannot* find `/Views/Home/Index.cshtml`, it will then look for `/Views/Shared/Index.cshtml`.
      * **Purpose:** This makes it the perfect place for any view component that is *shared* across the entire application, such as your layouts (`_Layout.cshtml`) and partial views (`_LoginPartial.cshtml`).

-----

### 3\. The Special "Underscore" Files for DRY

To avoid repeating code (like your navigation bar or `@using` statements) on every page, ASP.NET uses three special files.

#### `_Layout.cshtml` (The Master Template)

This file defines the master HTML template for your site. It's the "picture frame" that all your other views will be rendered inside of.

  * **Purpose:** Contains your `<html>`, `<head>`, `<body>`, main navigation, and footer.
  * **Key Method: `@RenderBody()`**
      * This is a **required placeholder**. It marks the exact spot where the content from your individual views (like `Index.cshtml`) will be injected.
  * **Optional Method: `@RenderSection("Scripts", required: false)`**
      * Creates an optional "section" that a view can "fill." This is commonly used to let individual pages add their own `<script>` tags at the bottom of the `<body>`.
  * **Location:** Almost always placed in `Views/Shared/`.

#### `_ViewStart.cshtml` (The Default Setter)

This file contains C\# code that is **executed before** any view in its folder (and subfolders) is rendered.

  * **Purpose:** Its primary job is to set the default `Layout` for all views in a directory, so you don't have to type `@{ Layout = "_Layout"; }` in every single view file.
  * **Example Content:**
    ```csharp
    @{
        Layout = "_Layout";
    }
    ```
  * **Hierarchy Rule (Override):** You can have more than one. A `_ViewStart.cshtml` in a subfolder (e.g., `Views/Admin/`) will **override** the settings from a `_ViewStart.cshtml` in a parent folder (e.g., `Views/`). This is useful for setting a different layout (like `_AdminLayout`) for an entire section of your site.

#### `_ViewImports.cshtml` (The Shared Toolkit)

This file does not render HTML. Its only job is to provide **global directives** to all views in its folder and subfolders.

  * **Purpose:** To globally import namespaces and register Tag Helpers. This saves you from typing the same `@using` statements at the top of every view.
  * **Example Content:**
    ```csharp
    @using MyWebApp
    @using MyWebApp.ViewModels
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
    ```
  * **Hierarchy Rule (Cumulative):** You can have more than one. Unlike `_ViewStart`, the `_ViewImports` files are **additive** (or cumulative). A view will inherit all directives from the `_ViewImports.cshtml` in its parent *and* its own folder.

-----

### 4\. Summary & Quick Reference

This table summarizes how the files and folders work, including their hierarchy behavior.

| File / Folder | Purpose | How Hierarchy Works (When you have more than one) |
| :--- | :--- | :--- |
| **`Views/ControllerName/`** | Holds views for a specific controller. | N/A |
| **`Views/Shared/`** | **Fallback folder** for shared layouts & partials. | N/A |
| **`_Layout.cshtml`** | **The Template.** Defines the master page HTML. | You can have many. You *select* which one to use (often via `_ViewStart`). |
| **`_ViewStart.cshtml`** | **Sets Defaults.** Runs before views to set `Layout`. | **Overrides.** Child folder's file *overrides* the parent's file. |
| **`_ViewImports.cshtml`** | **Adds Directives.** Provides `@using` and `@addTagHelper`. | **Cumulative.** Child folder's directives are *added to* the parent's. |