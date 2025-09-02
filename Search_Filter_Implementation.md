# Search Filter Implementation Guide

## Overview
This guide explains how to implement search functionality on a list page that calls a SQL stored procedure and displays filtered results.

## Step 1: Create Search Form in View

**Purpose**: Create a user interface that allows users to enter search criteria and submit search requests.

**Description**: This step involves creating an HTML form with input fields for search parameters. The form uses POST method to send data to the controller and includes Bootstrap classes for styling.

### 1.1 Search Form HTML
```html
<form method="post" asp-action="Index">
    <div class="row">
        <div class="col-md-3">
            <input type="text" name="ProductName" class="form-control" placeholder="Product Name" value="@ViewBag.ProductName" />
        </div>
        <div class="col-md-3">
            <input type="text" name="ProductCode" class="form-control" placeholder="Product Code" value="@ViewBag.ProductCode" />
        </div>
        <div class="col-md-2">
            <button type="submit" class="btn btn-primary">Search</button>
        </div>
        <div class="col-md-2">
            <a href="@Url.Action("Index")" class="btn btn-secondary">Clear</a>
        </div>
    </div>
</form>
```

## Step 2: Controller Implementation

**Purpose**: Handle search requests from the view and execute database operations to retrieve filtered data.

**Description**: The controller action receives search parameters, establishes database connection, calls stored procedure with parameters, and returns filtered results to the view. It also handles errors and preserves search criteria for user experience.

### 2.1 Index Action Method
```csharp
public IActionResult Index(string ProductName = "", string ProductCode = "")
{
    DataTable dt = new DataTable();
    
    try
    {
        SqlConnection objConn = new SqlConnection(this.Configuration.GetConnectionString("CoffeeShop"));
        objConn.Open();
        
        SqlCommand objCmd = objConn.CreateCommand();
        objCmd.CommandType = CommandType.StoredProcedure;
        objCmd.CommandText = "PR_Product_Search";
        
        objCmd.Parameters.Add("@ProductName", SqlDbType.VarChar).Value = ProductName ?? "";
        objCmd.Parameters.Add("@ProductCode", SqlDbType.VarChar).Value = ProductCode ?? "";
        
        SqlDataAdapter objAdapter = new SqlDataAdapter(objCmd);
        objAdapter.Fill(dt);
        
        objConn.Close();
    }
    catch (Exception ex)
    {
        TempData["ErrorMessage"] = ex.Message;
    }
    
    ViewBag.ProductName = ProductName;
    ViewBag.ProductCode = ProductCode;
    
    return View(dt);
}
```

## Step 3: SQL Stored Procedure

**Purpose**: Execute database queries with dynamic filtering based on search parameters.

**Description**: The stored procedure accepts search parameters and builds dynamic WHERE conditions. It uses LIKE operator for partial text matching and handles empty parameters by ignoring them in the filter. Results are ordered for consistent display.

### 3.1 Create Search Stored Procedure
```sql
CREATE PROCEDURE PR_Product_Search
    @ProductName VARCHAR(100) = '',
    @ProductCode VARCHAR(50) = ''
AS
BEGIN
    SELECT 
        ProductID,
        ProductName,
        ProductCode,
        ProductPrice,
        Description
    FROM Product
    WHERE 
        (@ProductName = '' OR ProductName LIKE '%' + @ProductName + '%')
        AND (@ProductCode = '' OR ProductCode LIKE '%' + @ProductCode + '%')
    ORDER BY ProductName
END
```

## Step 4: Display Filtered Results

**Purpose**: Present the filtered data in a user-friendly table format with action buttons.

**Description**: This step creates an HTML table that displays the search results. It includes conditional rendering to show "No records found" message when no data matches the search criteria. Each row includes action buttons for Edit and Delete operations.

### 4.1 Results Table in View
```html
<table class="table table-striped">
    <thead>
        <tr>
            <th>Product Name</th>
            <th>Product Code</th>
            <th>Price</th>
            <th>Description</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        @if (Model.Rows.Count > 0)
        {
            foreach (DataRow dr in Model.Rows)
            {
                <tr>
                    <td>@dr["ProductName"]</td>
                    <td>@dr["ProductCode"]</td>
                    <td>@dr["ProductPrice"]</td>
                    <td>@dr["Description"]</td>
                    <td>
                        <a asp-action="Edit" asp-route-id="@dr["ProductID"]" class="btn btn-sm btn-info">Edit</a>
                        <a asp-action="Delete" asp-route-id="@dr["ProductID"]" class="btn btn-sm btn-danger">Delete</a>
                    </td>
                </tr>
            }
        }
        else
        {
            <tr>
                <td colspan="5" class="text-center">No records found</td>
            </tr>
        }
    </tbody>
</table>
```
