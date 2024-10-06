# Deploying a RESTful API with C#, Docker & AWS ECS

- [C# and DotNet](#c-and-dotnet)
  - [Basic install](#basic-install)
  - [Create ASP.Net Web API Project](#create-aspnet-web-api-project)
  - [Add Ef Core](#add-ef-core)
  - [Add Controller](#add-controller)
  - [Automatically Initialise Database](#automatically-initialise-database)
  - [Add .gitignore for C#](#add-gitignore-for-c)
  - [Troubleshooting \& Tips](#troubleshooting--tips)
- [Docker and SQL Server](#docker-and-sql-server)
  - [Basic Install](#basic-install-1)
  - [Run SQL Server](#run-sql-server)
  - [Beekeeper connect database](#beekeeper-connect-database)
- [Build Project Image and Docker Compose](#build-project-image-and-docker-compose)
  - [Add Dockerfile and .dockerignore to Project](#add-dockerfile-and-dockerignore-to-project)
  - [Build project image](#build-project-image)
  - [Docker Compose](#docker-compose)
  - [Upload Image to Docker Hub](#upload-image-to-docker-hub)
  - [Troubleshooting \& Tips](#troubleshooting--tips-1)
- [Deploy Project Image and Database to ECS](#deploy-project-image-and-database-to-ecs)
  - [Create Task definitions](#create-task-definitions)
  - [Create Clusters](#create-clusters)
  - [Deploy your task to cluster](#deploy-your-task-to-cluster)
  - [Troubleshooting \& Tips](#troubleshooting--tips-2)

## C# and DotNet

### Basic install

1. Set up development environment
    - Start Visual Studio Code.
    - Install C# Dev Kit (The [C# extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) and the [.NET Install Tool](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.vscode-dotnet-runtime) will automatically be installed)
2. Install dotnet sdk 8.0
    
     [Download .NET 8.0 SDK(v8.0.402)-macOs x64 Installer](https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/sdk-8.0.402-macos-x64-installer)
    

### Create ASP.Net Web API Project

There are two ways to create a Web API project:

1. Create in Visual Studio Code.
    1. Start Visual Studio Code.
    2. Command+Shift+P
    3. Select **Create .NET Project**
    4. Select **ASP.Net Core Web API**
    5. Select the location where you would like the new project to be created.
    6. Give your new project a name, like "TrilloBackend".
    7. Select **Create Project,** the project template creates a `WeatherForecast` API with support for [Swagger](https://learn.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger?view=aspnetcore-8.0).
        
2. Create with terminal
    
    [Tutorial: Create a web API with ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-8.0&tabs=visual-studio-code)
    
    1. Open terminal, change directories (`cd`) to the folder that will contain the project folder. The project template creates a `WeatherForecast` API with support for [Swagger](https://learn.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger?view=aspnetcore-8.0).
        
        ```bash
        dotnet new webapi --use-controllers -o TrilloBackend
        cd TrilloBackend
        dotnet add package Microsoft.EntityFrameworkCore.InMemory
        code -r ../TrilloBackend
        ```
        
    2. Trust the HTTPS development certificate
        
        ```bash
        dotnet dev-certs https --trust
        ```
        
3. Run the project, check the result with: [https://localhost](https://localhost/):PORT/swagger/index.html
    - Control + F5
    - write in terminal: `dotnet run`
    - Run → Run without Debugging

### Add Ef Core

There are two common ORM options: EF Core and Dapper, here is the main differences:

![ef](https://github.com/user-attachments/assets/d6bfd0a7-aea1-4e99-9fd5-de51c4b2dda7)

1. Open terminal, run the following commands to install EF Core and Sql Server driver.
    
    ```bash
    dotnet add package Microsoft.EntityFrameworkCore --version 8.0.8
    dotnet add package Microsoft.EntityFrameworkCore.SqlServer
    ```
    
2. Create the model. In the project directory, create **Models folder**, and create [**Model.cs**](https://learn.microsoft.com/en-nz/ef/core/get-started/overview/first-app?tabs=netcore-cli) and other model files with the following code. (In this Project, I use Booking.cs, Hotel.cs, Order.cs, Review.cs, TrilloContext.cs instead of Model.cs)
    - **Models Code**
        
        **Models/Booking.cs**
        
        ```csharp
        public class Booking
        {
          public int BookingId { get; set; }
          // FK
          public int? HotelId { get; set; }
          public int? OrderId { get; set; }
          // Properties
          public DateTime Date { get; set; }
          public double? Price { get; set; }
          public bool? isAvailable { get; set; }
          // Internal
          public DateTime CreatedAt { get; set; }
          public DateTime UpdatedAt { get; set; }
          // Relation
          public Order Order { get; set; } = new();
          public Hotel Hotel { get; set; } = new();
        }
        ```
        
        **Models/Hotel.cs**
        
        ```csharp
        public class Hotel
        {
          public int HotelId { get; set; }
          // Properties
          public string? Name { get; set; }
          public List<string> Gallery { get; set; } = [];
          public string? Description { get; set; }
          public string? Address { get; set; }
          public int? TotalVote { get; set; }
          // Internal
          public DateTime CreatedAt { get; set; }
          public DateTime UpdatedAt { get; set; }
          // Relation
          public List<Review> Reviews { get; set; } = new();
          public List<Booking> Bookings { get; set; } = new();
        }
        ```
        
        **Models/Order.cs**
        
        ```csharp
        public class Order
        {
          public int OrderId { get; set; }
          // FK
          public int? UserId { get; set; }
          // Properties
          public double? Amount { get; set; }
          public string? GuestName { get; set; }
          // Internal
          public DateTime CreatedAt { get; set; }
          public DateTime UpdatedAt { get; set; }
          // Relation
          public List<Booking> Bookings { get; set; } = new();
        }
        ```
        
        **Models/Review.cs**
        
        ```csharp
        public class Review
        {
          public int ReviewId { get; set; }
          // FK
          public int? UserId { get; set; }
          public int? HotelId { get; set; }
          // Properties
          public string? Body { get; set; }
          public double? Rating { get; set; }
          // Internal
          public DateTime CreatedAt { get; set; }
          public DateTime UpdatedAt { get; set; }
          // Relation
          public Hotel Hotel { get; set; } = new();
        }
        
        ```
        
        **Models/TrilloContext.cs**
        
        ```csharp
        using Microsoft.EntityFrameworkCore;
        
        public class TrilloContext : DbContext
        {
          public DbSet<Hotel> Hotels { get; set; }
          public DbSet<Review> Reviews { get; set; }
          public DbSet<Order> Orders { get; set; }
          public DbSet<Booking> Bookings { get; set; }
        
          protected override void OnConfiguring(DbContextOptionsBuilder options)
          {
            var builder = WebApplication.CreateBuilder();
            var configuration = new ConfigurationBuilder()
              .AddJsonFile("appsettings.json")
              // Load configuration for current environment.
              .AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", true)
              .AddEnvironmentVariables()
              .Build();
            // Connect to the database.
            options.UseSqlServer(configuration.GetConnectionString("TrilloDatabase"));
          }
        
          // Generate Date/time.
          protected override void OnModelCreating(ModelBuilder modelBuilder)
          {
            modelBuilder.Entity<Booking>(e => {
              e.Property(b => b.CreatedAt).HasDefaultValueSql("getdate()");
              e.Property(b => b.UpdatedAt).HasDefaultValueSql("getdate()");
            });
        
            modelBuilder.Entity<Hotel>(e => {
              e.Property(b => b.CreatedAt).HasDefaultValueSql("getdate()");
              e.Property(b => b.UpdatedAt).HasDefaultValueSql("getdate()");
            });
        
            modelBuilder.Entity<Order>(e => {
              e.Property(b => b.CreatedAt).HasDefaultValueSql("getdate()");
              e.Property(b => b.UpdatedAt).HasDefaultValueSql("getdate()");
            });
        
            modelBuilder.Entity<Review>(e => {
              e.Property(b => b.CreatedAt).HasDefaultValueSql("getdate()");
              e.Property(b => b.UpdatedAt).HasDefaultValueSql("getdate()");
            });
          }
        }
        ```
        
3. Add Database ConnectionStrings to **appsettings.json**
    
    ```json
    "ConnectionStrings": {
        "TrilloDatabase": "Data Source=localhost,1433\\Catalog=trillo;Database=trillo;Trusted_Connection=false;User Id=SA;password=YOUR_PASSWORD;TrustServerCertificate=True;"}
    ```
    
    - Data Source: replace ‘1433’ with your database port;
    - Database: replace ‘trillo’ with your database name;
    - Attention **User Id** need a space;
4. Add **appsettings.Production.json** for production environement.
   - For docker compose: Data Source=sqlserver,1433;
   - For ECS: Data Source=localhost,1433
    
    ```csharp
    {
      "Logging": {
        "LogLevel": {
          "Default": "Information",
          "Microsoft.AspNetCore": "Warning"
        }
      },
      "AllowedHosts": "*",
      "ConnectionStrings": {
        "TrilloDatabase": "Data Source=localhost,1433\\Catalog=trillo;Database=trillo;Trusted_Connection=false;User Id=SA;password=YOUR_PASSWORD;TrustServerCertificate=True;"
      }
    }
    ```
    
5. If your database is up and running, you can run the following commands to create migrations and apply to database. 
    
    ```bash
    # install tool and package
    dotnet tool install --global dotnet-ef
    dotnet add package Microsoft.EntityFrameworkCore.Design
    
    # The first time, you need to execute this command to create the Migrations folder
    dotnet ef migrations add InitialCreate
    # this command will be replaced as migration() in the section:
    dotnet ef database update
    ```
    

6. Links: 

    [How YOU can use an ORM in .NET Core and C# to type less SQL](https://softchris.github.io/pages/dotnet-orm.html#resources)

    [Getting started with EF core: A beginner's guide](https://learningdaily.dev/getting-started-with-ef-core-a-beginners-guide-9b398488d222)

### Add Controller

  [Tutorial: Create a web API with ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-8.0&tabs=visual-studio-code)

1. Create Controller, run the following code to create Controllers folder with file HotelsController.cs.
    
    ```bash
    dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
    dotnet add package Microsoft.EntityFrameworkCore.Design
    dotnet add package Microsoft.EntityFrameworkCore.SqlServer
    dotnet add package Microsoft.EntityFrameworkCore.Tools
    dotnet tool uninstall -g dotnet-aspnet-codegenerator
    dotnet tool install -g dotnet-aspnet-codegenerator
    dotnet tool update -g dotnet-aspnet-codegenerator
    
    dotnet aspnet-codegenerator controller -name HotelsController -async -api -m Hotel -dc TrilloContext -outDir Controllers
    ```
    
2. Update HotelsController.cs to add nameof in POST return, to avoid hardcode the action name in CreatedAtAction function.

    <img width="740" alt="imagehotelcontroller" src="https://github.com/user-attachments/assets/8600b236-8ec0-4a8b-9eb2-fd07fcf8704f">
    
3. Add Controller service to program.cs
    
    ```csharp
    using Microsoft.EntityFrameworkCore;
    
    builder.Services.AddControllers();
    builder.Services.AddDbContext<TrilloContext>();
    
    app.MapControllers();
    ```
    
- **Code**
    
    **HotelsController.cs**
    
    ```csharp
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.EntityFrameworkCore;
    
    namespace TrilloBackend.Controllers
    {
    
        [Route("api/hotels")]
        [ApiController]
        public class HotelsController : ControllerBase
        {
            private readonly TrilloContext _context;
    
            public HotelsController(TrilloContext context)
            {
                _context = context;
            }
    
            // GET: api/hotels
            [HttpGet]
            public async Task<ActionResult<IEnumerable<Hotel>>> GetHotels()
            {
                return await _context.Hotels
                    .Include(e => e.Reviews)
                    .Include(e => e.Bookings)
                    .ToListAsync();
            }
    
            // GET: api/hotels/5
            [HttpGet("{id}")]
            public async Task<ActionResult<Hotel>> GetHotel(long id)
            {
                var hotel = await _context.Hotels
                    .Include(e => e.Reviews)
                    .Include(e => e.Bookings)
                    .FirstOrDefaultAsync(e => e.HotelId == id)
                   ;
    
                if (hotel == null)
                {
                    return NotFound();
                }
    
                return hotel;
            }
    
            // PUT: api/hotels/5
            // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
            [HttpPut("{id}")]
            public async Task<IActionResult> PutHotel(int id, Hotel hotel)
            {
                if (id != hotel.HotelId)
                {
                    return BadRequest();
                }
    
                _context.Entry(hotel).State = EntityState.Modified;
    
                try
                {
                    await _context.SaveChangesAsync();
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!HotelExists(id))
                    {
                        return NotFound();
                    }
                    else
                    {
                        throw;
                    }
                }
    
                return NoContent();
            }
    
            // POST: api/hotels
            // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
            [HttpPost]
            public async Task<ActionResult<Hotel>> PostHotel(Hotel hotel)
            {
                _context.Hotels.Add(hotel);
                await _context.SaveChangesAsync();
    
                return CreatedAtAction(nameof(GetHotel), new { id = hotel.HotelId }, hotel);
            }
    
            // PostGET: api/hotels/search
            [HttpPost("search")]
            public async Task<ActionResult<IEnumerable<Hotel>>> PostGetHotel(string? name, string? address)
            {
                // check Name and Address record
                var hotelQuery = _context.Hotels
                    .Include(e => e.Reviews)
                    .Include(e => e.Bookings)
                    .AsQueryable();
    
                // name && address
                if (!string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(address))
                {   
                    var result = await hotelQuery
                        .Where(e => EF.Functions.Like(e.Name, $"%{name}%") && EF.Functions.Like(e.Address, $"%{address}%"))
                        .ToListAsync();
                    return result;
                }
    
                // check name without address
                if (!string.IsNullOrEmpty(name))
                {
                    var result = await hotelQuery
                        .Where(e => EF.Functions.Like(e.Name, $"%{name}%"))
                        .ToListAsync();
                    if (result.Any())
                        return result;
                }
    
                // chenck address without name
                if (!string.IsNullOrEmpty(address))
                {
                    var result = await hotelQuery
                        .Where(e => EF.Functions.Like(e.Address, $"%{address}%"))
                        .ToListAsync();
                    if (result.Any())
                        return result;
                }
    
                // return []
                return new List<Hotel>();
            }
    
            // DELETE: api/hotels/5
            [HttpDelete("{id}")]
            public async Task<IActionResult> DeleteHotel(int id)
            {
                var hotel = await _context.Hotels.FindAsync(id);
                if (hotel == null)
                {
                    return NotFound();
                }
    
                _context.Hotels.Remove(hotel);
                await _context.SaveChangesAsync();
    
                return NoContent();
            }
    
            private bool HotelExists(int id)
            {
                return _context.Hotels.Any(e => e.HotelId == id);
            }
        }
    }
    ```
    
    **Program.cs**
    
    ```csharp
    using System.Text.Json.Serialization;
    using Microsoft.EntityFrameworkCore;
    var builder = WebApplication.CreateBuilder(args);
    
    // Add services to the container.
    builder.Services.AddControllers();
    // Fixing the error “A possible object cycle was detected”.
    builder.Services.AddControllers().AddJsonOptions(x =>
        x.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles);
    builder.Services.AddDbContext<TrilloContext>();
    builder.Services.AddEndpointsApiExplorer();
    builder.Services.AddSwaggerGen();
    
    var app = builder.Build();
    
    // Prepare database.
    using (var scope = app.Services.CreateScope())
    {
        var services = scope.ServiceProvider;
        var context = services.GetRequiredService<TrilloContext>();
        if (app.Environment.IsDevelopment())
        {
            Console.WriteLine($"DbConnectionString: {context.Database.GetConnectionString()}");
        }
        // Create Database and apply migrations.
        context.Database.Migrate();
        // Seed data to tables.
        DbInitializer.Initialize(context);
    }
    
    // Configure the HTTP request pipeline.
    if (app.Environment.IsDevelopment())
    {
        app.UseSwagger();
        app.UseSwaggerUI();
    }
    
    // Test api.
    app.MapGet("/Ping", () =>
    {
        return "Pong";
    })
    .WithName("Ping")
    .WithOpenApi();
    
    app.UseHttpsRedirection();
    app.MapControllers();
    app.Run();
    
    ```
    

### Automatically Initialise Database

[How and where to call Database.EnsureCreated and Database.Migrate?](https://stackoverflow.com/questions/38238043/how-and-where-to-call-database-ensurecreated-and-database-migrate)

[Create the database](https://learn.microsoft.com/en-nz/aspnet/core/data/ef-rp/intro?view=aspnetcore-8.0&tabs=visual-studio-code#create-the-database)

1. Change program.cs, add `Migrate()` to create database, apply migration and seed data.

    <img width="667" alt="migrate" src="https://github.com/user-attachments/assets/17028562-36f7-4c3c-8ff2-9d5701a76a73">
    
2. Add code that populates the database with data. Create `Data/DbInitializer.cs` with the following code:
    - Data/DbInitializer.cs
        
        ```csharp
        public static class DbInitializer {
          public static void Initialize(TrilloContext context) {
        
            // Check if any data exist.
            if (context.Hotels.Any()) {
              return;
            }
        
            var hotels = new Hotel[] {
              new Hotel{Name="The Venetian Macao",Gallery=["https://image-tc.galaxy.tf/wijpeg-9vualzt3dbue0hi00ba4q49ub/chatwalhotelnyc-c-004-build-crop.jpg?width=1920"],Description="This is a Hotel located in Macao",Address="Marco China",TotalVote=99, Bookings=[]},
              new Hotel{Name="The Parisian Macao",Gallery=["https://dynamic-media-cdn.tripadvisor.com/media/photo-s/01/ea/d8/2d/grand-canal-shoppes.jpg?w=600&h=-1&s=1"],Description="This is a Hotel located in China",Address="China",TotalVote=96},
              new Hotel{Name="Nina Hotel Tsuen Wan West",Gallery=["https://dynamic-media-cdn.tripadvisor.com/media/daodao/photo-s/0c/7b/d9/f2/exterior-night.jpg?w=600&h=-1&s=1"],Description="This is a Hotel located in Auckland",Address="Auckland",TotalVote=79},
              new Hotel{Name="Nina Hotel Causeway Bay",Gallery=["https://cf.bstatic.com/xdata/images/hotel/max1024x768/118479281.jpg?k=d5090d90ae7919b4637f2d7d08d0ae0df7517e4185eaebed5a5907e53cd3801d&o=&hp=1"],Description="This is a Hotel located in NZ",Address="NZ",TotalVote=39},
              new Hotel{Name="Conrad Macao",Gallery=["test1"],Description="This is a Hotel located in Beijing",Address="Beijing",TotalVote=27},
              new Hotel{Name="Legend Palace Hotel",Gallery=["test2"],Description="This is a Hotel located in Shanghai",Address="Shanghai",TotalVote=30},
              new Hotel{Name="Royal Plaza Hotel",Gallery=["test3"],Description="This is a Hotel located in Shenzhen",Address="Shenzhen",TotalVote=15},
            };
            context.Hotels.AddRange(hotels);
            context.SaveChanges();
        
            var reviews = new Review[] {
              new Review{UserId=101,HotelId=hotels[0].HotelId,Body="The environment is very nice.",Rating=8.8},
              new Review{UserId=102,HotelId=hotels[1].HotelId,Body="The food is very nice.",Rating=7.8},
              new Review{UserId=103,HotelId=hotels[2].HotelId,Body="The price is very nice.",Rating=6.8},
              new Review{UserId=104,HotelId=hotels[3].HotelId,Body="The bed is very nice.",Rating=3.8},
              new Review{UserId=105,HotelId=hotels[4].HotelId,Body="The location is very nice.",Rating=8.5},
              new Review{UserId=106,HotelId=hotels[5].HotelId,Body="The transport is very nice.",Rating=1.8},
              new Review{UserId=107,HotelId=hotels[6].HotelId,Body="The people is very nice.",Rating=8.2},
            };
            context.Reviews.AddRange(reviews);
            context.SaveChanges();
        
            var orders = new Order[] {
              new Order{UserId=101,Amount=200.3,GuestName="Ella"},
              new Order{UserId=102,Amount=229,GuestName="Hank"},
              new Order{UserId=103,Amount=93.4,GuestName="Mark"},
              new Order{UserId=104,Amount=3784.4,GuestName="Tala"},
              new Order{UserId=105,Amount=278743.4,GuestName="Ada"},
              new Order{UserId=106,Amount=3985,GuestName="Hui"},
              new Order{UserId=107,Amount=3875.6,GuestName="Kswi"},
              new Order{UserId=108,Amount=438273.5,GuestName="Edsa"},
            };
            context.Orders.AddRange(orders);
            context.SaveChanges();
        
            var bookings = new Booking[] {
              new Booking{HotelId=hotels[0].HotelId,OrderId=orders[0].OrderId,Price=83.3,isAvailable=true},
              new Booking{HotelId=hotels[1].HotelId,OrderId=orders[1].OrderId,Price=384.5,isAvailable=false},
              new Booking{HotelId=hotels[2].HotelId,OrderId=orders[2].OrderId,Price=22.3,isAvailable=true},
              new Booking{HotelId=hotels[3].HotelId,OrderId=orders[3].OrderId,Price=55.6,isAvailable=true},
              new Booking{HotelId=hotels[4].HotelId,OrderId=orders[4].OrderId,Price=394.5,isAvailable=false},
              new Booking{HotelId=hotels[5].HotelId,OrderId=orders[0].OrderId,Price=78,isAvailable=false},
              new Booking{HotelId=hotels[6].HotelId,OrderId=orders[6].OrderId,Price=92.4,isAvailable=true},
              new Booking{HotelId=hotels[2].HotelId,OrderId=orders[2].OrderId,Price=100,isAvailable=false},
            };
            context.Bookings.AddRange(bookings);
            context.SaveChanges();
          }
        }
        ```
        

### Add .gitignore for C#

```csharp
## Ignore Visual Studio temporary files, build results, and
## files generated by popular Visual Studio add-ons.

# User-specific files
*.suo
*.user
*.sln.docstates

# Build results

[Dd]ebug/
[Rr]elease/
x64/
[Bb]in/
[Oo]bj/

# MSTest test Results
[Tt]est[Rr]esult*/
[Bb]uild[Ll]og.*

*_i.c
*_p.c
*_i.h
*.ilk
*.meta
*.obj
*.pch
*.pdb
*.pgc
*.pgd
*.rsp
*.sbr
*.tlb
*.tli
*.tlh
*.tmp
*.tmp_proj
*.log
*.vspscc
*.vssscc
.builds
*.pidb
*.log
*.svclog
*.scc

# Visual C++ cache files
ipch/
*.aps
*.ncb
*.opensdf
*.sdf
*.cachefile

# Visual Studio profiler
*.psess
*.vsp
*.vspx

# Guidance Automation Toolkit
*.gpState

# ReSharper is a .NET coding add-in
_ReSharper*/
*.[Rr]e[Ss]harper
*.DotSettings.user

# Click-Once directory
publish/

# Publish Web Output
*.Publish.xml
*.pubxml
*.azurePubxml

# NuGet Packages Directory
## TODO: If you have NuGet Package Restore enabled, uncomment the next line
packages/
## TODO: If the tool you use requires repositories.config, also uncomment the next line
!packages/repositories.config

# Windows Azure Build Output
csx/
*.build.csdef

# Windows Store app package directory
AppPackages/

# Others
sql/
*.Cache
ClientBin/
[Ss]tyle[Cc]op.*
![Ss]tyle[Cc]op.targets
~$*
*~
*.dbmdl
*.[Pp]ublish.xml

*.publishsettings

# RIA/Silverlight projects
Generated_Code/

# Backup & report files from converting an old project file to a newer
# Visual Studio version. Backup files are not needed, because we have git ;-)
_UpgradeReport_Files/
Backup*/
UpgradeLog*.XML
UpgradeLog*.htm

# SQL Server files
App_Data/*.mdf
App_Data/*.ldf

# =========================
# Windows detritus
# =========================

# Windows image file caches
Thumbs.db
ehthumbs.db

# Folder config file
Desktop.ini

# Recycle Bin used on file shares
$RECYCLE.BIN/

# Mac desktop service store files
.DS_Store

_NCrunch*

```

### Troubleshooting & Tips

- Tips
    1. Each time the database or tables are changed, **migrations** and **updates** must be executed
    2. Generated Datetime in table: [Generate Values - EF Core](https://learn.microsoft.com/en-us/ef/core/modeling/generated-properties?tabs=data-annotations)

         <img width="591" alt="datetime" src="https://github.com/user-attachments/assets/5b5c95c4-6589-4078-8553-89a81b74096c">
        
    3. Add different table join

        [Eager Loading of Related Data](https://learn.microsoft.com/en-us/ef/core/querying/related-data/eager)
        
        <img width="332" alt="join" src="https://github.com/user-attachments/assets/bfe05743-e8da-459c-b670-b289d8d62ab9">
        
    4. Add SQL `LIKE` operation: **EF.Functions.Like**

        [Entity framework EF.Functions.Like vs string.Contains](https://stackoverflow.com/questions/45708715/entity-framework-ef-functions-like-vs-string-contains)

         <img width="533" alt="like" src="https://github.com/user-attachments/assets/f07b7624-2cac-4a20-8896-b3d18ae291e1">
        
- Troubleshooting
    1. [How to connect ASP.Net Core to a SQL Server Docker container on Mac](https://stackoverflow.com/questions/53024227/how-to-connect-asp-net-core-to-a-sql-server-docker-container-on-mac)

        <img width="738" alt="appseting" src="https://github.com/user-attachments/assets/74100c14-fddb-4969-89dc-a1a1fd1bf9b3">
        
    2. Error: A connection was successfully established with the server, but then an error occurred during the pre-login handshake. ‣
        
        **Fix:** Add `TrustServerCertificate=True` to ConnectionStrings
        
    3. **Error: “A possible object cycle was detected” in different versions of ASP.NET Core**
        
        [Fixing the error “A possible object cycle was detected” in different versions of ASP.NET Core](https://gavilan.blog/2021/05/19/fixing-the-error-a-possible-object-cycle-was-detected-in-different-versions-of-asp-net-core/)

        <img width="748" alt="cycleerror" src="https://github.com/user-attachments/assets/b65d9a39-288a-4def-8594-282eed677f45">
        
    4. Foreign-Key
        - Link: [How can I retrieve Id of inserted entity using Entity framework?](https://stackoverflow.com/questions/5212751/how-can-i-retrieve-id-of-inserted-entity-using-entity-framework)
        - `context.Reviews.AddRange(reviews)` will automatically generate the primary key, which can be called directly later. The error occurs due to a conflict; a foreign key needs to reference the primary key for it to work.

        <img width="982" alt="seed" src="https://github.com/user-attachments/assets/a09e6930-6813-4fc7-9e34-c5e05c7cd102">
        

**We have already completed all the code, now we can start deploying our project!**

## Docker and SQL Server

### Basic Install

- install Docker:  [https://www.docker.com/](https://www.docker.com/)
- install image of sqlserver, open terminal, execute this command any path:
    
    ```bash
    docker pull [mcr.microsoft.com/mssql/server](http://mcr.microsoft.com/mssql/server)
    ```
    
- install Beekeeper: [https://www.beekeeperstudio.io/get](https://www.beekeeperstudio.io/get)

### Run SQL Server

run sqlserver in docker. open terminal, run following command: https://hub.docker.com/r/microsoft/mssql-server

```bash
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=YOUR_PASSWORD" -e "MSSQL_PID=Evaluation" -p 1433:1433  --name sqlpreview --hostname sqlpreview -d mcr.microsoft.com/mssql/server
```

- Default User Name is ‘**SA**’
- Set database password in **MSSQL_SA_PASSWORD**
- set database port: 1433

### Beekeeper connect database

1. make sure database is running!
2. create new connection, choose SQL Server

    <img width="512" alt="beekeeper1" src="https://github.com/user-attachments/assets/48034824-cb00-4633-9c9d-17334de11c5a">
    
3. input **User** and **Password**, check ‘**Trust Server Certificate**’, click Connect.

    <img width="529" alt="beekeeper2" src="https://github.com/user-attachments/assets/8a4d6cb4-1f37-455f-96fa-496a5c764c6e">
    
4. Correct connection:

    <img width="1312" alt="beekeeper3" src="https://github.com/user-attachments/assets/5b661041-bc3a-4b48-bc0e-d39fbf92c04a">
    
5. Create database

    <img width="923" alt="beekeeper4" src="https://github.com/user-attachments/assets/38c2a9b0-5fea-4184-9342-77f076eca79e">
    

## Build Project Image and Docker Compose

### Add Dockerfile and .dockerignore to Project

[Run an ASP.NET Core app in Docker containers](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/docker/building-net-docker-images?view=aspnetcore-8.0)

Dockerfile: It uses `dotnet publish` the same way you will do in this section to build and deploy.

```docker
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG TARGETARCH
WORKDIR /source

# Copy project file and restore as distinct layers
COPY --link ./*.csproj .
RUN dotnet restore -a $TARGETARCH

# Copy source code and publish app
COPY --link . .
RUN dotnet publish -c Release -a $TARGETARCH --no-restore -o /app

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0
EXPOSE 8080
WORKDIR /app
COPY --link --from=build /app .
USER $APP_UID

# Write your project name
ENTRYPOINT ["./TrilloBackend"]
```

.dockerignore

```docker
# directories
**/bin/
**/obj/
**/out/

# files
Dockerfile*
**/*.md
```

### Build project image

1. **Optional**: navigate to the project folder, run the **dotnet publish** command: (I have written this command in Dockerfile, we can **ignore** this step)
    
    ```bash
    dotnet publish -c Release -o published 
    ```

    <img width="717" alt="dockerfile" src="https://github.com/user-attachments/assets/05c1ce7d-12f7-4425-9f89-a9309ae54738">
    
2. Build Project image
    
    Attention there is a **point** in the end! I advice to name image as **dockercount/imagename:date.version** so that you can easy to manage different versions.
    
    ```bash
    docker build -t ella0110/trillobackend:20241005.1 . 
    ```
    
3. **Optional**: Run image. If your image doesn’t need database, then you can run it directly. If you need database, you should use docker compose which is in the next section.
    
    ```bash
    docker run -it --rm -p 8080:8080 ella0110/trillobackend:20241005.1
    ```
    

### Docker Compose

1. **Install docker compose**, open terminal, run these command: 
    
    ```bash
    brew search docker compose
    brew install docker-compose
    ```
    
2. **Define services in a Compose file**
    
    - Docker Compose simplifies the control of your entire application stack, making it easy to manage services, networks, and volumes in a single, comprehensible YAML configuration file. 
    
    - Create a file called `compose.yaml` in your project directory and paste the following:
      - Change **image** as your image name;
      - Change **ports**
      - Change **Environment Variable and healthcheck password** as your database password
    
    ```yaml
    services:
      web:
        image: "ella0110/trillobackend:20241005.3"
        ports:
          - "8080:8080"
        environment:
          - ASPNETCORE_ENVIRONMENT=Production
        depends_on:
          sqlserver: 
            condition: service_healthy
      sqlserver:
        image: "mcr.microsoft.com/mssql/server:latest"
        environment:
          - ACCEPT_EULA=Y
          - MSSQL_SA_PASSWORD=@Aa12345678
        ports:
          - "1433:1433"
        healthcheck:
          test: ["CMD", "/opt/mssql-tools18/bin/sqlcmd", "-C", "-S", "localhost", "-U", "sa", "-P", "@Aa12345678", "-Q", "SELECT 1", "-b", "-o", "/dev/null"]
          # test: /opt/mssql-tools18/bin/sqlcmd -C -S localhost -U sa -P ${MSSQL_SA_PASSWORD} -Q 'SELECT 1' -b -o /dev/null
          interval: 10s
          timeout: 3s
          retries: 10
          start_period: 10s
    ```
    
4. Run docker compose
    
    ```bash
    docker compose up
    ```
    

**Congratulation!!! We have already run docker compose, you need to attention these step:**
- if you only change yml, you can just delete container in docker and run `docker compose up` again!
- if you change other file like cs or Dockerfile, you need to run `docker build -t testdemo-backend .`(your project image name) and then delete container in docker and run `docker compose up`!

### Upload Image to Docker Hub

- open terminal, login to docker and run this command to push image to your docker hub account.

```bash
docker push ella0110/trillobackend
```

- if you want to change your image name, use this command:

[Stack Overflow](https://stackoverflow.com/questions/49834454/how-do-you-fix-a-docker-push-error-tag-does-not-exist)

```bash
docker tag trillo-backend ella0110/trillobackend
```

### Troubleshooting & Tips

- Tips
    1. Check the health_check log
        
        `trillobackend-sqlserver-1` is container name
        
        ```bash
        docker inspect --format "{{json .State.Health}}" trillobackend-sqlserver-1
        ```
        
- Troubleshooting
    1. Dependency issue: On first run, the database may not be ready when back end tries to initialise the database, and cause back end service failing to start.
        - Link: [Stack Overflow](https://stackoverflow.com/questions/60539114/how-to-wait-for-mssql-in-docker-compose)
        - Fix: Add **health check** and **depend on** command

        <img width="1131" alt="healthcheck" src="https://github.com/user-attachments/assets/2d33f45e-ec43-4093-a486-6cf849952a06">
        
    2. When check health, it’s always failed
        - should be `/opt/mssql-tools18/bin/sqlcmd` rather than `/opt/mssql-tools/bin/sqlcmd`, can test it in Docker/Containers/sqlserver/Exec
    3. Error: `[Microsoft][ODBC Driver 18 for SQL Server]SSL Provider: [error:1416F086:SSL routines:tls_process_server_certificate:certificate verify failed:self signed certificate]`
        - Link: [Stack Overflow](https://stackoverflow.com/questions/71688125/odbc-driver-18-for-sql-serverssl-provider-error1416f086)
        - Fix: Add `-C` to health check

## Deploy Project Image and Database to ECS

### Create Task definitions

1. login your AWS account
2. search **ECS**, choose **Elastic Container Service**
3. choose **Task definitions**, click **Create new task definition/ Create new task definition**
4. input **Task definition family name**
5. you can change **Task size** if you want
6. **Container-1**, you can input infomation about your database
    - **Name**: sqlserver;
    - **Image URI**: mcr.microsoft.com/mssql/server:latest
    - **Environment varaiables**: follow with the section: Run SQL  Server

        <img width="380" alt="envvariable" src="https://github.com/user-attachments/assets/a876dfdd-50d2-4961-a0b2-e66e2643d7f5">
        
    - **HealthCheck**: CMD-SHELL,/opt/mssql-tools18/bin/sqlcmd -C -S localhost -U sa -P 'your password' -Q 'SELECT 1' -b -o /dev/null
    - Interval: 10
    - Timeout: 3
    - Start period: 10
    - Retries: 10
7. Container-2, you can input information about your project image
    - **Name**: your project
    - **Image URI**: eg. ella0110/trillobackend:20241005.4
    - **Port mappings**

        <img width="765" alt="port" src="https://github.com/user-attachments/assets/c5b7ec87-56a8-4705-9e07-828f492ac2ec">
        
    - **Startup dependency ordering**

        <img width="755" alt="depend" src="https://github.com/user-attachments/assets/2cc04083-aa79-4099-bb1c-8011f843dd16">
        
8. click **Create**

### Create Clusters

Clusters → Create cluster: input **Cluster name**, click **Create**

<img width="1344" alt="cluster" src="https://github.com/user-attachments/assets/06818ff9-9ffe-43c9-83fb-a311ee86bfd5">

### Deploy your task to cluster

1. Choose your task, click **Deploy**→ **Create Service**

    <img width="1350" alt="deploytask" src="https://github.com/user-attachments/assets/1d5e93fd-22a0-4089-b405-49e5a1c6d262">
    
2. **Choose your Cluster**, then click **Create**

### Troubleshooting & Tips

- Tips
    1. The public IP generated by ECS is inaccessible because a specific IP was used, not open IP range.
        
        **Fix:** VPC → Security Groups → click Security group ID → Inbound rules → Edit inbound rules → add a new rule

        <img width="1296" alt="ip" src="https://github.com/user-attachments/assets/31e56304-48d1-4299-9d16-b4f14f36d197">
        
- Troubleshooting
    1. Error: `Unhandled **exception**. Microsoft.Data.SqlClient.SqlException (0x80131904): A network-related or instance-specific **error** occurred while establishing a connection to SQL Server. The server was not found or was not accessible. Verify that the instance name is correct and that SQL Server is configured to allow remote connections. (provider: TCP Provider, **error**: 35 - An internal **exception** was caught)`
        - Link: [Stack Overflow](https://stackoverflow.com/questions/58196930/communication-between-containers-in-ecs-task-definition)
        - Fix:

        <img width="702" alt="host" src="https://github.com/user-attachments/assets/cc2d7c57-30de-466e-9cfc-1579fbb54b9d">
        
    2. Healthcheck can not work
        - Link: [AWS - HealthCheck](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_HealthCheck.html)
        
        ```docker
        CMD-SHELL,/opt/mssql-tools18/bin/sqlcmd -C -S localhost -U sa -P '@Aa12345678' -Q 'SELECT 1' -b -o /dev/null
        ```

        ![healthcheckecs](https://github.com/user-attachments/assets/ecb735cb-35f9-47a6-bb75-c06768655eca)
