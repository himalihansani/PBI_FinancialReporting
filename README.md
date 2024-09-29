# PBI_FinancialReporting
Financial Reporting in Power BI

CY_CM means Current Year Current Month 
---------------------------------------

CY_CM_Amount = 
VAR testvar = SELECTEDVALUE('PnLTitles'[Description])
// VAR ExportKG  = CALCULATE(
//                     SUM(ExportVolume[TOTALNETWEIGHT]),
//                     ExportVolume[SALESTYPE] = "EXPORT",
//                     USERELATIONSHIP(ExportVolume[DOCDATE],'Date'[Date]),
//                     ALL('PnLTitles'), 'PnLTitles'[Description] = "Total Export Volume - Kgs"
//                 )
VAR ExportKG = CALCULATE(
        SUM(ExportVolume[NETWEIGHTMINUSRETURNS]),
        USERELATIONSHIP(ExportVolume[DOCDATE],'Date'[Date]),
        USERELATIONSHIP(ExportVolume[COMPANYNAME],'PnL'[companyname]),
        ALL('PnLTitles')
    )
 
--USERELATIONSHIP(LocalKG_Volume[DOCDATE],'Date'[Date]),
// VAR LocalKG = CALCULATE(
//         SUM(LocalKG_Volume[NETWEIGHTMINUSRETURNS]),
//         USERELATIONSHIP(LocalKG_Volume[COMPANYNAME],'PnL'[companyname]),
//         ALL('PnLTitles')
//     )
 
VAR LocalKG =
    CALCULATE(
        IF(
            ISBLANK(SUM(LocalKG_Volume[NETWEIGHTMINUSRETURNS])),
            0,
            SUM(LocalKG_Volume[NETWEIGHTMINUSRETURNS])
        ),
        USERELATIONSHIP(LocalKG_Volume[COMPANYNAME],'PnL'[companyname]),
        ALL('PnLTitles')
    )
 
--VAR ExportsSales = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Exports Sales")
VAR ExportsSales =
    CALCULATE(
        [PnL_Amount],
        FILTER(
            ALL('PnLTitles'),
            'PnLTitles'[Description] = "Exports Sales" || 'PnLTitles'[Description] = "Export Sales" || 'PnLTitles'[Description] = "Revenue"
        )
    )
VAR testvar_localsalestea = SELECTEDVALUE('PnL'[accountcode])
VAR LocalSalesTea = CALCULATE(
                        [PnL_Amount],
                        USERELATIONSHIP('PnL'[docdate], 'Date'[Date]),
                        ALL('PnLTitles'),
                        'PnL'[display_title] = "Local Sales-Tea"
                    )
--VAR LocalSalesTea = CALCULATE(
                        --SUM('PnL'[amount]),
                        --USERELATIONSHIP('PnL'[docdate], 'Date'[Date]),
                        --ALL('PnLTitles'),
                        --'PnL'[SecondPartAccountCode] = 30201
                    --)
 
// VAR LocalSalesFoodBeverages = CALCULATE([PnL_Amount],
//                                          ALL('PnLTitles'),
//                                          'PnL'[SecondPartAccountCode] in { 30204,30216,30221, 30222, 30223 }
//                                         )
 
 // VAR LocalSalesFoodBeverages_Normal = CALCULATE([YTD],
//                                          ALL('PnLTitles'),
//                                          'PnL'[SecondPartAccountCode] in { 30204,30216,30221, 30222, 30223 }
//                                         )
 
// VAR LocalSalesFoodBeverages_MJFB = CALCULATE([YTD],ALL('PnLTitles'),
//                                 'PnL'[SecondPartAccountCode] in { 30201, 30202,30203,30204,30205,30206, 30207, 30208, 30209, 30210, 30211, 30212, 30213, 30214, 30215, 30216, 30221, 30222, 30223, 30324, 30326} )
 
--VAR LocalSalesFoodBeverages =
    --IF (
        --SELECTEDVALUE('PnL'[companyname]) = "MJF Beverages (Pvt) Ltd",
        --CALCULATE(
           -- [PnL_Amount],
            --ALL('PnLTitles'),
            --'PnL'[SecondPartAccountCode] IN { 30204, 30216, 30221, 30222, 30223, 30201, 30202, 30203, 30205, 30206, 30207, 30208, 30209, 30210, --30211, 30212, 30213, 30214, 30215, 30324, 30326 }
       -- ),
        --CALCULATE(
            --[PnL_Amount],
            --ALL('PnLTitles'),
            --'PnL'[SecondPartAccountCode] IN { 30204, 30216, 30221, 30222, 30223 }
       -- )
   -- )
VAR LocalSalesFoodBeverages =
    CALCULATE(
        [PnL_Amount],
        ALL('PnLTitles'),
        'PnL'[display_title] = "Local Sales" ||
        'PnL'[display_title] = "Local Sales-Food and Beverages" ||
        'PnL'[display_title] = "Locals Sales"
    )
 
VAR LocalSalesOthers =  CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[display_title] = "Locals Sales-Others")
VAR TotalTurnover = ExportsSales + LocalSalesTea + LocalSalesFoodBeverages+LocalSalesOthers
VAR SalesDiscountGiven = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Sales Discount Given") * -1
VAR NetTurnover = TotalTurnover + SalesDiscountGiven

VAR Cost_of_Good_Sold_Tea = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Cost of Good Sold - Tea") *-1
VAR Cost_of_Good_Sold_PM = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Cost of Good Sold - PM") * -1
VAR Cost_of_Good_Sold_Other = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Cost of Good Sold - Other") * -1
 
VAR Cost_of_Good_Sold_FoodBeverages = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[display_title] = "Cost of Good Sold - Local") * -1
VAR Cost_of_Good_Sold_Spices = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[SecondPartAccountCode] in {4010110}) * -1
VAR Cost_of_Good_Sold_LabourOH = CALCULATE([PnL_Amount], ALL('PnLTitles'),
                                           'PnLTitles'[Description] = "Cost of Good Sold - Labour & OH") * -1

VAR Overhead_Absorbtion = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Overhead Absorbtion") * -1
VAR TotalCostofGoodSold = Cost_of_Good_Sold_Tea + Cost_of_Good_Sold_PM + Cost_of_Good_Sold_Other 
                          + Cost_of_Good_Sold_FoodBeverages + Cost_of_Good_Sold_Spices+ Cost_of_Good_Sold_LabourOH 
                          + Overhead_Absorbtion

VAR GrossMargin = NetTurnover + TotalCostofGoodSold
VAR CostOfSales =  CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Cost of Sales") * -1
 
 
VAR Direct_Expense = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[display_title] = "Direct Expenses") * -1
VAR Factory_Overheads = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Factory Overheads") * -1
VAR Factory_Depreciation = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Factory Depreciation") * -1
VAR TotalDirectExpenses = Direct_Expense + Factory_Overheads + Factory_Depreciation
VAR TotalCostofSales = TotalCostofGoodSold + TotalDirectExpenses+ CostOfSales
VAR GrossProfitLoss = NetTurnover + TotalCostofSales
VAR Administration_Expenses = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Administration Expenses") * -1
VAR Donations = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Donations") * -1
VAR Administration_Depreciation = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Administration Depreciation") * -1
VAR Selling_Distribution_Expenses = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Selling & Distribution Expenses") * -1
VAR Finance_Charges = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Finance Charges") * -1
VAR TotalOverheads = Administration_Expenses+Administration_Depreciation+Selling_Distribution_Expenses+Finance_Charges+Donations
VAR OperatingProfit = GrossProfitLoss + TotalOverheads
 
VAR testvar_exchange = SELECTEDVALUE('PnL'[accountname])
VAR EXCHANGE_VARIATION_AP = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[accountname] = "EXCHANGE VARIATION-AP")
VAR EXCHANGE_VARIATION_AR = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[accountname] = "EXCHANGE VARIATION-AR")
VAR EXCHANGE_VARIATION_CASHBOOK = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[accountname] = "EXCHANGE VARIATION-CASH BOOKS")
VAR ExchangeGainLoss = EXCHANGE_VARIATION_AP + EXCHANGE_VARIATION_AR + EXCHANGE_VARIATION_CASHBOOK
VAR ProfitorLossonOperations = ExchangeGainLoss+ OperatingProfit
 
 
--VAR Interest_Income = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Interest Income")
 
--VAR testInterestIncome = SELECTEDVALUE('PnL'[accountcode])
--VAR Interest_Income = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[accountcode] in {"10-30401","10-30402","10-30403","10-30404","10-30405","10-30406","10-30407","10-30408","10-30409","10-30410","10-30411","10-30412","10-30413","10-30414","10-30415","10-30416"} )
 
--VAR Interest_Income = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[SecondPartAccountCode] >= 30401 && 'PnL'[SecondPartAccountCode] <= 30416 )
 
VAR Interest_Income =
    IF (
        MAX('PnL'[companyname]) = "City Properties (Pvt) Ltd",
        CALCULATE(
            [PnL_Amount],
            ALL('PnLTitles'),
            'PnL'[SecondPartAccountCode] <= 30416,
            'PnL'[SecondPartAccountCode] >= 30401,
            'PnL'[SecondPartAccountCode] <> 30414
        ),
        CALCULATE(
            [PnL_Amount],
            ALL('PnLTitles'),
            'PnL'[SecondPartAccountCode] <= 30416,
            'PnL'[SecondPartAccountCode] >= 30401
        )
    )
VAR OtherIncome = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[display_title] = "Other Income")
VAR RentIncome = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[display_title] = "Income Rent"|| 'PnL'[display_title] = "Income - Rent")
VAR Income = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[display_title] = "Income")
VAR IncomeDividend = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnL'[display_title] = "Income - Dividend")
VAR Sundry_Income = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Sundry Income")
VAR DividendIncomeNonGroup = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Dividend Income-Non-Group")
VAR DividendIncomeGroup = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Dividend Income-Group")
VAR TotalOtherIncome = Interest_Income + Sundry_Income + DividendIncomeNonGroup + DividendIncomeGroup +OtherIncome+RentIncome + Income +IncomeDividend
VAR NetProfitLossBeforeTax = ProfitorLossonOperations + TotalOtherIncome
VAR Income_Tax_C = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Income Tax (C)") * -1
VAR Income_Tax_E = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Income Tax (E)") * -1
VAR Income_Tax_F = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Income Tax (F)") * -1
VAR Income_Tax_G = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Income Tax (G)") * -1
VAR Income_Tax_ = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Income Tax") * -1
VAR Income_Tax = Income_Tax_C + Income_Tax_E + Income_Tax_F + Income_Tax_G + Income_Tax_
VAR NetProfitLossAfterTax = NetProfitLossBeforeTax + Income_Tax
VAR DividendPaid = CALCULATE([PnL_Amount], ALL('PnLTitles'), 'PnLTitles'[Description] = "Dividend Paid")
VAR NetProfitLoss_C_fwd = NetProfitLossAfterTax - DividendPaid
--VAR NetProfitLoss_C_fwd = NetProfitLossAfterTax
 
RETURN
SWITCH (
    TRUE (),
    testvar = "Total Export Volume - Kgs", ExportKG,
    testvar = "Total Local Volume - Kgs", LocalKG,
    testvar = "Exports Sales", ExportsSales,
    testvar = "Local Sales - Tea", LocalSalesTea,
    testvar = "Local Sales-Food & Beverages", LocalSalesFoodBeverages,
    testvar = "Local Sales - Others", LocalSalesOthers,
    testvar = "Total Turnover", TotalTurnover,
    testvar = "Sales Discount Given", SalesDiscountGiven,
    testvar = "Net Turnover", NetTurnover,      
    testvar = "Cost of Good Sold - Tea", Cost_of_Good_Sold_Tea,
    testvar = "Cost of Good Sold - PM", Cost_of_Good_Sold_PM,
    testvar = "Cost of Good Sold - Other", Cost_of_Good_Sold_Other,
    testvar = "Cost of Good Sold - Food & Beverages", Cost_of_Good_Sold_FoodBeverages,
    testvar = "Cost of Good Sold - Spices", Cost_of_Good_Sold_Spices,
    testvar = "Cost of Good Sold - Labour & OH", Cost_of_Good_Sold_LabourOH,
    testvar = "Overhead Absorbtion", Overhead_Absorbtion,
    testvar = "Total Cost of Good Sold", TotalCostofGoodSold,  
    testvar = "Gross Margin", GrossMargin,  
    testvar = "Direct Expense", Direct_Expense,
    testvar = "Factory Overheads", Factory_Overheads,
    testvar = "Factory Depreciation", Factory_Depreciation,
    testvar = "Total Direct Expenses", TotalDirectExpenses,
    testvar ="Cost of Sales", CostOfSales,
    testvar = "Total Cost of Sales", TotalCostofSales,
    testvar = "Gross Profit/(Loss)", GrossProfitLoss,  
    testvar = "Administration Expenses", Administration_Expenses,
    testvar ="Donations", Donations,
    testvar = "Administration Depreciation", Administration_Depreciation,
    testvar = "Selling & Distribution Expenses", Selling_Distribution_Expenses,
    testvar = "Finance Charges", Finance_Charges,
    testvar = "Total Overheads", TotalOverheads,
    testvar = "Operating Profit", OperatingProfit,
    testvar = "Exchange Gain/(Loss)", ExchangeGainLoss,
    testvar = "Profit or (Loss) on Operations", ProfitorLossonOperations,  
    testvar = "Interest Income", Interest_Income,
    testvar = "Income - Dividend", IncomeDividend,
    testvar = "Income", Income,
    testvar = "Income Rent", RentIncome,
    testvar = "Other Income", OtherIncome,
    testvar = "Sundry Income", Sundry_Income,
    testvar = "Dividend Income-Non-Group", DividendIncomeNonGroup,
    testvar = "Dividend Income-Group", DividendIncomeGroup,
    testvar = "Total Other Income", TotalOtherIncome,
    testvar = "Net Profit/(Loss) Before Tax", NetProfitLossBeforeTax,  
    testvar = "Income Tax", Income_Tax,
    testvar = "Net Profit/(Loss) After Tax", NetProfitLossAfterTax,
    testvar = "Dividend Paid",DividendPaid,
    testvar = "Net Profit/(Loss) C/fwd", NetProfitLoss_C_fwd    
)


New format in Below
--------------------

CY_CM_New_Format = 

VAR SubTotalActual =
    IF (
        COUNTROWS ( PnL_Block ) = 1,
        CALCULATE (
            IF (
                SELECTEDVALUE ( 'PnL'[category_type] ) = "Expenses",
                [PnL_Amount] * -1,
                [PnL_Amount]
            ),
            ALL ( 'PnL_Block' ),
            'PnL_Block'[Row Index] <= VALUES ( 'PnL_Block'[Row Index] )
        )
    )
VAR DoFilter =
    ISFILTERED ( 'PnL'[display_title] )
VAR SubTotal =
    MAX ( 'PnL_Block'[Subtotal] )
RETURN
    IF (
        AND ( DoFilter = TRUE (), SubTotal = 1 ),
        BLANK (),
        SWITCH (
            TRUE (),
            SELECTEDVALUE ( 'PnL_Block'[Subtotal] ) = 0, 
            IF (
                    SELECTEDVALUE ( 'PnL'[category_type] ) = "Expenses",
                    [PnL_Amount] * -1,
                    [PnL_Amount]
                ),
            SELECTEDVALUE ( 'PnL_Block'[Subtotal] ) = 1, 
            SubTotalActual,
            BLANK ()
        )
    )
