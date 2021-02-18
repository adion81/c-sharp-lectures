# Day 8 - Entity Framework Core & LINQ

<img src="https://miro.medium.com/max/480/1*SnZqHENpIMiEKsg999Q0DQ.png" alt="ef-core" />

## More Tools

Before we can start working with Entity Framework, we need to install the following tool...

```
dotnet tool install --global dotnet-ef
```

This will be installed globally, meaning we only need to do this once on our computer.

## Adding Packages

After starting a new project and changing directory into it...

```
dotnet new mvc --no-https -o LizardPirates
cd LizardPirates
```

we need to add in new packages that will allow us to connect to a MySQL Database. 

```
dotnet add package Microsoft.EntityFrameworkCore.Design --version 3.1.5
dotnet add package Pomelo.EntityFrameworkCore.MySql --version 3.1.1
```

## Making a Model

Once installed we need to create a model `Models/Lizard.cs` that we can use to be a table in our database.

```cs
using System;
using System.ComponentModel.DataAnnotations;

namespace LizardPirates.Models
{
    public class Lizard
    {
        [Key]
        public int LizardId { get; set; }
        public string Name { get; set; }
        public string LizardType { get; set; }
        public string PirateRole { get; set; }
        public DateTime CreatedAt { get; set; } = DateTime.Now;
        public DateTime UpdatedAt { get; set; } = DateTime.Now;
    }
}
```

## Making a Context

Next we need to create a context class `Models/MyContext.cs`... this is the file we will use to connect to our database.

```cs
using Microsoft.EntityFrameworkCore;
 
namespace LizardPirates.Models
{
    public class MyContext : DbContext
    {
        public MyContext(DbContextOptions options) : base(options) { }
        
        // this is the variable we will use to connect to the MySQL table, Lizards
        public DbSet<Lizard> Lizards { get; set; }
    }
}
```

## Modifying Startup

And once we have this file, we need to update our `Startup.cs` file...

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using LizardPirates.Models;
using Microsoft.EntityFrameworkCore;


namespace LizardPirates
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSession();
            services.AddDbContext<MyContext>(o => o.UseMySql (Configuration["DBInfo:ConnectionString"]));
            services.AddControllersWithViews();
            services.AddMvc(option => option.EnableEndpointRouting = false);
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseDeveloperExceptionPage();
            app.UseStaticFiles();
            app.UseSession();
            app.UseMvc();
        }
    }
}
```

## Setting up appsettings

We also need to modify our `appsettings.json` to include "DBInfo"...

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*",
  "DBInfo":
  {
    "Name": "MySQLconnect",
    "ConnectionString": "server=localhost;userid=root;password=root;port=3306;database=mydb;SslMode=None"
  }
}
```

Make sure you change the `mydb` to whatever you want your schema to be called...

**Note:** if you connect using a different port, username, or password make sure you update the "ConnectionString" accordingly.

## Controller Changes

Next we need to [dependency inject](https://stackify.com/dependency-injection/) our `MyContext` into our `HomeController` class...

```cs
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using LizardPirates.Models;

namespace LizardPirates.Controllers
{
    public class HomeController : Controller
    {

        private MyContext _context { get; set; } 

        public HomeController(MyContext context)
        {
            _context = context;
        }

        [HttpGet("")]
        public IActionResult Index()
        {
            return View();
        }
    }
}
```

This `_context` is an instance of our `MyContext` class that is helpfully provided to us when ASP.Net Core creates our `HomeController`.

## Migrating our Database

After creating these files, we need to create the appropriate schema in our database...

```
dotnet ef migrations add initial -v
dotnet ef database update -v
```

This will create a migration called `initial` and then apply that migration to our MySQL database.

## Create

An example of how we can enter a new `Lizard` into our database would look like...

```cs
[HttpPost("create")]
public IActionResult Create(Lizard lizard)
{
    _context.Lizards.Add(lizard);
    _context.SaveChanges();
    return Redirect("/");
}
```

## Read

An example of how we can get back our lizards to display to in the frontend would look like...

```cs
[HttpGet("")]
public IActionResult Index()
{
    List<Lizard> Lizards = _context.Lizards.ToList();
    return View(Lizards);
}
```
## LINQ

<img src="https://www.zelda.com/links-awakening/assets/img/home/hero-char.png" alt="Link" width="400px">

<em>The Hero of (saving) Time</em>

LINQ is an acronym for Language Integrated Query

We can use LINQ with any type of dataset or SQL database.<br>
There are two notations of LINQ `Method` and `Query`<br>
<br>
We will be focusing on the Method Notation.

#### Lambda Expressions

<img src="https://res.cloudinary.com/teepublic/image/private/s--QH6eyKSY--/c_fit,g_north_west,h_840,w_837/co_000000,e_outline:40/co_000000,e_outline:inner_fill:1/co_ffffff,e_outline:40/co_ffffff,e_outline:inner_fill:1/co_bbbbbb,e_outline:3:1000/c_mpad,g_center,h_1260,w_1260/b_rgb:eeeeee/c_limit,f_jpg,h_630,q_90,w_630/v1582915404/production/designs/8222015_0.jpg" alt="Lambda" width="400px">

Lambda Expressions are a way we can simplify a function into one line.<br>
We use the `=>` to express a function returning something.<br>
<br>
Now we can use these expressions to query our database for the things we want to display on our page.

`_context.Users.FirstOrDefault( u => u.UserId == userId);` will return the first user that it finds in the database.<br>

There are methods for filtering, sorting, and general purpose.

#### Filtering
<ol>
    <li>FirstOrDefault</li>
    <li>Where</li>
    <li>Select</li>
</ol>

#### Sorting
<ol>
    <li>OrderBy</li>
    <li>OrderByDescending</li>
    <li>ThenBy</li>
    <li>ThenByDescending</li>
</ol>

#### General Purpose
<ol>
    <li>Min</li>
    <li>Max</li>
    <li>Take</li>
    <li>ToList</li>
</ol>
