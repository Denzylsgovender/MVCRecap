# MVCRecap
This repository contains a simple ASP.NET MVC application for managing student information. This guide will walk you through setting up the application and implementing CRUD (Create, Read, Update, Delete) operations.
# StudentMVCApp Setup

## Step 1: Set Up the Project

### Create a New ASP.NET Core MVC Project

1. Open Visual Studio and create a new ASP.NET Core Web Application.
2. Select **MVC** as the template.
3. Name your project `StudentMVCApp`.

### Install Necessary Packages

1. Open the NuGet Package Manager and install the following packages:
   - `Microsoft.EntityFrameworkCore.Sqlite` (for SQLite database)
   - `Microsoft.EntityFrameworkCore.Tools` (for migrations)

### Enable Entity Framework in `Program.cs`

In the `Program.cs` file, add the following code to set up the SQLite database:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddDbContext<StudentDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddControllersWithViews();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Student}/{action=Index}/{id?}");

app.Run();
```

### Add the Connection String in appsettings.json

Add the following connection string for SQLite in appsettings.json:
```csharp
"ConnectionStrings": {
    "DefaultConnection": "Data Source=students.db"
}
```
### Create the Database Context and Model
Create a Student Model
In the Models folder, create a new class Student.cs with the following code:
```csharp
namespace StudentMVCApp.Models
{
    public class Student
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public int Age { get; set; }
    }
}

```
### Create a StudentDbContext
In the Data folder, create a new class StudentDbContext.cs:

```csharp
using Microsoft.EntityFrameworkCore;
using StudentMVCApp.Models;

namespace StudentMVCApp.Data
{
    public class StudentDbContext : DbContext
    {
        public StudentDbContext(DbContextOptions<StudentDbContext> options)
            : base(options)
        {
        }

        public DbSet<Student> Students { get; set; }
    }
}

```

