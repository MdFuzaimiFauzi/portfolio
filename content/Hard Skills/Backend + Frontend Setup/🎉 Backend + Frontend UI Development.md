---
tags:
  - userinterface
  - frontend
  - backend
  - API
  - json
  - apicontroller
  - cmd
  - command
  - controller
  - CRUD
  - dotnet
  - dotnet9
  - mudblazor
  - server
---
> [!note]
> This procedures are undergo with **.Net 9.0.2** and **MudBlazor 8.4.0** and **Visual Studio Community 2022 Preview 17.14.0 Preview 2.0**
>

tested
# 1️⃣ Create Necessary Files and Solution

### 1️⃣ **Create New Solution Folder**

```
mkdir QueueUpSolution
cd QueueUpSolution
```

### 2️⃣ **Create Solution File**
```
dotnet new sln -n QueueUpSolution
```

### 3️⃣ **Create API Project**

```
dotnet new webapi -n QueueUpAPI
```

Then, add the project to the solution:

```
dotnet sln add QueueUpAPI/QueueUpAPI.csproj
```

### 4️⃣ **Create MudBlazor Server UI Project**

```
dotnet new mudblazor --interactivity Server --name MyApplication --all-interactive
```

Then, add to the solution :

```
dotnet sln add QueueUpUI/QueueUpUI.csproj
```

---

# 2️⃣ Setup the Backend 

### 1️⃣ **Create `QueueEntry.cs` in `Models` folder :**

```
namespace QueueUpAPI.Models
{
	public class QueueEntry
	{
		public int Id {get;set;}
		public int QueueNumber {get;set;}
		public string? Name {get;set;} = string.Empty;
		public DateTime CreatedAt {get;set;}
		public bool IsCancelled {get;set;}
	}
}

```

### 2️⃣ **Add `QueueController.cs` to `Controllers` folders:**


```
using Microsoft.AspNetCore.Mvc;
using QueueAPI.Models;

namespace QueueUpAPI.Controllers
{
	[ApiController]
	[Route("api/[controller]")]
	public class QueueController : ControllerBase
	{
		private static readonly List<QueueEntry> _queueList = new();
	
		[HttpGet]
		public IActionResult GetAll()
		{
			return Ok(_queueList);
		}

		[HttpPost]
		public IActionResult Create([FromBody] string name)
		{
			var entry = new QueueEntry
			{
				Id = _queueList.Count + 1,
				QueueNumber = _queueList.Count +1,
				Name = name,
				CreatedAt = DateTime.Now,
				IsCancelled = false
				
			};	
			_queueList.Add(entry);
			return Ok(entry);	
		}
		
		[HttpPut("{id}/cancel")]
		public IActionResult CancelQueue(int id)
		{
			var entry = _queueList.FirstOrDefault(q => q.Id === id);
			if (entry == null)
				return NotFound(new {Message = $"Queue with ID {id} not found"});

			entry.IsCancelled = true;
			return Ok(entry);
		}
	}	
}

```

### 3️⃣ **Update `Program.cs` in `QueueUpAPI`**

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthorization();

//Configure HTTP request pipeline
if (app.Environment.IsDevelopment())
{
	app.MapOpenApi();
}
app.UseHttpsRedirection();

//added
app.MapsController();

app.Run();

```

---

# 3️⃣ **Setup the Frontend**


### 1️⃣ **Register `HttpClient` in `QueueUpUI/Program.cs`** :

```
builder.Services.AddScoped(sp => new HttpClient
{
	BaseAddress = new Uri("http://localhost:5158")
});

```

### 2️⃣ **Create a model `QueueEntry.cs` in `QueueUpUI/Models`:

```
namespace QueueUpUI.Models
{

	public class QueueEntry
	{
		public int Id {get;set;}
		public int QueueNumber {get;set;}
		public string? Name {get;set;} = string.Empty;
		public DateTime CreatedAt {get;set;}
		public bool IsCancelled {get;set;}

	}
}
```

### 3️⃣ **Create a page `Pages/Queue.razor`** :

> [!tip]
> You can also use existing page to add the Queue stuff


```
@page "/queue"
@using QueueUpUI.Models

@inject HttpClient Http

<h3>Queue List</h3>

@if (queueEntries == null)
{
    <p><em>Loading...</em></p>
}
else
{
    @foreach (var entry in queueEntries)
    {
        <p>Queue Number: @entry.QueueNumber Name: @entry.Name Created: @entry.CreatedAt</p>
        
    }
}

@code {
    List<QueueEntry>? queueEntries;

    protected override async Task OnInitializedAsync()
    {
        queueEntries = await Http.GetFromJsonAsync<List<QueueEntry>>("api/queue");
    }

```


### 4️⃣ **Add QueueUI.Models into `_Imports.razor`**

```
//full version

@using System.Net.Http
@using System.Net.Http.Json
@using Microsoft.AspNetCore.Components.Forms
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.Web
@using static Microsoft.AspNetCore.Components.Web.RenderMode
@using Microsoft.AspNetCore.Components.Web.Virtualization
@using Microsoft.JSInterop
@using MudBlazor
@using MudBlazor.Services
@using QueueUpUI
@using QueueUpUI.Components
@using QueueUpUI.Models

```

### 5️⃣ **Add QueueUpAPI into the project**

- **Right Click** (QueueUpUI) ➡️ **Add Project Reference** ➡️ `Project` tab ➡️ Tick `QueueUpAPI`

 ![[rightclick_addprojectref.png]]

![[tick_relatedproject.png]]


### 6️⃣ **Run both UI and API simultaneously**

1. **Right click on `QueueUpSolution`** (solution level) ➡️ **Set Startup Projects**
2. Choose **Multiple startup projects**
3. Set `QueueUpAPI` ➡️ Start
4. Set `QueueUpUI` ➡️ Start
5. Click Ok
6. Try to `Build` and `Run`

---

# 🚩 **Troubleshoot**

## 🔦 **Test API manually**

1. Open up your default browser
2. Navigate to `http://localhost:5158/api/queue`

Should be printing out this JSON format :

```
[
  {
    "id": 1,
    "queueNumber": 1,
    "name": "Test Queue",
    "createdAt": "2025-03-25T12:00:00"
  }
]
```

---
## 🔦 **Change to the correct API Port in UI**

- Open up `Program.cs` in project `QueueUpUI`
- Then, find this line
```
builder.Services.AddScoped(sp => new HttpClient)
{
	BaseAddress = new Uri("http://localhost:5062/")
}
```

---
## 🔦**Other Options: Fix the Fixed Port**

- If you want to make the port as fixed, open up :
`QueueUpAPI\Properties\launchSettings.json`

- Find and replace :
```
"applicationUrl" : "http://localhost:5158"
```

---
## 🔦 **What to do if the UI is empty or not showing any data?**

- Navigate to the backend (`QueueUpAPI`) and check the controller `QueueController.cs`:

```
[ApiController]
[Route("api/[controller]")]
public class QueueController : ControllerBase
{
    [HttpGet]
    public IEnumerable<QueueEntry> Get()
    {
        return new List<QueueEntry>
        {
            new QueueEntry { QueueNumber = 1, Name = "Test Queue", CreatedAt = DateTime.Now }
        };
    }
}

```


---
# 🎉 **Attaching our Application to the database on PostgreSQL server**

## 1️⃣ **Install PostgreSQL**

1. Download PostgreSQL from *https://www.posgresql.org/download
2. Set username (default: `posgres`) dan your own password.
3. Default port: `5432`

## 2️⃣ **Create Database**

1. Open up **pgAdmin**
2. Right-click on `Databases` ➡️ `Create` ➡️ `Database`
3. Name : `QueueUpDB`

> [!note]
> **Recommendation**
> Create a database name that is similar to your application name for easy reference.

## 3️⃣ **Add Required NuGet Packages**

1. Install these packages into `QueueUPAPI` **(API)**
```
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL.Design

``` 

## 4️⃣ **Setup `QueueEntry` Model**

File : `Models/QueueEntry.cs`

```
namespace QueueUpAPI.Models
{
    public class QueueEntry
    {
        public int Id { get; set; }
        public int QueueNumber { get; set; }
        public string? Name {get; set; }
        public DateTime CreatedAt { get; set; }
        public bool IsCancelled { get; set; }
    }
}

```

## 5️⃣ **Setup `QueueUpDbContext`**

File : `QueueUpDbContext.cs`

```
using Microsoft.EntityFrameworkCore;
using QueueUpAPI.Models;

public class QueueUpDbContext : DbContext
{
    public QueueUpDbContext(DbContextOptions<QueueUpDbContext> options) : base(options) { }

    public DbSet<QueueEntry> QueueEntries { get; set; }
}

```

## 6️⃣ **Configure** `appsettings.json` **(API)**

```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=QueueUpDB;Username=postgrestest;Password=pass;SSL Mode=Disable;Trust Server Certificate=true"
  }
}
```

## 7️⃣ **Update `Program.cs`** **(QueueUpAPI)**

```
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring OpenAPI at https://aka.ms/aspnet/openapi
builder.Services.AddOpenApi();

//added
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();

builder.Services.AddDbContext<QueueUpDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthorization();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
}

app.UseHttpsRedirection();

//added
app.MapControllers();

app.Run();

```


## 8️⃣ **Create / Update `QueueController.cs

> [!info]
> All CRUD methods are categorized, the full code is attached at the bottom of this steps.


**Constructor**
```
 public class QueueController : ControllerBase
 {
     //private static readonly List<QueueEntry> _queueList = new();

     private readonly QueueUpDbContext _context;

     public QueueController(QueueUpDbContext context)
     {
         _context = context;
     }

```

**[HttpGet]**

```
  [HttpGet]
  //public IEnumerable<QueueEntry> Get()
  //{
  //    return _queueList;
  //}
 public async Task<ActionResult<IEnumerable<QueueEntry>>> Get()
  {
      var entries = await _context.QueueEntries.OrderBy(q => q.QueueNumber).ToListAsync();

      var malaysiaTimeZone = TimeZoneInfo.FindSystemTimeZoneById("Singapore");
      foreach (var entry in entries)
      {
          entry.CreatedAt = TimeZoneInfo.ConvertTimeFromUtc(entry.CreatedAt, malaysiaTimeZone);
      }

      return entries;
  }
```

**[HttpPost]**

```
 [HttpPost]
 //public IActionResult Post([FromBody] QueueEntry entry)
 //{

 //    entry.Id = _queueList.Count + 1;
 //    entry.QueueNumber = _queueList.Count + 1;
 //    entry.CreatedAt = DateTime.Now;
 //    entry.IsCancelled = false;
     
 //    _queueList.Add(entry);
 //    return Ok(entry);
 //}
 public async Task<ActionResult<QueueEntry>> Post([FromBody] QueueEntry entry)
 {
     int nextNumber = await _context.QueueEntries.CountAsync() + 1;

     entry.QueueNumber = nextNumber;
     entry.CreatedAt = DateTime.UtcNow;
     entry.IsCancelled = false;

     _context.QueueEntries.Add(entry);
     await _context.SaveChangesAsync();

     var malaysiaTimeZone = TimeZoneInfo.FindSystemTimeZoneById("Singapore");
     entry.CreatedAt = TimeZoneInfo.ConvertTimeFromUtc(entry.CreatedAt, malaysiaTimeZone);

     return Ok(entry);
 }
```

**[HttpPut]**

```
  [HttpPut("{id}/cancel")]
  //public IActionResult CancelQueue(int id)
  //{
  //    var entry = _queueList.FirstOrDefault(q => q.Id == id);
  //    if (entry == null)
  //        return NotFound(new { Message = $"Queue with ID {id} not found" });

  //    entry.IsCancelled = true;
  //    return Ok(entry);
  //}
  public async Task<IActionResult> CancelQueue(int id)
  {
      var entry = await _context.QueueEntries.FindAsync(id);
      if (entry == null)
          return NotFound(new { Message = $"Queue with Id {id} not found" });

      entry.IsCancelled = true;
      await _context.SaveChangesAsync();

      return Ok(entry);
  }
```

**[HttpDelete]**

```
[HttpDelete("{id}")]
//public IActionResult DeleteQueue(int id)
//{
//    var entry = _queueList.FirstOrDefault(q => q.Id == id);
//    if (entry == null) return NotFound(new { Message = $"Queue with ID {id} not found" });

//    _queueList.Remove(entry);

//    for (int i = 0; i < _queueList.Count; i++)
//    {
//        _queueList[i].QueueNumber = i + 1;
//    }

//    return Ok(new { Message = $"Queue with ID {id} deleted successfully" });
//}
public async Task<IActionResult> DeleteQueue(int id)
{
    var entry = await _context.QueueEntries.FindAsync(id);
    if (entry == null)
        return NotFound(new { Message = $"Queue with ID {id} not found" });

    _context.QueueEntries.Remove(entry);
    await _context.SaveChangesAsync();

    var remaining = await _context.QueueEntries
                        .OrderBy(q => q.QueueNumber)
                        .ToListAsync();

    for (int i = 0; i < remaining.Count; i++)
    {
        remaining[i].QueueNumber = i + 1;

    }

    await _context.SaveChangesAsync();
    return Ok(new { Message = $"Queue with ID {id} is removed and queued re-numbered" });
}
```

**Full Code of QueueController**

```
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using QueueUpAPI.Models;

namespace QueueUpAPI.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class QueueController : ControllerBase
    {
        //private static readonly List<QueueEntry> _queueList = new();

        private readonly QueueUpDbContext _context;

        public QueueController(QueueUpDbContext context)
        {
            _context = context;
        }

        [HttpGet]
        //public IEnumerable<QueueEntry> Get()
        //{
        //    return _queueList;
        //}
       public async Task<ActionResult<IEnumerable<QueueEntry>>> Get()
        {
            var entries = await _context.QueueEntries.OrderBy(q => q.QueueNumber).ToListAsync();

            var malaysiaTimeZone = TimeZoneInfo.FindSystemTimeZoneById("Singapore");
            foreach (var entry in entries)
            {
                entry.CreatedAt = TimeZoneInfo.ConvertTimeFromUtc(entry.CreatedAt, malaysiaTimeZone);
            }

            return entries;
        }

        [HttpPost]
        //public IActionResult Post([FromBody] QueueEntry entry)
        //{

        //    entry.Id = _queueList.Count + 1;
        //    entry.QueueNumber = _queueList.Count + 1;
        //    entry.CreatedAt = DateTime.Now;
        //    entry.IsCancelled = false;
            
        //    _queueList.Add(entry);
        //    return Ok(entry);
        //}
        public async Task<ActionResult<QueueEntry>> Post([FromBody] QueueEntry entry)
        {
            int nextNumber = await _context.QueueEntries.CountAsync() + 1;

            entry.QueueNumber = nextNumber;
            entry.CreatedAt = DateTime.UtcNow;
            entry.IsCancelled = false;

            _context.QueueEntries.Add(entry);
            await _context.SaveChangesAsync();

            var malaysiaTimeZone = TimeZoneInfo.FindSystemTimeZoneById("Singapore");
            entry.CreatedAt = TimeZoneInfo.ConvertTimeFromUtc(entry.CreatedAt, malaysiaTimeZone);

            return Ok(entry);
        }

        [HttpPut("{id}/cancel")]
        //public IActionResult CancelQueue(int id)
        //{
        //    var entry = _queueList.FirstOrDefault(q => q.Id == id);
        //    if (entry == null)
        //        return NotFound(new { Message = $"Queue with ID {id} not found" });

        //    entry.IsCancelled = true;
        //    return Ok(entry);
        //}
        public async Task<IActionResult> CancelQueue(int id)
        {
            var entry = await _context.QueueEntries.FindAsync(id);
            if (entry == null)
                return NotFound(new { Message = $"Queue with Id {id} not found" });

            entry.IsCancelled = true;
            await _context.SaveChangesAsync();

            return Ok(entry);
        }


        [HttpDelete("{id}")]
        //public IActionResult DeleteQueue(int id)
        //{
        //    var entry = _queueList.FirstOrDefault(q => q.Id == id);
        //    if (entry == null) return NotFound(new { Message = $"Queue with ID {id} not found" });

        //    _queueList.Remove(entry);

        //    for (int i = 0; i < _queueList.Count; i++)
        //    {
        //        _queueList[i].QueueNumber = i + 1;
        //    }

        //    return Ok(new { Message = $"Queue with ID {id} deleted successfully" });
        //}
        public async Task<IActionResult> DeleteQueue(int id)
        {
            var entry = await _context.QueueEntries.FindAsync(id);
            if (entry == null)
                return NotFound(new { Message = $"Queue with ID {id} not found" });

            _context.QueueEntries.Remove(entry);
            await _context.SaveChangesAsync();

            var remaining = await _context.QueueEntries
                                .OrderBy(q => q.QueueNumber)
                                .ToListAsync();

            for (int i = 0; i < remaining.Count; i++)
            {
                remaining[i].QueueNumber = i + 1;

            }

            await _context.SaveChangesAsync();
            return Ok(new { Message = $"Queue with ID {id} is removed and queued re-numbered" });
        }

    }
}

```

## 9️⃣ **Run Migration and Update Database**

> [!note]
> Run on **API** directory
>

```
dotnet ef migrations add Init
dotnet ef database update
```


---
# 🔦 **Troubleshoot**

> [!warning]
> Web (API) Port **cannot be similar with** PostgreSQL Port !

| Component        | Purposes                | Port               |
| ---------------- | ----------------------- | ------------------ |
| Web API (dotNet) | Handle browser/API call | Normally `5000+`   |
| UI (Blazor)      | Serve UI to browser     | Normally `7000+`   |
| PostgreSQL       | Handle DB queries       | **5432 (default)** |

**What happened when using the SAME PORT**

If **BOTH** process tried to use the **same port**, on of the process will start with this error:
```
Address already in use
```

Or it is more dangerous, if we are accidentally insert like this:
```
"Host=localhost;Port=5062;"
```

In the PostgreSQL connection string, it tries to connect to **Web API**, not to the **database**. Then, this cause to this error :
```
Npgsql.NpgsqlException: Received unknown response H for SSLRequest (expecting S or N)

```


