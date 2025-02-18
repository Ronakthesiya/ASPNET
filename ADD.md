# Add-Edit Operation in ASP.NET Core MVC

**Prerequisite**: Procedures for Insert, Update, and Select by Primary Key Operations

### Insert Procedure

```sql
CREATE PROCEDURE [dbo].[PR_Country_Insert]
@countryName varchar(50), @countryCode varchar(5), @countryCapital varchar(100), @userID int
AS
BEGIN
    INSERT INTO [dbo].[Country]
    (
        [dbo].[Country].[CountryName], 
        [dbo].[Country].[CountryCode], 
        [dbo].[Country].[CountryCapital]
    )
    VALUES
    (
        @countryName, 
        @countryCode, 
        @countryCapital
    )
END
```



## Step 1: Implement Logic for Insert Operations in the `CountryAddEdit` Action Method

```csharp
public IActionResult CountryAddEdit(CountryModel model)
{
    if (ModelState.IsValid)
    {
        string connectionString = this.configuration.GetConnectionString("ConnectionString");
        SqlConnection connection = new SqlConnection(connectionString);
        connection.Open();
        SqlCommand command = connection.CreateCommand();
        command.CommandType = CommandType.StoredProcedure;
    
       
        command.CommandText = "PR_Country_Insert";
        command.Parameters.Add("@CountryName", SqlDbType.VarChar).Value = model.CountryName;
        command.Parameters.Add("@CountryCode", SqlDbType.VarChar).Value = model.CountryCode;
        command.Parameters.Add("@CountryCapital", SqlDbType.VarChar).Value = model.CountryCapital;
        command.ExecuteNonQuery();
        return RedirectToAction("CountryList");
    }
    
    return View("CountryForm", model);
}
```

### Part 1: Fetch and Populate Existing Data in the Text Fields

To retrieve the existing country data and display it in the form for editing, update the `CountryForm` method:

```csharp
public IActionResult CountryForm()
{
    return View();
}
```

## Step 2: Add Edit Button with Link in the List Page

```csharp
<a asp-controller="Country" asp-action="CountryForm" class="btn btn-primary btn-sm" style="padding: 2px 6px; font-size: 12px;">
    ADD
</a>
```

