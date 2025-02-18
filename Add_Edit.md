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
        [dbo].[Country].[CountryCapital], 
        [dbo].[Country].[UserID]
    )
    VALUES
    (
        @countryName, 
        @countryCode, 
        @countryCapital, 
        @userID
    )
END
```

### Update Procedure

```sql
CREATE PROCEDURE [dbo].[PR_Country_UpdateByPK]
    @CountryID INT,
    @CountryName VARCHAR(100),
    @CountryCode VARCHAR(5),
    @CountryCapital VARCHAR(100),
    @UserID INT
AS
BEGIN
    UPDATE [dbo].[Country]
    SET 
        [dbo].[Country].[CountryName] = @CountryName,
        [dbo].[Country].[CountryCode] = @CountryCode,
        [dbo].[Country].[CountryCapital] = @CountryCapital
    WHERE [dbo].[Country].[CountryID] = @CountryID AND [dbo].[Country].[UserID] = @UserID;
END
```

### Select By Primary Key Procedure

```sql
CREATE PROCEDURE [dbo].[PR_Country_SelectByPK]
@countryID int, @userID int
AS
BEGIN
    SELECT
		[dbo].[Country].[CountryName],
		[dbo].[Country].[CountryCode],
		[dbo].[Country].[CountryCapital]
	FROM 
        [dbo].[Country]
	WHERE 
        [dbo].[Country].[CountryID] = @countryID and [dbo].[Country].[UserID] = @userID
END
```

## Step 1: Implement Logic for Insert-Update Operations in the `CountryAddEdit` Action Method

```csharp
public IActionResult CountryAddEdit(CountryModel model)
{
    if (ModelState.IsValid)
    {
        string connectionString = this._configuration.GetConnectionString("ConnectionString");
        SqlConnection connection = new SqlConnection(connectionString);
        connection.Open();
        SqlCommand command = connection.CreateCommand();
        command.CommandType = CommandType.StoredProcedure;

        if (model.CountryID == 0)
        {
            command.CommandText = "PR_Country_Insert";
        }
        else
        {
            command.CommandText = "PR_Country_UpdateByPK";
            command.Parameters.Add("@CountryID", SqlDbType.Int).Value = model.CountryID;
        }
        command.Parameters.Add("@CountryName", SqlDbType.VarChar).Value = model.CountryName;
        command.Parameters.Add("@CountryCode", SqlDbType.VarChar).Value = model.CountryCode;
        command.Parameters.Add("@CountryCapital", SqlDbType.VarChar).Value = model.CountryCapital;
        command.Parameters.Add("@UserID", SqlDbType.Int).Value = CommonVariable.UserID();
        command.ExecuteNonQuery();
        return RedirectToAction("CountryList");
    }

    return View("CountryForm", model);
}
```

## Step 2: Verify Insert Operation

## Step 3: Update Operation (Two Parts)

### Part 1: Fetch and Populate Existing Data in the Text Fields

To retrieve the existing country data and display it in the form for editing, update the `CountryForm` method:

```csharp
public IActionResult CountryForm(int? countryID)
{
    if (countryID == null)
    {
        var m = new CountryModel
        {
            CreationDate = DateTime.Now
        };
        return View(m);
    }

    string connectionString = this._configuration.GetConnectionString("ConnectionString");
    SqlConnection connection = new SqlConnection(connectionString);
    connection.Open();
    SqlCommand command = connection.CreateCommand();
    command.CommandType = CommandType.StoredProcedure;
    command.CommandText = "PR_Country_SelectByPK";
    command.Parameters.AddWithValue("@CountryID", countryID);
    command.Parameters.AddWithValue("@UserID", CommonVariable.UserID());
    SqlDataReader reader = command.ExecuteReader();
    DataTable table = new DataTable();
    table.Load(reader);
    CountryModel model = new CountryModel();

    foreach (DataRow dataRow in table.Rows)
    {
        model.CountryName = dataRow["CountryName"].ToString();
        model.CountryCode = dataRow["CountryCode"].ToString();
        model.CountryCapital = dataRow["CountryCapital"].ToString();
    }
    return View(model);
}
```

### Part 2: Update the Country Data in the Database

The logic for updating the country data has already been implemented in the `CountryAddEdit` method.

## Step 4: Add Edit Button with Link in the List Page

```csharp
<a asp-controller="Country" asp-action="CountryForm" asp-route-CountryID="@dataRow["CountryID"]" class="btn btn-primary btn-sm" style="padding: 2px 6px; font-size: 12px;">
    <i class="fa fa-pencil"></i>
</a>
```

## Step 5: Verify Update Operation
