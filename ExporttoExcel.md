# Export to Excel using ClosedXML in ASP.NET Core MVC

This project demonstrates how to export data to an Excel file using **ClosedXML** in ASP.NET Core MVC. The process involves retrieving data via a stored procedure, generating an Excel file, and returning it to the user as a downloadable file.

---

## 📌 Features
- Retrieve data from SQL Server using a stored procedure.
- Export data into `.xlsx` format using **ClosedXML**.
- Download the file directly from the browser.

---

## 📂 Project Structure

### **1. Using NuGet Package Manager**
In Visual Studio, go to:
#### Tools → NuGet Package Manager → Manage NuGet Packages for Solution...
Go To Browse Tab.

#### Search For `ClosedXML.`

Select the package and Click Install.

Accept the License Agreement.


### **2. Model: `StateModel.cs`**
Represents the structure of the `State` entity, with data annotations for validation.

```csharp
using Microsoft.AspNetCore.Mvc;
using System.ComponentModel.DataAnnotations;

namespace Lab_19_26_Address_Book.Areas.State.Models
{
    public class StateModel
    {
        public int StateID { get; set; }
        
        [Required(ErrorMessage = "State Name Must Be Required.")]
        [Length(0, 100)]
        public string StateName { get; set; }

        [Required(ErrorMessage = "Country Name Must Be Select For State.")]
        public int CountryID { get; set; }
        
        [Required(ErrorMessage = "StateCode Must Be Required.")]
        [Length(0, 10)]
        public string StateCode { get; set; }

        public int? CityCount { get; set; }
    }
}
```

### **3. Controller: `StateController.cs`**
Represents the code for Export Data to Excel File using ClosedXML Library.

```csharp
[Route("ExportToExcel")]
public IActionResult ExportToExcel()
{
    DataTable dt = RetrieveData("PR_LOC_State_SelectAll");

    using (var workbook = new XLWorkbook())
    {
        // Add the DataTable to a worksheet
        workbook.Worksheets.Add(dt, "States");

        using (var stream = new MemoryStream())
        {
            workbook.SaveAs(stream);
            var content = stream.ToArray();

            return File(
                content,
                "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
                "States.xlsx"
            );
        }
    }
}
```

Represents the code for RetrieveData() method.
This method fetches data using a stored procedure and returns it as a DataTable.

```csharp
public DataTable RetrieveData(String SP, int? PKID = 0, String PKName = "")
{
    SqlConnection conn = new SqlConnection(this.configuration.GetConnectionString("myConnection"));
    conn.Open();

    SqlCommand cmd = conn.CreateCommand();
    cmd.CommandType = CommandType.StoredProcedure;
    cmd.CommandText = SP;
    if (PKID != 0)
    {
        cmd.Parameters.AddWithValue("@" + PKName, PKID);
    }
    SqlDataReader reader = cmd.ExecuteReader();
    DataTable dt = new DataTable();
    dt.Load(reader);
    conn.Close();

    return dt;
}
```

### **4. View: `StateTable.cshtml`**
below given button triggers the ExportToExcel action.

```html
<a class="btn btn-success ms-2"
    asp-area="State"
    asp-controller="State"
    asp-action="ExportToExcel">
    Export To Excel
</a>
```
