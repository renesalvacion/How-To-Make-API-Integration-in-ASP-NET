📘 ASP.NET Core Web API — User Image Upload Example

This project demonstrates how to build a simple ASP.NET Core Web API that allows users to upload profile images and store their file names in a database.

🧱 1. Create the Database and Tables
📂 Step 1: Create a Models folder

Add a file named User.cs:

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

🗄️ 2. Create the Database Context
📂 Step 2: Create a Data folder

Add a file named AppDbContext.cs:

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

🧩 3. Create the User Controller
📂 Step 3: Create a Controllers folder

Add a file named UserController.cs:

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

⚙️ 4. Configure the Database Connection

In your appsettings.json, add the connection string:

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


In Program.cs, register the AppDbContext:

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



🧱 5. Apply Migrations

Run the following commands in the terminal:

dotnet ef migrations add InitialCreate
dotnet ef database update
