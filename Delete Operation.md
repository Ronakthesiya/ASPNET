# Deleting a Data Entry in ASP.NET Core MVC

**Prerequisite**: Ensure the Delete Procedure is set up in your database.

### Delete Procedure

Below is the SQL stored procedure for deleting a product from the database:

```sql
CREATE PROCEDURE [dbo].[PR_City_Delete]
    @CityID INT,
    @UserID INT
AS
BEGIN
    DELETE FROM [dbo].[City]
    WHERE 
        CityID = @CityID AND UserID = @UserID;
END
```

## Step 1: Implement the `CityDelete` Action Method in the Controller

Add the following code in your controller to handle the deletion of a product. This method connects to the database, executes the delete procedure, and then redirects to the product list page.

```csharp
public IActionResult CityDelete(int CityID)
{
    try
    {
        string connectionString = configuration.GetConnectionString("ConnectionString");
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            connection.Open();
            SqlCommand command = connection.CreateCommand();
            command.CommandType = CommandType.StoredProcedure;
            command.CommandText = "PR_City_DeleteByPK";
            command.Parameters.Add("@CityID", SqlDbType.Int).Value = CityID;


            command.ExecuteNonQuery();
        }

        TempData["SuccessMessage"] = "City deleted successfully.";
        return RedirectToAction("CityList");
    }
    catch (Exception ex)
    {
        TempData["ErrorMessage"] = "An error occurred while deleting the city: " + ex.Message;
        return RedirectToAction("CityList");
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
