# 📘 ASP.NET Core Web API — User Image Upload Example

This project demonstrates how to build a simple **ASP.NET Core Web API** that allows users to upload profile images and store their file names in a database.

---

## 📑 Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Project Structure](#project-structure)
4. [Database Setup](#database-setup)
5. [API Endpoints](#api-endpoints)
6. [Code Implementation](#code-implementation)
   - [User Model](#1-user-model)
   - [Database Context](#2-database-context)
   - [User Controller](#3-user-controller)
7. [Configuration](#configuration)
8. [Migrations & Database Update](#migrations--database-update)
9. [Testing the API](#testing-the-api)
10. [Example Result](#example-result)

---

## 🧠 Overview

This **Web API** provides a basic endpoint that:
- Accepts user profile image uploads.
- Validates image extensions (`.jpg`, `.jpeg`, `.png`).
- Saves files in the `wwwroot/images` folder.
- Stores the file name in the database.

---

## ⚙️ Features

✅ Upload user profile images  
✅ Validate allowed file types  
✅ Store file names in SQL database  
✅ Supports **SQL Server** or **MySQL (Pomelo)**  
✅ Includes Swagger for API testing  

---

## 📁 Project Structure

app/
│
├── Controllers/
│ └── UserController.cs
│
├── Data/
│ └── AppDbContext.cs
│
├── Models/
│ └── User.cs
│
├── wwwroot/
│ └── images/ # Uploaded files are stored here
│
├── appsettings.json
└── Program.cs





---

## 🧱 Database Setup

### 📂 Step 1: Create a Models Folder

**`Models/User.cs`**
```csharp
using System.ComponentModel.DataAnnotations;

namespace app.Models
{
    public class User
    {
        public int UserId { get; set; }

        [MaxLength(300)]
        public string? UserProfile { get; set; } = null;
    }
}


📂 Step 2: Create a Data Folder

Data/AppDbContext.cs

using app.Models;
using Microsoft.EntityFrameworkCore;

namespace app.Data
{
    public class AppDbContext : DbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
        {
        }

        public DbSet<User> Users { get; set; } = null!;
    }
}


🧩 Step 3: Create the User Controller

Controllers/UserController.cs

using Microsoft.AspNetCore.Mvc;
using app.Models;
using app.Data;
using Microsoft.EntityFrameworkCore;
using Microsoft.AspNetCore.Identity;

namespace app.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class UserController(AppDbContext context) : ControllerBase
    {
        private readonly AppDbContext _context = context;
        private readonly PasswordHasher<User> _passwordHasher = new();

        // Upload Profile Image
        [HttpPost("image")]
        public async Task<IActionResult> Register([FromForm] IFormFile? image)
        {
            string? fileName = null;

            if (image != null && image.Length > 0)
            {
                var allowedExtension = new[] { ".jpg", ".jpeg", ".png" };
                var fileExtension = Path.GetExtension(image.FileName).ToLower();

                if (!allowedExtension.Contains(fileExtension))
                {
                    return BadRequest(new { message = "Invalid File Extension! Allowed: .jpg, .jpeg, .png" });
                }

                // Create upload directory if missing
                var uniqueFileName = Guid.NewGuid().ToString() + fileExtension;
                var uploadsFolder = Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", "images");

                if (!Directory.Exists(uploadsFolder))
                {
                    Directory.CreateDirectory(uploadsFolder);
                }

                var filePath = Path.Combine(uploadsFolder, uniqueFileName);

                using (var stream = new FileStream(filePath, FileMode.Create))
                {
                    await image.CopyToAsync(stream);
                }

                fileName = uniqueFileName;
            }

            var user = new User
            {
                UserProfile = fileName,
            };

            _context.Users.Add(user);
            await _context.SaveChangesAsync();

            return Ok(new { message = "User Added Successfully" });
        }
    }
}



Configuration
🧾 appsettings.json

{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=UserDb;Trusted_Connection=True;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}


Program.cs

using app.Data;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// ✅ Configure MySQL Database (Pomelo)
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseMySql(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        new MySqlServerVersion(new Version(8, 0, 39)) // 👈 use your MySQL version here
    ));

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseStaticFiles();
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();



Migrations & Database Update

Run the following commands in the terminal:

dotnet ef migrations add InitialCreate
dotnet ef database update
