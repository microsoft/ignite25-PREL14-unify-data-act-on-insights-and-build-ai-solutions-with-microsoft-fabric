![](../assets/images/microsoft-logo.png)

# End-to-End Data Engineering:<br>Modern Data Warehousing on Microsoft Fabric

## Lab 4 - Data transformation using T-SQL

Before you begin:

- Make sure you have read the overview on the [workshop homepage](<../README.md>).
- If you have not completed [Lab 3 - Load data](<03 - Loading data.md>), go complete all the steps then return here to continue.

This lab will cover:

- <a href="#4.1">Stored procedures</a>
- <a href="#4.2">Incrementally updating tables</a>

<hr>

<h3 id = "4.1">4.1 - Stored procedures</h3>

*Note: If you just completed Lab 3 and still have The Workshop notebook open, remain in The Workshop notebook, navigate to **Lab 4 - Data transformation using T-SQL**, locate the **4.1 - Stored procedures** section, and move straight to step 3 below.*

1. Return to the *Modern Data Warehousing on Microsoft Fabric* workspace created in *Lab 0 - Lab environment setup* by selecting the **workspace icon** from the left navigation bar. 

    *Note: The icons on the navigation bar can be pinned and unpinned. Therefore, the icons you see may differ from the screenshot.*

    ![](../assets/images/04_navigation_bar.png)

1. From the item list, select **The Workshop** notebook and navigate to **Lab 4 - Data transformation using T-SQL**, and locate the **4.1 - Stored procedures** section.

    ![](../assets/images/04_workspace.png)

1. Create the stored procedure to generate the unknown member for each dimension table if it does not exist by running the cell for **Step 4.1.3** in *The Workshop* notebook. Upon completion, the cell will have a message output but no query results.

    ``` sql
    DROP PROCEDURE IF EXISTS dbo.CreateUnknownMembers
    GO

    CREATE PROCEDURE dbo.CreateUnknownMembers
    AS
    BEGIN

        IF NOT EXISTS (SELECT * FROM dbo.DimCity WHERE CitySK = 0)
        INSERT INTO dbo.DimCity SELECT 0, 0, 'Unknown', 'N/A', 'N/A', 'N/A', 'N/A', 'N/A', 'N/A', NULL, 0

        IF NOT EXISTS (SELECT * FROM dbo.DimCustomer WHERE CustomerSK = 0)
        INSERT INTO dbo.DimCustomer SELECT 0, 0, 'Unknown', 'N/A', 'N/A', 'N/A', 'N/A', 'N/A'

        IF NOT EXISTS (SELECT * FROM dbo.DimEmployee WHERE EmployeeSK = 0)
        INSERT INTO dbo.DimEmployee SELECT 0, 0, 'Unknown', 'N/A', 0

        IF NOT EXISTS (SELECT * FROM dbo.DimStockItem WHERE StockItemSK = 0)
        INSERT INTO dbo.DimStockItem SELECT 0, 0, 'Unknown', 'N/A', 'N/A', 'N/A', 'N/A', 'N/A', 0, 0, 0, 'N/A', 0, 0, 0, 0

    END
    GO 
    ```

    ![](../assets/images/04_stored_procedure_unknown_member.png)

1. Create the stored procedures to perform an UPSERT or MERGE operation on the dimensional model tables by running the cell for **Step 4.1.4** in *The Workshop* notebook. Upon completion, the cell will have a messages output but no query results.

    ``` sql
    -- dbo.UpdateDimCity stored procedure
    DROP PROCEDURE IF EXISTS dbo.UpdateDimCity
    GO


    CREATE PROCEDURE dbo.UpdateDimCity
    AS
    BEGIN

        UPDATE destination
        SET
            destination.[CityAK] 				    = source.[CityKey],
            destination.[City] 						= source.[City],
            destination.[StateProvince] 			= source.[StateProvince],
            destination.[Country] 					= source.[Country],
            destination.[Continent] 				= source.[Continent],
            destination.[SalesTerritory] 			= source.[SalesTerritory],
            destination.[Region] 					= source.[Region],
            destination.[Subregion] 				= source.[Subregion],
            destination.[Location] 					= source.[Location],
            destination.[LatestRecordedPopulation] 	= source.[LatestRecordedPopulation]
        FROM dbo.DimCity AS destination
        INNER JOIN stage.DimCity AS source
            ON destination.[CityAK] = source.[CityKey]

        DECLARE @MaxID BIGINT = (SELECT ISNULL(MAX(CitySK), 0) FROM dbo.DimCity)

        INSERT INTO dbo.DimCity
        SELECT
            @MaxID + ROW_NUMBER() OVER(ORDER BY (SELECT NULL)) AS [CitySK],
            [CityKey],
            [City],
            [StateProvince],
            [Country],
            [Continent],
            [SalesTerritory],
            [Region],
            [Subregion],
            [Location],
            [LatestRecordedPopulation]
        FROM stage.DimCity
        WHERE CityKey NOT IN (SELECT CityAK FROM dbo.DimCity)

    END
    GO


    -- dbo.UpdateDimCustomer stored procedure
    DROP PROCEDURE IF EXISTS dbo.UpdateDimCustomer
    GO

    CREATE PROCEDURE dbo.UpdateDimCustomer
    AS
    BEGIN

        UPDATE destination
        SET
            destination.[CustomerAK] 	    = source.[CustomerKey],
            destination.[Customer] 			= source.[Customer],
            destination.[BillToCustomer] 	= source.[BillToCustomer],
            destination.[Category] 			= source.[Category],
            destination.[BuyingGroup] 		= source.[BuyingGroup],
            destination.[PrimaryContact] 	= source.[PrimaryContact],
            destination.[PostalCode] 		= source.[PostalCode]
        FROM dbo.DimCustomer AS destination
        INNER JOIN stage.DimCustomer AS source
            ON destination.[CustomerAK] = source.[CustomerKey]

        DECLARE @MaxID BIGINT = (SELECT ISNULL(MAX(CustomerSK), 0) FROM dbo.DimCustomer)

        INSERT INTO dbo.DimCustomer
        SELECT
            @MaxID + ROW_NUMBER() OVER(ORDER BY (SELECT NULL)) AS [CustomerSK],
            [CustomerKey],
            [Customer],
            [BillToCustomer],
            [Category],
            [BuyingGroup],
            [PrimaryContact],
            [PostalCode]		
        FROM stage.DimCustomer
        WHERE CustomerKey NOT IN (SELECT CustomerAK FROM dbo.DimCustomer)

    END
    GO


    -- dbo.UpdateDimEmployee stored procedure
    DROP PROCEDURE IF EXISTS dbo.UpdateDimEmployee
    GO

    CREATE PROCEDURE dbo.UpdateDimEmployee
    AS
    BEGIN

        UPDATE destination
        SET
            destination.[EmployeeAK] 	    = source.[EmployeeKey],
            destination.[Employee] 			= source.[Employee],
            destination.[PreferredName] 	= source.[PreferredName],
            destination.[IsSalesperson]		= source.[IsSalesperson]
        FROM dbo.DimEmployee AS destination
        INNER JOIN stage.DimEmployee AS source
            ON destination.[EmployeeAK] = source.[EmployeeKey]

        DECLARE @MaxID BIGINT = (SELECT ISNULL(MAX(EmployeeSK), 0) FROM dbo.DimEmployee)

        INSERT INTO dbo.DimEmployee
        SELECT
            @MaxID + ROW_NUMBER() OVER(ORDER BY (SELECT NULL)) AS [EmployeeSK],
            [EmployeeKey],
            [Employee],
            [PreferredName],
            [IsSalesperson]		
        FROM stage.DimEmployee
        WHERE EmployeeKey NOT IN (SELECT EmployeeAK FROM dbo.DimEmployee)

    END
    GO


    -- dbo.UpdateDimStockItem stored procedure
    DROP PROCEDURE IF EXISTS dbo.UpdateDimStockItem
    GO

    CREATE PROCEDURE dbo.UpdateDimStockItem
    AS
    BEGIN

        UPDATE destination
        SET
            destination.[StockItemAK] 			    = source.[StockItemKey],
            destination.[StockItem] 				= source.[StockItem],
            destination.[Color] 					= source.[Color],
            destination.[SellingPackage] 			= source.[SellingPackage],
            destination.[BuyingPackage] 			= source.[BuyingPackage],
            destination.[Brand] 					= source.[Brand],
            destination.[Size] 						= source.[Size],
            destination.[LeadTimeDays] 				= source.[LeadTimeDays],
            destination.[QuantityPerOuter] 			= source.[QuantityPerOuter],
            destination.[IsChillerStock] 			= source.[IsChillerStock],
            destination.[Barcode] 					= source.[Barcode],
            destination.[TaxRate] 					= source.[TaxRate],
            destination.[UnitPrice] 				= source.[UnitPrice],
            destination.[RecommendedRetailPrice] 	= source.[RecommendedRetailPrice],
            destination.[TypicalWeightPerUnit] 		= source.[TypicalWeightPerUnit]
        FROM dbo.DimStockItem AS destination
        INNER JOIN stage.DimStockItem AS source
            ON destination.[StockItemAK] = source.[StockItemKey]

        DECLARE @MaxID BIGINT = (SELECT ISNULL(MAX(StockItemSK), 0) FROM dbo.DimStockItem)

        INSERT INTO dbo.DimStockItem
        SELECT
            @MaxID + ROW_NUMBER() OVER(ORDER BY (SELECT NULL)) AS [StockItemSK],
            [StockItemKey],
            [StockItem],
            [Color],
            [SellingPackage],
            [BuyingPackage],
            [Brand],
            [Size],
            [LeadTimeDays],
            [QuantityPerOuter],
            [IsChillerStock],
            [Barcode],
            [TaxRate],
            [UnitPrice],
            [RecommendedRetailPrice],
            [TypicalWeightPerUnit]
        FROM stage.DimStockItem
        WHERE StockItemKey NOT IN (SELECT StockItemAK FROM dbo.DimStockItem)

    END
    GO


    -- dbo.UpdateFactSale stored procedure
    DROP PROCEDURE IF EXISTS dbo.UpdateFactSale
    GO

    CREATE PROCEDURE dbo.UpdateFactSale
    AS
    BEGIN

        DECLARE @MaxID BIGINT = (SELECT ISNULL(MAX(SaleKey), 0) FROM dbo.FactSale)

        INSERT INTO dbo.FactSale
        SELECT
            @MaxID + ROW_NUMBER() OVER(ORDER BY (SELECT NULL)) AS [SaleKey],
            ISNULL(dci.CitySK, 0) AS CityKey,
            ISNULL(dcu.CustomerSK, 0) AS CustomerKey,
            ISNULL(dbtc.CustomerSK, 0) AS BillToCustomerKey,
            ISNULL(dsi.StockItemSK, 0) AS StockItemKey,
            fs.InvoiceDateKey,
            fs.DeliveryDateKey,
            ISNULL(de.EmployeeSK, 0) AS SalespersonKey,
            fs.WWIInvoiceID,
            fs.[Description],
            fs.Package,
            fs.Quantity,
            fs.UnitPrice,
            fs.TaxRate,
            fs.TotalExcludingTax,
            fs.TaxAmount,
            fs.Profit,
            fs.TotalIncludingTax,
            fs.TotalDryItems,
            fs.TotalChillerItems
        FROM stage.FactSale AS fs
        LEFT JOIN dbo.DimCity AS dci
            ON fs.CityKey = dci.CityAK
        LEFT JOIN dbo.DimCustomer AS dcu
            ON fs.CustomerKey = dcu.CustomerAK
        LEFT JOIN dbo.DimCustomer AS dbtc
            ON fs.BillToCustomerKey = dbtc.CustomerAK
        LEFT JOIN dbo.DimStockItem AS dsi
            ON fs.StockItemKey = dsi.StockItemAK
        LEFT JOIN dbo.DimEmployee de
            ON fs.SalespersonKey = de.EmployeeAK
        LEFT JOIN dbo.FactSale AS f
            ON fs.WWIInvoiceID = f.InvoiceID
            AND dsi.StockItemSK = f.StockItemSK
            AND fs.InvoiceDateKey = f.InvoiceDateKey
        WHERE
            f.SaleKey IS NULL
            
    END
    GO
    ```

    ![](../assets/images/04_stored_procedure_create.png)

1. Validate the stored procedures were created by running the cell for **Step 4.1.5** in *The Workshop* notebook and comparing the results to those below.

    ``` sql
    SELECT * FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_TYPE = 'PROCEDURE'
    ```

    ![](../assets/images/04_stored_procedure_create_validate.png)


1. If you prefer to validate using the **Explorer**, refresh the warehouse object list by selecting the ellipsis (**...**) that appears when hovering the mouse of the warehouse name *WideWorldImportersDW*, then select **Refresh**.

    ![](../assets/images/04_explorer_refresh.png)

1. In the **Explorer**, expand **WideWorldImportersDW -> Schemas -> dbo -> Stored Procedures** to validate that all the procedures were created successfully. Do note that it can take up to **15 minutes** for the notebook Explorer to display the procedures even though they are in the database as seen in step 5 above.

    - CreateUnknownMembers
    - UpdateDimCity
    - UpdateDimCustomer
    - UpdateDimEmployee
    - UpdateDimStockItem
    - UpdateFactSale

    ![](../assets/images/04_explorer_stored_procedures.png)

<h3 id = "4.2">4.2 - Incrementally updating tables</h3>

Before beginning, open *The Workshop* notebook, navigate to **Lab 4 - Data transformation using T-SQL**, and locate the **4.2 - Incrementally updating tables** section.

1. Execute all the stored procedures created in the previous section by running the cell for **Step 4.2.1** in *The Workshop* notebook. 

    *Note: More detail about what this code does and exploration of the results are provided in the next step.*

    ``` sql
    DECLARE @CountBeforeLoadDimCity      BIGINT = (SELECT COUNT_BIG(*) FROM dbo.DimCity)
    DECLARE @CountBeforeLoadDimCustomer  BIGINT = (SELECT COUNT_BIG(*) FROM dbo.DimCustomer)
    DECLARE @CountBeforeLoadDimEmployee  BIGINT = (SELECT COUNT_BIG(*) FROM dbo.DimEmployee)
    DECLARE @CountBeforeLoadDimStockItem BIGINT = (SELECT COUNT_BIG(*) FROM dbo.DimStockItem)
    DECLARE @CountBeforeLoadFactSale     BIGINT = (SELECT COUNT_BIG(*) FROM dbo.FactSale)

    EXEC dbo.CreateUnknownMembers;
    EXEC dbo.UpdateDimCity;
    EXEC dbo.UpdateDimCustomer;
    EXEC dbo.UpdateDimEmployee;
    EXEC dbo.UpdateDimStockItem;
    EXEC dbo.UpdateFactSale;

    SELECT 'dbo'   AS SchemaName, 'DimCity'        AS TableName, FORMAT(@CountBeforeLoadDimCity,      'N0') AS RecordCount_BeforeLoad, FORMAT(COUNT_BIG(*), 'N0') AS RecordCount_AfterLoad FROM dbo.DimCity         UNION ALL
    SELECT 'dbo'   AS SchemaName, 'DimCustomer'    AS TableName, FORMAT(@CountBeforeLoadDimCustomer,  'N0') AS RecordCount_BeforeLoad, FORMAT(COUNT_BIG(*), 'N0') AS RecordCount_AfterLoad FROM dbo.DimCustomer     UNION ALL
    SELECT 'dbo'   AS SchemaName, 'DimEmployee'    AS TableName, FORMAT(@CountBeforeLoadDimEmployee,  'N0') AS RecordCount_BeforeLoad, FORMAT(COUNT_BIG(*), 'N0') AS RecordCount_AfterLoad FROM dbo.DimEmployee     UNION ALL
    SELECT 'dbo'   AS SchemaName, 'DimStockItem'   AS TableName, FORMAT(@CountBeforeLoadDimStockItem, 'N0') AS RecordCount_BeforeLoad, FORMAT(COUNT_BIG(*), 'N0') AS RecordCount_AfterLoad FROM dbo.DimStockItem    UNION ALL
    SELECT 'dbo'   AS SchemaName, 'FactSale'       AS TableName, FORMAT(@CountBeforeLoadFactSale,     'N0') AS RecordCount_BeforeLoad, FORMAT(COUNT_BIG(*), 'N0') AS RecordCount_AfterLoad FROM dbo.FactSale
    ORDER BY
        SchemaName,
        TableName
    ```

1. Upon completion, compare the results with those in the screenshot and table below. The code in the executed cell performed several steps:

    1. Store the record count for each of the dimensional model tables before any data is transformed from stage -> dimensional model.
    1. Run all the five stored procedures to load the dimensional model tables (one procedure per table).
    1. Display the record count from before the data load and after the data load to ensure data was properly loaded.

    The *RecordCount_AfterLoad* numbers should match those of the stage table record counts produced after running the COPY INTO commands at the end of *Lab 3 - Loading data* indicating all the data was loaded from stage into the dimensional model.

    ![](../assets/images/04_incremental_load_results.png)

## Next steps
In this lab, you created several stored procedures that will be used to load data from the stage tables (medallion bronze layer) into the dimensional model (medallion silver layer). These procedures used a method for incremental loading where by existing records were updated if a key match was found and inserted if a key match was not found. You also created a surrogate key for each row inserted.

After creating the procedures you ran them to seed the initial dataset into the dimensional model. Next, you will see how to operationalize the data load and validate that the incremental loading logic works properly on subsequent runs. 

- Continue to [Lab 5 - Orchestrating warehouse operations](<05 - Orchestrating warehouse operations.md>)
- Return to the [workshop homepage](<../README.md>)

## Additional Resources
- [Surrogate keys](https://learn.microsoft.com/en-us/fabric/data-warehouse/dimensional-modeling-load-tables#surrogate-keys)
- [SCD type 1, 2, and 3](https://learn.microsoft.com/en-us/fabric/data-warehouse/dimensional-modeling-load-tables#scd-type-1)
- [Date dimension](https://learn.microsoft.com/en-us/fabric/data-warehouse/dimensional-modeling-load-tables#date-dimension)
- [Process fact tables](https://learn.microsoft.com/en-us/fabric/data-warehouse/dimensional-modeling-load-tables#process-fact-tables)