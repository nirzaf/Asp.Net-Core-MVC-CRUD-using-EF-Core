Asp.Net Core MVC CRUD with EF Core
Asp.Net Core Tutorial0 Comments
Asp.Net Core MVC CRUD Operations with EF Core.
60
SHARES
Share
Tweet
Linkedin
Google
Subscribe

 
In this article, we will implement CRUD Operations in Asp.Net Core MVC Application with EF Core Code First Approach.

GitHub link for this project : https://goo.gl/E13H2R.

Create Asp.Net Core MVC Project
In Visual Studio, Goto File > New > Project ( Ctrl + Shift + N). Then select Asp.Net Core Web Application.

Image Showing Creation of Asp.Net Core Web Application.

In template wizard, Select Web Application( MVC ) template. Make sure to select latest Asp.Net Core Version from top dropdown. We don’t need to configure this application with HTTPS (Unchecked corresponding option from bottom).

Picture showing creation MVC Web Application in .Net Core.
Setup Database for EF Core
For this application development, we will use EF Core – Code First Approach. First of all, we have to install NuGet Package for EFCore. for that you can right click on project in solution explorer, click on Manage NuGet Packages. In Browse tab, search for Microsoft.EntityFrameworkCore. Install the package with same version as that of Asp.Net Core.

We will create the DBContext class inside Models folder. To demonstrate Asp.Net Core CRUD Operation, we will deal with employee details like Full Name, Employee Code,Position and Office Location. So I have named the DbContext as EmployeeContext.

public class EmployeeContext:DbContext
{
    public EmployeeContext(DbContextOptions<EmployeeContext> options):base(options)
    {
    }

    public DbSet<Employee> Employees { get; set; }
}
1
2
3
4
5
6
7
8
public class EmployeeContext:DbContext
{
    public EmployeeContext(DbContextOptions<EmployeeContext> options):base(options)
    {
    }
 
    public DbSet<Employee> Employees { get; set; }
}
Inside the class, we have a DbSet property for Employees Collection of the type Employee class. So we have to define the class with required properties as follows.

public class Employee
{
    [Key]
    public int EmployeeId { get; set; }

    [Column(TypeName ="nvarchar(250)")]
    [Required(ErrorMessage ="This field is required.")]
    [DisplayName("Full Name")]
    public string FullName { get; set; }

    [Column(TypeName = "varchar(10)")]
    [DisplayName("Emp. Code")]
    public string EmpCode { get; set; }

    [Column(TypeName = "varchar(100)")]
    public string Position { get; set; }

    [Column(TypeName = "varchar(100)")]
    [DisplayName("Office Location")]
    public string OfficeLocation { get; set; }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
public class Employee
{
    [Key]
    public int EmployeeId { get; set; }
 
    [Column(TypeName ="nvarchar(250)")]
    [Required(ErrorMessage ="This field is required.")]
    [DisplayName("Full Name")]
    public string FullName { get; set; }
 
    [Column(TypeName = "varchar(10)")]
    [DisplayName("Emp. Code")]
    public string EmpCode { get; set; }
 
    [Column(TypeName = "varchar(100)")]
    public string Position { get; set; }
 
    [Column(TypeName = "varchar(100)")]
    [DisplayName("Office Location")]
    public string OfficeLocation { get; set; }
}
Configure the DbContext Class with Dependency Injection.
As per the DbContext class constructor parameter options of the type – DbContextOptions, we have to send DB Provider ( like for MSSQL Server / MySQL / PostgreSQL etc) and DB connection string. How can we do that ?

First of all, we will save the connection string in appsettings.json file as follows.

{
  ...,
  "ConnectionStrings": {
    "DevConnection": "Server=(local)\\sqlexpress;Database=EmployeeDB;Trusted_Connection=True;MultipleActiveResultSets=True;"
  }
}
1
2
3
4
5
6
{
  ...,
  "ConnectionStrings": {
    "DevConnection": "Server=(local)\\sqlexpress;Database=EmployeeDB;Trusted_Connection=True;MultipleActiveResultSets=True;"
  }
}
In order to pass both DB connection string and DB provider, we can use Dependency Injection from Asp.Net Core. For that we just need to update ConfigureServices method from Startup class. Invoke AddDbConext class from services collection as follows.


 
public void ConfigureServices(IServiceCollection services)
        {
            ...

            services.AddDbContext<EmployeeContext>(options => 
            options.UseSqlServer(Configuration.GetConnectionString("DevConnection")));
        }
1
2
3
4
5
6
7
public void ConfigureServices(IServiceCollection services)
        {
            ...
 
            services.AddDbContext<EmployeeContext>(options => 
            options.UseSqlServer(Configuration.GetConnectionString("DevConnection")));
        }
Initiate DB Migration
So far we have been modeling our Database, in-order to create the actual DB we have to initiate DB Migration. First of all open Package Manager Console. select project from Solution Explorer. Goto Tools > NuGet Package Manager > Package Manager Console.
In-order to create the Database from EF Model, you can execute following commands from Package Manager Console.

Add-Migration "InitialCreate"
Update-Database
1
2
Add-Migration "InitialCreate"
Update-Database
After executing these two commands there will be physical DB – EmployeeDB in SQL Server Instance, which is specified in connection string.

Create MVC Controller For CRUD Operations
We have to create a new MVC controller, So right click, Controller > Add > Controller. then select ‘MVC controller with views, using Entity Framework‘ template. Name the controller as EmployeeController. Select Employee class as model and EmployeeContext as DbContext class.

Image showing creation of MVC Controller in Asp.Net core for CRUD operations.
Created MVC controller will have all action methods for CRUD operations, even though I would like to arrange them as follows.

public class EmployeeController : Controller
    {
        private readonly EmployeeContext _context;

        public EmployeeController(EmployeeContext context)
        {
            _context = context;
        }

        // GET: Employee
        public async Task<IActionResult> Index()
        {
            return View(await _context.Employees.ToListAsync());
        }

        // GET: Employee/Create
        public IActionResult AddOrEdit(int id = 0)
        {
            if (id == 0)
                return View(new Employee());
            else
                return View(_context.Employees.Find(id));
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> AddOrEdit([Bind("EmployeeId,FullName,EmpCode,Position,OfficeLocation")] Employee employee)
        {
            if (ModelState.IsValid)
            {
                if (employee.EmployeeId == 0)
                    _context.Add(employee);
                else
                    _context.Update(employee);
                await _context.SaveChangesAsync();
                return RedirectToAction(nameof(Index));
            }
            return View(employee);
        }

        // GET: Employee/Delete/5
        public async Task<IActionResult> Delete(int? id)
        {
            var employee =await _context.Employees.FindAsync(id);
            _context.Employees.Remove(employee);
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Index));
        }
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
public class EmployeeController : Controller
    {
        private readonly EmployeeContext _context;
 
        public EmployeeController(EmployeeContext context)
        {
            _context = context;
        }
 
        // GET: Employee
        public async Task<IActionResult> Index()
        {
            return View(await _context.Employees.ToListAsync());
        }
 
        // GET: Employee/Create
        public IActionResult AddOrEdit(int id = 0)
        {
            if (id == 0)
                return View(new Employee());
            else
                return View(_context.Employees.Find(id));
        }
 
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> AddOrEdit([Bind("EmployeeId,FullName,EmpCode,Position,OfficeLocation")] Employee employee)
        {
            if (ModelState.IsValid)
            {
                if (employee.EmployeeId == 0)
                    _context.Add(employee);
                else
                    _context.Update(employee);
                await _context.SaveChangesAsync();
                return RedirectToAction(nameof(Index));
            }
            return View(employee);
        }
 
        // GET: Employee/Delete/5
        public async Task<IActionResult> Delete(int? id)
        {
            var employee =await _context.Employees.FindAsync(id);
            _context.Employees.Remove(employee);
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Index));
        }
    }
Inside this controller constructor, we have injected DbContext Instance from Startup Class. Index action method will fetch all employees from Employees table. Post MVC action method- AddOrEdit will do the Insert and Update Operation. Delete Operation can be handled inside Get action method Delete.

Now let’s add corresponding views. all employees retrived using index action method will be listed in index view file inside HTML table.

@model IEnumerable<Asp.netCoreMVCCRUD.Models.Employee>

@{
    ViewData["Title"] = "Index";
}

<h4>Employee Register</h4>
<hr />


<table class="table table-hover">
    <thead>
        <tr>
            <th>
                @Html.DisplayNameFor(model => model.FullName)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.EmpCode)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Position)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.OfficeLocation)
            </th>
            <th>
                <a asp-action="AddOrEdit" class="btn btn-outline-success"><i class="far fa-plus-square"></i> Employee</a>
            </th>
        </tr>
    </thead>
    <tbody>
@foreach (var item in Model) {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.FullName)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.EmpCode)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.Position)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.OfficeLocation)
            </td>
            <td>
                <a asp-action="AddOrEdit" asp-route-id="@item.EmployeeId"><i class="fa fa-marker fa-lg"></i></a>
                <a asp-action="Delete" asp-route-id="@item.EmployeeId" class="text-danger ml-1" onclick="return confirm('Are you sure to delete this record?')"><i class="fa fa-trash-alt fa-lg"></i></a>
            </td>
        </tr>
}
    </tbody>
</table>
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
@model IEnumerable<Asp.netCoreMVCCRUD.Models.Employee>
 
@{
    ViewData["Title"] = "Index";
}
 
<h4>Employee Register</h4>
<hr />
 
 
<table class="table table-hover">
    <thead>
        <tr>
            <th>
                @Html.DisplayNameFor(model => model.FullName)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.EmpCode)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Position)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.OfficeLocation)
            </th>
            <th>
                <a asp-action="AddOrEdit" class="btn btn-outline-success"><i class="far fa-plus-square"></i> Employee</a>
            </th>
        </tr>
    </thead>
    <tbody>
@foreach (var item in Model) {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.FullName)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.EmpCode)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.Position)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.OfficeLocation)
            </td>
            <td>
                <a asp-action="AddOrEdit" asp-route-id="@item.EmployeeId"><i class="fa fa-marker fa-lg"></i></a>
                <a asp-action="Delete" asp-route-id="@item.EmployeeId" class="text-danger ml-1" onclick="return confirm('Are you sure to delete this record?')"><i class="fa fa-trash-alt fa-lg"></i></a>
            </td>
        </tr>
}
    </tbody>
</table>
It will look like this.

Image Showing List of Employee which retrieved from SQL Server Database.
The navbar and surrounding HTML body and header part is designed in Shared/_Layout.cshtml as follows.

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Asp.netCoreMVCCRUD</title>

    <environment include="Development">
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
    </environment>
    <environment exclude="Development">
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css"
              asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
              asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute"
              crossorigin="anonymous"
              integrity="sha256-eSi1q2PG6J7g7ib17yAaWMcrr5GrtohYChqibrV7PBE=" />
    </environment>
    <link rel="stylesheet" href="~/css/site.css" />
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.6.3/css/all.css" integrity="sha384-UHRtZLI+pbxtHCWp1t77Bi1L4ZtiqrqD80Kn4Z8NTSRyMA2Fd33n5dQ8lWUE00s/" crossorigin="anonymous">
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">Asp.netCoreMVCCRUD</a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex flex-sm-row-reverse">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">Home</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="container">
        <partial name="_CookieConsentPartial" />
        <main role="main" class="pb-3">
            @RenderBody()
        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2018 - Asp.netCoreMVCCRUD - <a asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
        </div>
    </footer>

    <environment include="Development">
        <script src="~/lib/jquery/dist/jquery.js"></script>
        <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.js"></script>
    </environment>
    <environment exclude="Development">
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"
                asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
                asp-fallback-test="window.jQuery"
                crossorigin="anonymous"
                integrity="sha256-FgpCb/KJQlLNfOu91ta32o/NMZxltwRo8QtmkMRdAu8=">
        </script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/js/bootstrap.bundle.min.js"
                asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"
                asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal"
                crossorigin="anonymous"
                integrity="sha256-E/V4cWE4qvAeO5MOhjtGtqDzPndRO1LBk8lJ/PR7CA4=">
        </script>
    </environment>
    <script src="~/js/site.js" asp-append-version="true"></script>

    @RenderSection("Scripts", required: false)
</body>
</html>
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Asp.netCoreMVCCRUD</title>
 
    <environment include="Development">
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
    </environment>
    <environment exclude="Development">
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/css/bootstrap.min.css"
              asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
              asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute"
              crossorigin="anonymous"
              integrity="sha256-eSi1q2PG6J7g7ib17yAaWMcrr5GrtohYChqibrV7PBE=" />
    </environment>
    <link rel="stylesheet" href="~/css/site.css" />
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.6.3/css/all.css" integrity="sha384-UHRtZLI+pbxtHCWp1t77Bi1L4ZtiqrqD80Kn4Z8NTSRyMA2Fd33n5dQ8lWUE00s/" crossorigin="anonymous">
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">Asp.netCoreMVCCRUD</a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex flex-sm-row-reverse">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">Home</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="container">
        <partial name="_CookieConsentPartial" />
        <main role="main" class="pb-3">
            @RenderBody()
        </main>
    </div>
 
    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2018 - Asp.netCoreMVCCRUD - <a asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
        </div>
    </footer>
 
    <environment include="Development">
        <script src="~/lib/jquery/dist/jquery.js"></script>
        <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.js"></script>
    </environment>
    <environment exclude="Development">
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"
                asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
                asp-fallback-test="window.jQuery"
                crossorigin="anonymous"
                integrity="sha256-FgpCb/KJQlLNfOu91ta32o/NMZxltwRo8QtmkMRdAu8=">
        </script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.1.3/js/bootstrap.bundle.min.js"
                asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"
                asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal"
                crossorigin="anonymous"
                integrity="sha256-E/V4cWE4qvAeO5MOhjtGtqDzPndRO1LBk8lJ/PR7CA4=">
        </script>
    </environment>
    <script src="~/js/site.js" asp-append-version="true"></script>
 
    @RenderSection("Scripts", required: false)
</body>
</html>
We have designed a form for Add or Edit Operation in AddOrEdit.cshml file.

@model Asp.netCoreMVCCRUD.Models.Employee

@{
    ViewData["Title"] = "Create";
}

<h4>Employee Form</h4>
<hr />
<div class="row">
    <div class="col-md-6">
        <form asp-action="AddOrEdit">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <input type="hidden" asp-for="EmployeeId" />
            <div class="form-group">
                <label asp-for="FullName" class="control-label"></label>
                <input asp-for="FullName" class="form-control" />
                <span asp-validation-for="FullName" class="text-danger"></span>
            </div>
            <div class="form-row">
                <div class="form-group col-md-6">
                    <label asp-for="EmpCode" class="control-label"></label>
                    <input asp-for="EmpCode" class="form-control" />
                    <span asp-validation-for="EmpCode" class="text-danger"></span>
                </div>
                <div class="form-group col-md-6">
                    <label asp-for="Position" class="control-label"></label>
                    <input asp-for="Position" class="form-control" />
                    <span asp-validation-for="Position" class="text-danger"></span>
                </div>
            </div>
            <div class="form-group">
                <label asp-for="OfficeLocation" class="control-label"></label>
                <input asp-for="OfficeLocation" class="form-control" />
                <span asp-validation-for="OfficeLocation" class="text-danger"></span>
            </div>
            <div class="form-row">
                <div class="form-group col-md-6">
                    <input type="submit" value="Submit" class="btn btn-primary btn-block" />
                </div>
                <div  class="form-group col-md-6">
                    <a asp-action="Index"  class="btn btn-secondary btn-block"><i class="fa fa-table"></i> Back to List</a>
                </div>
            </div>
        </form>
    </div>
</div>



@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
@model Asp.netCoreMVCCRUD.Models.Employee
 
@{
    ViewData["Title"] = "Create";
}
 
<h4>Employee Form</h4>
<hr />
<div class="row">
    <div class="col-md-6">
        <form asp-action="AddOrEdit">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <input type="hidden" asp-for="EmployeeId" />
            <div class="form-group">
                <label asp-for="FullName" class="control-label"></label>
                <input asp-for="FullName" class="form-control" />
                <span asp-validation-for="FullName" class="text-danger"></span>
            </div>
            <div class="form-row">
                <div class="form-group col-md-6">
                    <label asp-for="EmpCode" class="control-label"></label>
                    <input asp-for="EmpCode" class="form-control" />
                    <span asp-validation-for="EmpCode" class="text-danger"></span>
                </div>
                <div class="form-group col-md-6">
                    <label asp-for="Position" class="control-label"></label>
                    <input asp-for="Position" class="form-control" />
                    <span asp-validation-for="Position" class="text-danger"></span>
                </div>
            </div>
            <div class="form-group">
                <label asp-for="OfficeLocation" class="control-label"></label>
                <input asp-for="OfficeLocation" class="form-control" />
                <span asp-validation-for="OfficeLocation" class="text-danger"></span>
            </div>
            <div class="form-row">
                <div class="form-group col-md-6">
                    <input type="submit" value="Submit" class="btn btn-primary btn-block" />
                </div>
                <div  class="form-group col-md-6">
                    <a asp-action="Index"  class="btn btn-secondary btn-block"><i class="fa fa-table"></i> Back to List</a>
                </div>
            </div>
        </form>
    </div>
</div>
 
 
 
@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}
CRUD Operation form look like this.


 
Image showing design of form for insert and update operation.
