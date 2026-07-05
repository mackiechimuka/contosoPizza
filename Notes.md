# W01 Assignments

## Part 1 - ASP.NET Core Controllers

The original Pizza list from the Microsoft Learn module contained:

- Classic Italian
- Veggie

I added the following pizza record:

- BBQ Steak

Current Pizza List:

```json
[
  {
    "id": 1,
    "name": "Classic Italian",
    "isGlutenFree": false
  },
  {
    "id": 2,
    "name": "Veggie",
    "isGlutenFree": true
  },
  {
    "id": 3,
    "name": "BBQ Steak",
    "isGlutenFree": false
  }
]
```

The Microsoft Learn module originally ended with two pizzas (Classic Italian and Veggie). The BBQ Steak pizza was added as the required additional record. 

## Part 2 - The Whole code for working with files
using System.IO;
using System.Collections.Generic;
using Newtonsoft.Json;
using System.Text;

var currentDirectory = Directory.GetCurrentDirectory();
var storesDirectory = Path.Combine(currentDirectory, "stores");
var salesTotalDir = Path.Combine(currentDirectory, "salesTotalDir");
var salesSummaryDir = Path.Combine(currentDirectory,"salesSummary.Dir");
Directory.CreateDirectory(salesTotalDir);
Directory.CreateDirectory(salesSummaryDir);
var salesFiles = FindFiles(storesDirectory);
var salesTotal = CalculateSalesTotal(salesFiles); 
var salesSummary = CalculateSalesSummary(salesFiles);
File.AppendAllText(Path.Combine(salesTotalDir, "totals.txt"), $"{salesTotal:C}{Environment.NewLine}");
StringBuilder salesSummayBuilder = new StringBuilder();
salesSummayBuilder.AppendLine("Sales Summary");
salesSummayBuilder.AppendLine("---------------------");
salesSummayBuilder.AppendLine($"Total Sales:{salesTotal:C}");
salesSummayBuilder.AppendLine();
salesSummayBuilder.AppendLine("Report generated on:"+ DateTime.Now.ToString("MM/dd/yyyy HH:mm:ss"));
foreach(var item in salesSummary)
{
    salesSummayBuilder.AppendLine($"{item.Key} : {item.Value:C}");
}
File.WriteAllText(Path.Combine(salesSummaryDir, "summary.txt"), $"{salesSummayBuilder.ToString()}{Environment.NewLine}");

IEnumerable<string> FindFiles(string folderName)
{ 
    List<string> salesFiles = new List<string>();

    var foundFiles = Directory.EnumerateFiles(folderName, "*", SearchOption.AllDirectories);

    foreach (var file in foundFiles)
    {
        // The file name will contain the full path, so only check the end of it
        if (file.EndsWith("sales.json"))
        {
            salesFiles.Add(file);
        }
    }

    return salesFiles;
}

double CalculateSalesTotal(IEnumerable<string> salesFiles)
{
    double salesTotal = 0;

      // Loop over each file path in salesFiles
    foreach (var file in salesFiles)
    {      
        // Read the contents of the file
        string salesJson = File.ReadAllText(file);

        // Parse the contents as JSON
        SalesData? data = JsonConvert.DeserializeObject<SalesData?>(salesJson);

        // Add the amount found in the Total field to the salesTotal variable
        salesTotal += data?.Total ?? 0;
    }

    return salesTotal;

}

Dictionary<string,double> CalculateSalesSummary(IEnumerable<string> salesFiles)
{
    Dictionary<string,double> salesSummary = new Dictionary<string, double>();

    foreach(var file in salesFiles)
    {
        string salesjson = File.ReadAllText(file);
        SalesData? data = JsonConvert.DeserializeObject<SalesData?>(salesjson);
        salesSummary[file] = data?.Total ?? 0;
    }
    return salesSummary;
}
record SalesData (double Total);


