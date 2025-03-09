# MVCRecap
This repository contains a simple ASP.NET MVC application for managing student information. This guide will walk you through setting up the application and implementing CRUD (Create, Read, Update, Delete) operations.
# StudentMVCApp Setup

## Set Up the Project

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

### Set Up CRUD Operations
1. Create Controller
In the Controllers folder, create a new controller StudentController.cs and add the following code:

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using StudentMVCApp.Data;
using StudentMVCApp.Models;

namespace StudentMVCApp.Controllers
{
    public class StudentController : Controller
    {
        private readonly StudentDbContext _context;

        public StudentController(StudentDbContext context)
        {
            _context = context;
        }

        // GET: Student
        public IActionResult Index()
        {
            var students = _context.Students.ToList();
            return View(students);
        }

        // GET: Student/Details
        public IActionResult Details(int id)
        {
            var student = _context.Students.FirstOrDefault(s => s.Id == id);
            if (student == null)
            {
                return NotFound();
            }
            return View(student);
        }

        // GET: Student/Create
        public IActionResult Create()
        {
            return View();
        }

        // POST: Student/Create
        [HttpPost]
        public IActionResult Create([Bind("Name, Age")] Student student)
        {
            if (ModelState.IsValid)
            {
                _context.Add(student);
                _context.SaveChanges();
                return RedirectToAction(nameof(Index));
            }
            return View(student);
        }

        // GET: Student/Edit
        public IActionResult Edit(int id)
        {
            var student = _context.Students.FirstOrDefault(s => s.Id == id);
            if (student == null)
            {
                return NotFound();
            }
            return View(student);
        }

        // POST: Student/Edit
        [HttpPost]
        public IActionResult Edit(int id, [Bind("Id, Name, Age")] Student student)
        {
            if (id != student.Id)
            {
                return NotFound();
            }

            if (ModelState.IsValid)
            {
                try
                {
                    _context.Update(student);
                    _context.SaveChanges();
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!_context.Students.Any(e => e.Id == student.Id))
                    {
                        return NotFound();
                    }
                    else
                    {
                        throw;
                    }
                }
                return RedirectToAction(nameof(Index));
            }
            return View(student);
        }

        // GET: Student/Delete
        public IActionResult Delete(int id)
        {
            var student = _context.Students.FirstOrDefault(s => s.Id == id);
            if (student == null)
            {
                return NotFound();
            }
            return View(student);
        }

        // POST: Student/Delete
        [HttpPost, ActionName("Delete")]
        public IActionResult DeleteConfirmed(int id)
        {
            var student = _context.Students.FirstOrDefault(s => s.Id == id);
            if (student == null)
            {
                return NotFound();
            }

            _context.Students.Remove(student);
            _context.SaveChanges();
            return RedirectToAction(nameof(Index));
        }
    }
}
```
### Create Views for Each Action
1. Index View (Display list of students)
In Views/Student/Index.cshtml:
```html
@model IEnumerable<StudentMVCApp.Models.Student>

<h2>Student List</h2>

<table class="table">
    <thead>
        <tr>
            <th>Id</th>
            <th>Name</th>
            <th>Age</th>
            <th>Details</th>
            <th>Edit</th>
            <th>Delete</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var student in Model)
        {
            <tr>
                <td>@student.Id</td>
                <td>@student.Name</td>
                <td>@student.Age</td>
                <td>
                    <a href="@Url.Action("Details", "Student", new { id = student.Id })">View Details</a>
                </td>
                <td>
                    <a href="@Url.Action("Edit", "Student", new { id = student.Id })">Edit</a>
                </td>
                <td>
                    <a href="@Url.Action("Delete", "Student", new { id = student.Id })">Delete</a>
                </td>
            </tr>
        }
    </tbody>
</table>

<a href="@Url.Action("Create", "Student")" class="btn btn-primary">Create New Student</a>
```

### Create View
In Views/Student/Create.cshtml:
```html
@model StudentMVCApp.Models.Student

<h2>Create Student</h2>

<form method="post">
    <div class="form-group">
        <label for="Name">Name</label>
        <input type="text" class="form-control" id="Name" name="Name" value="@Model.Name" required />
    </div>
    <div class="form-group">
        <label for="Age">Age</label>
        <input type="number" class="form-control" id="Age" name="Age" value="@Model.Age" required />
    </div>
    <button type="submit" class="btn btn-primary">Create</button>
</form>

```

### Edit View
In Views/Student/Edit.cshtml:
```html
@model StudentMVCApp.Models.Student

<h2>Edit Student</h2>

<form method="post">
    <div class="form-group">
        <label for="Name">Name</label>
        <input type="text" class="form-control" id="Name" name="Name" value="@Model.Name" required />
    </div>
    <div class="form-group">
        <label for="Age">Age</label>
        <input type="number" class="form-control" id="Age" name="Age" value="@Model.Age" required />
    </div>
    <button type="submit" class="btn btn-primary">Save Changes</button>
</form>
```

### Delete View
In Views/Student/Delete.cshtml:
```html
@model StudentMVCApp.Models.Student

<h2>Delete Student</h2>

<h3>Are you sure you want to delete this student?</h3>

<div>
    <h4>@Model.Name</h4>
    <p>Age: @Model.Age</p>
</div>

<form method="post">
    <button type="submit" class="btn btn-danger">Delete</button>
    <a href="@Url.Action("Index", "Student")" class="btn btn-secondary">Cancel</a>
</form>

```

### Add a Migration: In the Package Manager Console- run:
```powershell
Add-Migration InitialCreate
```
### Apply the Migration: After the migration is created, apply it to the database:
```powershell
Update-Database
```
