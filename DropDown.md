# Implementing a Drop-Down in ASP.NET Core MVC

**Prerequisite**: Ensure the Drop-Down Procedure is created in your database.

### Drop-Down Procedure

Below is the SQL stored procedure for retrieving country data for the drop-down list:

```sql
CREATE PROCEDURE [dbo].[PR_Country_SelectForDropDown]
@UserID INT
AS
BEGIN
    SELECT 
        [dbo].[Country].[CountryID],
        [dbo].[Country].[CountryName]
    FROM [dbo].[Country]
    WHERE [dbo].[Country].[UserID] = @UserID
    ORDER BY [dbo].[Country].[CountryName]
END
```

## Step 1: Create a Model for the Drop-Down

In your `CountryModel.cs`, create a model that represents the drop-down data.


example: inside CountryModel create a CountryDropDownModel

```csharp
public class CountryDropDownModel
{
    public int CountryID { get; set; }
    public string CountryName { get; set; }
}
```

## Step 2: Implement Logic to Fetch Data for the Drop-Down in the Controller

Create New Method `CountryDropDown` add logic to retrieve the country data for the drop-down list from the database.

```csharp
public void CountryDropDown()
{
    string connectionString = this._configuration.GetConnectionString("ConnectionString");
    SqlConnection connection = new SqlConnection(connectionString);
    connection.Open();
    SqlCommand command2 = connection.CreateCommand();
    command2.CommandType = System.Data.CommandType.StoredProcedure;
    command2.CommandText = "PR_Country_SelectForDropDown";
    command2.Parameters.Add("@UserID", SqlDbType.Int).Value = CommonVariable.UserID();
    SqlDataReader reader2 = command2.ExecuteReader();
    DataTable dataTable2 = new DataTable();
    dataTable2.Load(reader2);
    List<CountryDropDownModel> countryList = new List<CountryDropDownModel>();
    foreach (DataRow data in dataTable2.Rows)
    {
        CountryDropDownModel model = new CountryDropDownModel();
        model.CountryID = Convert.ToInt32(data["CountryID"]);
        model.CountryName = data["CountryName"].ToString();
        countryList.Add(model);
    }
    ViewBag.CountryList = countryList;
}
```

## Step 3: Call This Method in `StateForm` Action Method and `StateAddEdit` method where `ModelState.IsValid` gets False

```csharp
public IActionResult StateForm()
{
    CountryDropDown();
}
```

```csharp
public IActionResult StateAddEdit(StateModel model)
{
    if (ModelState.IsValid)
    {
        return RedirectToAction("StateList");
    }
    CountryDropDown();
    return View("StateForm", model);
}
```

## Step 4: Add the Drop-Down in `StateForm.cshtml`

In the `StateForm.cshtml` file, add a `<select>` tag for the drop-down list and use the `asp-items` tag helper to bind the data.

```csharp
<select class="form-control" asp-for="CountryID" asp-items="@(new SelectList(ViewBag.CountryList, "CountryID", "CountryName"))">
    <option value="">Select Country</option>
</select>
```

another way to add the drop-down list is to use the `foreach` loop in the view file.

```csharp
<select class="form-control" asp-for="CountryID">
    <option value="">Select Country</option>
    @foreach (var country in ViewBag.CountryList)
    {
        <option value="@country.CountryID">@country.CountryName</option>
    }
</select>
```

## Step 5: Test the Drop-Down

Open the Add-Edit page in your application and verify that the drop-down list is populated with the correct data from the database.
