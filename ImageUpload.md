# ASP.NET Core File Upload Demo
## Code Overview

### Model: UserModal.cs
```csharp
namespace Praticse.Models
{
    public class UserModal
    {
        public IFormFile profileImg { get; set; }
        public string name;
    }
}

```


### Controller: HomeController.cs

```csharp
using System.Diagnostics;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Praticse.Helpers;
using Praticse.Models;

namespace Praticse.Controllers;

public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }

    public IActionResult Save(UserModal user)
    {
        string filePath = "";
        try
        {
            filePath = ImageHelper.SaveImage(user.profileImg, "Profile");
        }
        catch (System.Exception)
        {
            Console.WriteLine("file not provided mostly");
        }
        Console.WriteLine($"file path :- {filePath}");
        Console.WriteLine($"name :- {user.name}");

        return View("Index");
    }
}
```

### ImageHelper.cs

```csharp
namespace Praticse.Helpers;

using System;
using System.IO;
public class ImageHelper
{
    public static string SaveImage(IFormFile imageFile, string dir)
    {
        string finalDirPath = $"wwwroot/{dir}";
        if (imageFile == null || imageFile.Length == 0)
        {
            throw new Exception();
        }

        if (!Directory.Exists(finalDirPath))
        {
            Directory.CreateDirectory(finalDirPath);
        }
        //extract extension from file
        string fileExtension = Path.GetExtension(imageFile.FileName);

        //genrate unique file name
        string uniqueNameForFile = $"{Guid.NewGuid()}.{fileExtension}";

        //get full path which we will store in db ( we dont need to store from wwwroot)
        string fullPathToStoreInDB = $"{dir}/{uniqueNameForFile}";

        //get path where we store image means wwwroot
        string fullPathToWrite = $"{finalDirPath}/{uniqueNameForFile}";


        // use stream to manipulate or save image in disk
        FileStream stream = new FileStream(fullPathToWrite, FileMode.CreateNew);
        imageFile.CopyTo(stream);
        stream.Close();

        //retrurn path which we will store in db
        return fullPathToStoreInDB;
    }
}
```

### View: Index.cshtml

- Upload form with `multipart/form-data`.

```csharp
<div class="text-center">
    <h1 class="display-4">Welcome</h1>
    <p>Learn about <a href="https://learn.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>

    <form enctype="multipart/form-data" method="post" asp-action="Save" asp-controller="Home">
        <input name="name" />
        <input type="file" name="profileImg" />
        <input type="submit" />
    </form>
</div>
```
