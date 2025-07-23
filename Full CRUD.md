# Establishing SQL Connection & Displaying Data in ASP.NET Core MVC

**Prerequisite**: Ensure SQL Server is installed and the `SelectAll` stored procedure is created.

```sql
CREATE PROCEDURE sp_GetAllEmployees
AS
BEGIN
    SET NOCOUNT ON;
    SELECT
        EmployeeId,
        FirstName,
        LastName,
        Email,
        PhoneNumber,
        DateOfBirth,
        Gender,
        HireDate,
        JobTitle,
        Department,
        Salary,
        IsActive,
        CreatedAt,
        UpdatedAt
    FROM Employee
    WHERE IsActive = 1;
END
GO
```

## Step 1: Add Static Data to SQL Server

Insert static data into your SQL Server database

## Step 2: Configure the Connection String in `appsettings.json`

### For Windows Users:

Add the connection string for your SQL Server database in the `appsettings.json` file as shown below:

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=SQL Server Name;Initial Catalog=DatabaseName;Integrated Security=true;"
}
```

**Example:**

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=LAPTOP-LBMAFD6U\\SQLEXPRESS;Initial Catalog=StudentMaster;Integrated Security=true;"
}
```

### For Mac Users:

For Mac users, the connection string should include the user ID and password:

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=SQL Server Name;Initial Catalog=DatabaseName;User id=userID; password=Password;"
}
```

**Example:**

```json
"ConnectionStrings": {
    "ConnectionString": "Data Source=localhost;Initial Catalog=Practice;User id=SA; password=MyStrongPass123;"
}
```

## Step 3: Set Up the Configuration Variable in the Controller

In your controller, declare a configuration variable and initialize it using the constructor. This will allow you to access the connection string from the configuration.

```csharp
 private readonly IConfiguration configuration;

 public EmployeeController(IConfiguration configuration)
 {
     this.configuration = configuration;
 }
```

## Step 4: Install the `System.Data.SqlClient` Package

Install the `System.Data.SqlClient` package from the NuGet Package Manager to enable SQL Server connectivity.

- Navigate to **Tools** -> **NuGet Package Manager** -> **Manage NuGet Packages for Solution**.
- Ensure the project is not running during the installation.

## Step 5: Write Logic to Fetch Data in the List Page Action Method

In the action method for your list page, add the following logic to fetch data from the SQL Server database:

```csharp
 public IActionResult EmployeeList()
 {
     DataTable dataTable = new DataTable();

     string connectionString = configuration.GetConnectionString("ConnectionString");
     using (SqlConnection connection = new SqlConnection(connectionString))
     {
         connection.Open();

         SqlCommand command = connection.CreateCommand();
         command.CommandType = CommandType.StoredProcedure;
         command.CommandText = "sp_GetAllEmployees";

         SqlDataReader reader = command.ExecuteReader();

         dataTable.Load(reader);

         reader.Close();
     }

     return View(dataTable);
 }
```

## Step 6: Display the Data in the View

In your view page, import the `DataTable` and use a `foreach` loop to iterate through the data and display it:

```csharp
@model DataTable;
@using System.Data;

<main id="main" class="main">
    <div class="pagetitle"></div><!-- End Page Title -->

    <section class="section">
        <div class="row">
            <div class="col-12">
                <div class="card">
                    <div class="card-body">
                        <div class="row mb-4">
                            <div class="col-9">
                                <h5 class="card-title text-center">Employee Table</h5>
                            </div>
                            <div class="col text-end">
                                <a asp-controller="Employee" asp-action="EmployeeAddEdit" class="btn btn-primary mt-3">
                                    <i class="bi bi-plus-circle"></i> Add Employee
                                </a>
                            </div>
                        </div>

                        @if (@TempData["ErrorMessage"] != null)
                        {
                            <div class="alert alert-danger alert-dismissible fade show" role="alert">
                                <h5 class="text-danger">@TempData["ErrorMessage"]</h5>
                                <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="close"></button>
                            </div>
                        }

                        @if (@TempData["SuccessMessage"] != null)
                        {
                            <div class="alert alert-success alert-dismissible fade show" role="alert">
                                <h5 class="text-success">@TempData["SuccessMessage"]</h5>
                                <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="close"></button>
                            </div>
                        }

                        <!-- Employee Table -->
                        <table class="table table-hover">
                            <thead class="table-dark">
                                <tr>
                                    <th class="text-center">#</th>
                                    <th class="text-center">Full Name</th>
                                    <th class="text-center">Email</th>
                                    <th class="text-center">Phone</th>
                                    <th class="text-center">Hire Date</th>
                                    <th class="text-center">Department</th>
                                    <th class="text-center">Job Title</th>
                                    <th class="text-end">Salary</th>
                                    <th class="text-center">Update</th>
                                    <th class="text-center">Delete</th>
                                </tr>
                            </thead>
                            <tbody>
                                @{
                                    foreach (DataRow dataRow in Model.Rows)
                                    {
                                        <tr>
                                            <td class="border-2 text-center">@dataRow["FirstName"] @dataRow["LastName"]</td>
                                            <td class="border-2 text-center">@dataRow["Email"]</td>
                                            <td class="border-2 text-center">@dataRow["PhoneNumber"]</td>
                                            <td class="border-2 text-center">@Convert.ToDateTime(dataRow["HireDate"]).ToString("yyyy-MM-dd")</td>
                                            <td class="border-2 text-center">@dataRow["Department"]</td>
                                            <td class="border-2 text-center">@dataRow["JobTitle"]</td>
                                            <td class="border-2 text-end">@dataRow["Salary"]</td>
                                            <td class="border-2 text-center">
                                                <a asp-controller="Employee" asp-action="EmployeeAddEdit" asp-route-EmployeeId="@dataRow["EmployeeId"]" class="btn btn-warning">
                                                    <i class="bi bi-pencil-square">update</i>
                                                </a>
                                            </td>
                                            <td class="border-2 text-center">
                                                <form method="post" asp-controller="Employee" asp-action="EmployeeDelete">
                                                    <input type="hidden" name="EmployeeId" value="@dataRow["EmployeeId"]" />
                                                    <button type="submit" class="btn btn-danger" onclick="return confirmDelete()">
                                                        <i class="bi bi-trash">delete</i>
                                                    </button>
                                                </form>
                                            </td>
                                        </tr>
                                    }
                                }
                            </tbody>
                        </table>
                        <!-- End Table -->
                    </div>
                </div>

            </div>
        </div>
    </section>
</main><!-- End #main -->

<script type="text/javascript">
    function confirmDelete() {
        return confirm("Are you sure you want to delete this employee?");
    }
</script>

```


# Deleting a Data Entry in ASP.NET Core MVC

**Prerequisite**: Ensure the Delete Procedure is set up in your database.

### Delete Procedure

Below is the SQL stored procedure for deleting a product from the database:

```sql
CREATE PROCEDURE sp_DeleteEmployee
    @EmployeeId INT
AS
BEGIN
    SET NOCOUNT ON;
    DELETE FROM Employee
    WHERE EmployeeId = @EmployeeId;
END
GO
```

## Step 1: Implement the `CityDelete` Action Method in the Controller

Add the following code in your controller to handle the deletion of a product. This method connects to the database, executes the delete procedure, and then redirects to the product list page.

```csharp
public IActionResult EmployeeDelete(int EmployeeId)
{
    try
    {
        string connectionString = configuration.GetConnectionString("ConnectionString");
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            connection.Open();

            SqlCommand command = connection.CreateCommand();
            command.CommandType = CommandType.StoredProcedure;
            command.CommandText = "sp_DeleteEmployee";
            command.Parameters.AddWithValue("@EmployeeId", EmployeeId);

            command.ExecuteNonQuery();
        }

        TempData["SuccessMessage"] = "Employee deleted successfully.";
        return RedirectToAction("EmployeeList");
    }
    catch (Exception ex)
    {
        TempData["ErrorMessage"] = "An error occurred while deleting the employee: " + ex.Message;
        return RedirectToAction("EmployeeList");
    }
}
```

## Step 2: Add a Delete Link on the List Page

In the list page, add a delete link/button that calls the `CityDelete` action method. This link will pass the `CityID` to the method to identify which product to delete.

```html
<form method="post" asp-controller="City" asp-action="CityDelete">
  <input type="hidden" name="CityID" value="@dataRow["CityID"]" />
  <button type="submit" class="btn btn-outline-danger btn-xs">
    <i class="bi bi-x"></i>
  </button>
</form>
```

Another way to add a delete link is to use an anchor tag with a `href` attribute that points to the `CityDelete` action method. This method will be called when the user clicks the link.

```html
<a href="/City/CityDelete?CityID=@dataRow["CityID"]" class="btn btn-outline-danger btn-xs">
  <i class="bi bi-x"></i>
</a>

```

using the `asp-route-` attribute:

```html
<a asp-controller="City" asp-action="CityDelete" asp-route-CityID="@dataRow["CityID"]" class="btn btn-outline-danger btn-xs">
  <i class="bi bi-x"></i>
</a>
```

## Step 3: Test the Delete Operation

Display Error Message in View Page
```csharp
<span class="text-danger">@TempData["ErrorMessage"]</span>
```
Ensure that the delete functionality works by testing it in the application. Check that the product is removed from the database and that the list page updates accordingly.


# Add-Edit Operation in ASP.NET Core MVC

**Prerequisite**: Procedures for Insert, Update, and Select by Primary Key Operations

### Insert Procedure

```sql
CREATE PROCEDURE sp_InsertEmployee
    @FirstName NVARCHAR(100),
    @LastName NVARCHAR(100),
    @Email NVARCHAR(255),
    @PhoneNumber NVARCHAR(20) = NULL,
    @DateOfBirth DATE = NULL,
    @Gender NVARCHAR(10) = NULL,
    @HireDate DATE,
    @JobTitle NVARCHAR(100) = NULL,
    @Department NVARCHAR(100) = NULL,
    @Salary DECIMAL(18, 2) = NULL
AS
BEGIN
    SET NOCOUNT ON;
    INSERT INTO Employee (
        FirstName,
        LastName,
        Email,
        PhoneNumber,
        DateOfBirth,
        Gender,
        HireDate,
        JobTitle,
        Department,
        Salary
    )
    VALUES (
        @FirstName,
        @LastName,
        @Email,
        @PhoneNumber,
        @DateOfBirth,
        @Gender,
        @HireDate,
        @JobTitle,
        @Department,
        @Salary
    );
END
```

### Update Procedure

```sql
CREATE PROCEDURE sp_UpdateEmployee
    @EmployeeId INT,
    @FirstName NVARCHAR(100),
    @LastName NVARCHAR(100),
    @Email NVARCHAR(255),
    @PhoneNumber NVARCHAR(20),
    @DateOfBirth DATE,
    @Gender NVARCHAR(10),
    @HireDate DATE,
    @JobTitle NVARCHAR(100),
    @Department NVARCHAR(100),
    @Salary DECIMAL(18, 2),
    @IsActive BIT
AS
BEGIN
    SET NOCOUNT ON;
    UPDATE Employee
    SET
        FirstName = @FirstName,
        LastName = @LastName,
        Email = @Email,
        PhoneNumber = @PhoneNumber,
        DateOfBirth = @DateOfBirth,
        Gender = @Gender,
        HireDate = @HireDate,
        JobTitle = @JobTitle,
        Department = @Department,
        Salary = @Salary,
        IsActive = @IsActive,
        UpdatedAt = GETDATE() -- Update the timestamp
    WHERE
        EmployeeId = @EmployeeId;
END
```

### Select By Primary Key Procedure

```sql
CREATE PROCEDURE sp_GetEmployeeById
    @EmployeeId INT
AS
BEGIN
    SET NOCOUNT ON;
    SELECT
        EmployeeId,
        FirstName,
        LastName,
        Email,
        PhoneNumber,
        DateOfBirth,
        Gender,
        HireDate,
        JobTitle,
        Department,
        Salary,
        IsActive,
        CreatedAt,
        UpdatedAt
    FROM Employee
    WHERE EmployeeId = @EmployeeId;
END
```

## Step 1: Implement Logic for Insert-Update Operations in the `CountryAddEdit` Action Method

```csharp
 public IActionResult EmployeeSave(EmployeeModel employeeModel)
 {
     if (ModelState.IsValid)
     {
         string connectionString = this.configuration.GetConnectionString("ConnectionString");
         using (SqlConnection connection = new SqlConnection(connectionString))
         {
             connection.Open();
             SqlCommand command = connection.CreateCommand();
             command.CommandType = CommandType.StoredProcedure;

             if (employeeModel.EmployeeId == 0 || employeeModel.EmployeeId == null)
             {
                 command.CommandText = "sp_InsertEmployee";
             }
             else
             {
                 command.CommandText = "sp_UpdateEmployee";
                 command.Parameters.Add("@EmployeeId", SqlDbType.Int).Value = employeeModel.EmployeeId;
                 command.Parameters.Add("@IsActive", SqlDbType.Bit).Value = employeeModel.IsActive;

             }

             command.Parameters.Add("@FirstName", SqlDbType.NVarChar).Value = employeeModel.FirstName;
             command.Parameters.Add("@LastName", SqlDbType.NVarChar).Value = employeeModel.LastName;
             command.Parameters.Add("@Email", SqlDbType.NVarChar).Value = employeeModel.Email ?? (object)DBNull.Value;
             command.Parameters.Add("@PhoneNumber", SqlDbType.NVarChar).Value = employeeModel.PhoneNumber ?? (object)DBNull.Value;
             command.Parameters.Add("@DateOfBirth", SqlDbType.Date).Value = employeeModel.DateOfBirth ?? (object)DBNull.Value;
             command.Parameters.Add("@Gender", SqlDbType.NVarChar).Value = employeeModel.Gender ?? (object)DBNull.Value;
             command.Parameters.Add("@HireDate", SqlDbType.Date).Value = employeeModel.HireDate;
             command.Parameters.Add("@JobTitle", SqlDbType.NVarChar).Value = employeeModel.JobTitle ?? (object)DBNull.Value;
             command.Parameters.Add("@Department", SqlDbType.NVarChar).Value = employeeModel.Department ?? (object)DBNull.Value;
             command.Parameters.Add("@Salary", SqlDbType.Decimal).Value = employeeModel.Salary ?? (object)DBNull.Value;

             command.ExecuteNonQuery();
         }

         TempData["SuccessMessage"] = "Employee saved successfully.";
         return RedirectToAction("EmployeeList");
     }

     return View("EmployeeAddEdit", employeeModel);
 }
```

## Step 2: Verify Insert Operation

## Step 3: Update Operation (Two Parts)

### Part 1: Fetch and Populate Existing Data in the Text Fields

To retrieve the existing country data and display it in the form for editing, update the `CountryForm` method:

```csharp
 public IActionResult EmployeeSave(EmployeeModel employeeModel)
 {
     if (ModelState.IsValid)
     {
         string connectionString = this.configuration.GetConnectionString("ConnectionString");
         using (SqlConnection connection = new SqlConnection(connectionString))
         {
             connection.Open();
             SqlCommand command = connection.CreateCommand();
             command.CommandType = CommandType.StoredProcedure;

             if (employeeModel.EmployeeId == 0 || employeeModel.EmployeeId == null)
             {
                 command.CommandText = "sp_InsertEmployee";
             }
             else
             {
                 command.CommandText = "sp_UpdateEmployee";
                 command.Parameters.Add("@EmployeeId", SqlDbType.Int).Value = employeeModel.EmployeeId;
                 command.Parameters.Add("@IsActive", SqlDbType.Bit).Value = employeeModel.IsActive;

             }

             command.Parameters.Add("@FirstName", SqlDbType.NVarChar).Value = employeeModel.FirstName;
             command.Parameters.Add("@LastName", SqlDbType.NVarChar).Value = employeeModel.LastName;
             command.Parameters.Add("@Email", SqlDbType.NVarChar).Value = employeeModel.Email ?? (object)DBNull.Value;
             command.Parameters.Add("@PhoneNumber", SqlDbType.NVarChar).Value = employeeModel.PhoneNumber ?? (object)DBNull.Value;
             command.Parameters.Add("@DateOfBirth", SqlDbType.Date).Value = employeeModel.DateOfBirth ?? (object)DBNull.Value;
             command.Parameters.Add("@Gender", SqlDbType.NVarChar).Value = employeeModel.Gender ?? (object)DBNull.Value;
             command.Parameters.Add("@HireDate", SqlDbType.Date).Value = employeeModel.HireDate;
             command.Parameters.Add("@JobTitle", SqlDbType.NVarChar).Value = employeeModel.JobTitle ?? (object)DBNull.Value;
             command.Parameters.Add("@Department", SqlDbType.NVarChar).Value = employeeModel.Department ?? (object)DBNull.Value;
             command.Parameters.Add("@Salary", SqlDbType.Decimal).Value = employeeModel.Salary ?? (object)DBNull.Value;

             command.ExecuteNonQuery();
         }

         TempData["SuccessMessage"] = "Employee saved successfully.";
         return RedirectToAction("EmployeeList");
     }

     return View("EmployeeAddEdit", employeeModel);
 }
```

### Part 2: Update the Country Data in the Database

The logic for updating the country data has already been implemented in the `CountryAddEdit` method.

## Step 4: Add Edit Button with Link in the List Page

```csharp
<a asp-controller="Employee" asp-action="EmployeeAddEdit" class="btn btn-primary mt-3">
    <i class="bi bi-plus-circle"></i> Add Employee
</a>
```

## Step 5: Verify Update Operation
