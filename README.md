# ISO 20022 Parsing .Net 8 Library
[![NuGet Version](https://img.shields.io/nuget/v/ISO20022.Parser)](https://www.nuget.org/packages/ISO20022.Parser/)
![NuGet Downloads](https://img.shields.io/nuget/dt/ISO20022.Parser)

ISO20022.Parser is the .Net 8 library which help to parse ISO 200022 messages in the fastes way possible with the predefined rule sets.


**NOTE: MXParser has been deprecated. Please switch to ISO20022.Parser**

## Features
* Parse the ISO 20022 XML Message with pre-defined rules.
  * Rules can be in JSON structure or Database structured.

## Install the `ISO20022.Parser` NuGet package.
  * .NET CLI
  ```cs
    dotnet add package ISO20022.Parser
  ```
  * Package Manager
  ```cs
  Install-Package ISO20022.Parser
  ```

## Features in details:

### Parsing the ISO 20022 Messages


```C#
using System.Data;
using System.Diagnostics;
using ISO20022.Parser;

public class Program
{
    private static async Task Main(string[] args)
    {
        Stopwatch stopwatch = new();
        stopwatch.Start();
        IParsingService parsingService = new ParsingService(new ParsingJsonRules(@"data\parsing_rules.json"), "Document");
        await parsingService.ParseXmlAsync(@"data\camt.053.xml", (DataSet ds, Guid messageUniqueId, Guid uniquiId) =>
        {
          //DataSet ds: camt.053.xml will have multiple messages in a file and this dataset will have the parsed data of the given message. Each message parsed will be callbacked over here for the further process.
           // messageUniqueId :  As camt.053 will have multiple message, each message will identified with the given messageUniqueId. 
           // uniqueId: this id is represent the unique file Id. Meaning each passed file has been represent by the given Id. 
            Task task = Task.Run(() =>
            {
                // either we can transform to CSV/Excel or transfered to DB using bulk DB operation
                string dateTime = DateTime.Now.ToString("ddmmyyyyHHmmssfff");
                foreach(DataTable dt in ds.Tables)
                {
                    ExportDataTableToCsv(dt, @$"export\{messageUniqueId}_{dt.TableName}_{uniquiId}.csv");
                }
               
            });
            return task;
        });
        stopwatch.Stop();
        TimeSpan elapsedTime = stopwatch.Elapsed;

        // Print the elapsed time
        Console.WriteLine("Elapsed Time: " + elapsedTime);
        Console.Read();
    }

    static void ExportDataTableToCsv(DataTable dataTable, string filePath)
    {
        if (!File.Exists(filePath))
        {
            File.Create(filePath).Close();
        }
        // Create a StreamWriter to write to the CSV file
        using (StreamWriter writer = new StreamWriter(filePath))
        {
            // Write the header row with column names
            foreach (DataColumn column in dataTable.Columns)
            {
                writer.Write(column.ColumnName);
                writer.Write(",");
            }
            writer.WriteLine(); // Move to the next line

            // Write the data rows
            foreach (DataRow row in dataTable.Rows)
            {
                foreach (object item in row.ItemArray)
                {
                    writer.Write(item);
                    writer.Write(",");
                }
                writer.WriteLine(); // Move to the next line
            }
        }
    }

}
```
**NOTE
The library can parse the given 60.6MB xml data file in 00:00:19 seconds with 4 Cores 8 Logical Processor with 16 MB of RAM.**

### Parsing Rules
Please follow the Parsing Rules example given in the data directory of the repository.
Mapping Rules should follow below model class structure

```C#
public class MappingRules
{
    public string Namespace { get; set; }
    public List<MappingTable> Mappings { get; set; }
}

public class MappingTable
{
    public string NodeName { get; set; }
    public string XPath { get; set; }
    public IList<MappingColumn> Columns { get; set;}
}

public class MappingColumn
{
    public int OrderNumber { get; set; }
    public string NodeName { get; set; }
    public string XPath { get; set; }
    public string DataType { get; set; }
    public int MinLength { get; set; }
    public int MaxLength { get; set; }
    public int fractionDigits { get; set; }
    public int totalDigits { get; set; }
}
```
### Override the Parsing Rules
The given example is in json file. Suppose you want to store Parsing rules in Database. Now you need to override the ParsingRules class method DeserializeRules.

For Example:
```C#
public class ParsingDatabaseRules : ParsingRules
{
    public override void DeserializeRules()
    {
      //MapParsingRules is of type IList<MappingRules>?
        MapParsingRules = GetMyRulesFromDB();
    }
}
```

### Sample Files

[camt.053.xml](https://github.com/pixelbyaj/data/raw/main/MParser/camt.053.xml)

[parsing_rules.xml](https://github.com/pixelbyaj/data/raw/main/MParser/parsing_rules.json)
