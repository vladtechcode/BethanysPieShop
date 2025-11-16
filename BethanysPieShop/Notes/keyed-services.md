<<<<<<< HEAD
﻿
**Keyed services**, introduced in .NET 8, are a new feature for the built-in Dependency Injection (DI) container. They solve a common problem: **how to register and consume multiple different implementations of the same interface.**

Before keyed services, if you registered two classes for one interface, the DI container would only resolve the *last one registered*. Keyed services allow you to register multiple implementations, each identified by a unique "key" (like a string or an enum). You can then ask the container for the specific implementation you need by providing its key.

-----

### 🤔 The Problem Keyed Services Solve

Imagine you have a notification system that can send messages via email or SMS. You might have one interface and two implementations:

```csharp
// The interface
public interface INotificationService
{
    Task SendAsync(string message);
}

// Implementation 1
public class EmailNotificationService : INotificationService
{
    public Task SendAsync(string message) { /* ... send email ... */ }
}

// Implementation 2
public class SmsNotificationService : INotificationService
{
    public Task SendAsync(string message) { /* ... send SMS ... */ }
}
```

Without keyed services, registering these is problematic:

```csharp
// In Program.cs
builder.Services.AddScoped<INotificationService, EmailNotificationService>();
builder.Services.AddScoped<INotificationService, SmsNotificationService>(); // This one "wins"

// In your class
public class MyService(INotificationService notificationService)
{
    // Uh oh! notificationService will ALWAYS be SmsNotificationService
}
```

The old workarounds were clunky, such as injecting an `IEnumerable<INotificationService>` and filtering it manually, or creating a custom factory class. Keyed services provide a clean, built-in solution.

-----

### 🔧 How to Use Keyed Services

Using keyed services involves two steps: registering with a key and consuming with a key.

#### 1\. Registering Keyed Services

In your `Program.cs` file, you use the `AddKeyed...` methods instead of the standard `Add...` methods. You provide a key (a string, enum, or any object) to identify each implementation.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Registering services with keys
builder.Services.AddKeyedScoped<INotificationService, EmailNotificationService>("email");
builder.Services.AddKeyedScoped<INotificationService, SmsNotificationService>("sms");

// You can even use enums as keys
public enum NotificationType { Email, Sms }
builder.Services.AddKeyedScoped<INotificationService, EmailNotificationService>(NotificationType.Email);
builder.Services.AddKeyedScoped<INotificationService, SmsNotificationService>(NotificationType.Sms);
```

#### 2\. Consuming Keyed Services

You have two primary ways to get a keyed service from the container:

**A) Using the `[FromKeyedServices]` Attribute (Recommended)**

This is the cleanest way, especially in controllers or other services. You apply the attribute to the constructor parameter, specifying which key to inject.

```csharp
public class NotificationController : ControllerBase
{
    private readonly INotificationService _emailService;
    private readonly INotificationService _smsService;

    // The DI container injects the correct service based on the key
    public NotificationController(
        [FromKeyedServices("email")] INotificationService emailService,
        [FromKeyedServices("sms")] INotificationService smsService)
    {
        _emailService = emailService;
        _smsService = smsService;
    }

    [HttpPost("send-email")]
    public async Task<IActionResult> SendEmail(string message)
    {
        await _emailService.SendAsync(message);
        return Ok();
    }
}
```

**B) Manually from `IServiceProvider`**

This method is useful if you need to resolve a service dynamically (e.g., based on user input or configuration).

```csharp
public class DynamicNotificationSender
{
    private readonly IServiceProvider _serviceProvider;

    public DynamicNotificationSender(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public async Task SendNotification(string channel, string message)
    {
        // "channel" could be "email" or "sms"
        
        // GetKeyedService returns null if not found
        // GetRequiredKeyedService throws an exception if not found
        
        var notificationService = _serviceProvider
            .GetRequiredKeyedService<INotificationService>(channel);

        await notificationService.SendAsync(message);
    }
}
```

-----

### 🎯 Common Use Cases

Keyed services are perfect for any scenario where you have a "strategy" pattern:

  * **Payment Gateways:** Injecting `IPaymentGateway` with keys like `"stripe"`, `"paypal"`, or `"authorize.net"`.
  * **Notification Channels:** As in the example (email, SMS, push).
  * **Multi-Tenancy:** A tenant ID could be the key to resolve tenant-specific services (like `ITenantDatabase`).
  * **File Storage:** Resolving an `IFileStorage` service with keys like `"local"`, `"azure"`, or `"s3"`.
  * **Named `HttpClient`:** This is a common one. You can register different `HttpClient` instances with different base addresses or configurations, each with its own key.

This video provides a great visual explanation of the problem and how keyed services solve it.
[Keyed Services in ASP.NET Core](https://www.youtube.com/watch?v=DdzhQS2-9js)
http://googleusercontent.com/youtube_content/0
=======
﻿# Using Keyed Services

```c#
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddKeyedScoped<IPieRepository, MockPieRepository>("pieRepo");
var app = builder.Build();
```
>>>>>>> 1705f8023058cc0dc8188e6575bcfbc5f56565ab
