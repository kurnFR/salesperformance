# Tableau Formula Extract (reference only)

Extracted from `Analisa Kinerja Sales mockup.twbx` (the INA path). This is **evidence of how the current Tableau dashboard calculates things**, not the final specification. Where this file and `docs/01-metrics-contract.md` differ, the contract wins; differences are logged in `docs/decisions.md`.

Notes: names like `Calculation_802485210379128833` are Tableau internal IDs. `ZN()` converts null to 0. `LOOKUP(x, -12)` reads the value 12 months earlier in the table. `{FIXED ...}` is a level-of-detail aggregation. `Parameter 8` is the Start_Date (month) parameter; `Start_Date (copy)_971...` is End_Date.

The workbook contains 100+ worksheets, including ASW variants; only the INA formulas relevant to the four reference sheets are listed.


## Period, rolling window and working days

### Start_Date

```
#2026-10-01#
```

### End_Date

```
#2026-10-01#
```

### Today

```
today()
```

### rolling month

```
IF DATEPART('year', [monthyear]) < DATEPART('year', [Parameters].[Parameter 8]) THEN

    // Tahun sebelumnya → rolling
    IF DATEPART('month', [monthyear]) <= DATEPART('month', [Parameters].[Parameter 8]) THEN
        DATEPART('month', [monthyear]) + 12
    ELSE
        DATEPART('month', [monthyear])
    END

ELSE

    // Tahun yang sama → normal
    DATEPART('month', [monthyear])

END
```

### filter this year month

```
//DATEPART('year', [monthyear]) = DATEPART('year', TODAY()) 
//AND 
//DATEPART('month', [monthyear]) <= DATEPART('month', TODAY())

DATEPART('year', [monthyear]) = DATEPART('year', [Parameters].[Parameter 8])

OR
(
    DATEPART('year', [monthyear]) = DATEPART('year', [Parameters].[Parameter 8]) - 1
)
```

### DisplayFilter

```
(
    DATEDIFF(
        'month',
        DATETRUNC('month', LOOKUP(MIN([monthyear]), 0)),
        DATETRUNC('month', [Parameters].[Parameter 8])
    ) >= 0
    AND
    DATEDIFF(
        'month',
        DATETRUNC('month', LOOKUP(MIN([monthyear]), 0)),
        DATETRUNC('month', [Parameters].[Parameter 8])
    ) <= 6
)
OR
(
    DATEDIFF(
        'month',
        DATETRUNC('month', LOOKUP(MIN([monthyear]), 0)),
        DATETRUNC('month', DATEADD('year', -1, [Parameters].[Parameter 8]))
    ) >= 0
    AND
    DATEDIFF(
        'month',
        DATETRUNC('month', LOOKUP(MIN([monthyear]), 0)),
        DATETRUNC('month', DATEADD('year', -1, [Parameters].[Parameter 8]))
    ) <= 6
)
```

### Label Growth

```
DATEDIFF(
    'month',
    DATETRUNC('month', LOOKUP(MIN([monthyear]), 0)),
    DATETRUNC('month', [Parameters].[Parameter 8])
) >= 0
AND
DATEDIFF(
    'month',
    DATETRUNC('month', LOOKUP(MIN([monthyear]), 0)),
    DATETRUNC('month', [Parameters].[Parameter 8])
) <= 6
```

### Filter until closing month

```
IF [monthyear] <= [Calculation_971370148216811524] -1 THEN 1 ELSE 0 END
```

### MTD days

```
//DATEDIFF('day', [Start_Date], TODAY())   // total days so far
//- INT( (DATEDIFF('day', [Start_Date], TODAY()) + DATEPART('weekday', [Start_Date])) / 7 )
//- IF DATEPART('weekday', TODAY()) = 1 THEN 1 ELSE 0 END

// [MTD Working Days]
IF DATETRUNC('month', [Parameters].[Parameter 8]) = DATETRUNC('month', TODAY()) THEN
    SUM(
        IF [dates] <= TODAY() THEN [networkeddays] END
    )-1
ELSE
    SUM([networkeddays])
END
```

### % WorkingMTD

```
// % of working days completed
[Calculation_802485210379128833]
/ 
SUM([networkeddays])
```

### Color Working Days

```
IF [Calculation_802485210386206722] = 0 THEN 0.0001
ELSE [Calculation_802485210386206722]
END
```

### LY and CY Filter

```
(
    //Last year period: from Jan last year until current month last year
    [years] = DATEPART('year', TODAY()) - 1
    AND DATEPART('month', [monthyear]) <= DATEPART('month', TODAY())
)
OR
(
    //This year period: from Jan this year until current month this year
    [years] = DATEPART('year', TODAY())
    AND DATEPART('month', [monthyear]) <= DATEPART('month', TODAY())
)
```


## Achievement and colors (Sales Performance)

### %Capai SellIN

```
SUM([TOTALRealSellIn])/SUM([TOTALTargetSellIn])
```

### %Capai RSM,ASM SellIN

```
SUM([TOTALRealSellIn])/SUM([TOTALTargetSellIn])
```

### %Capai RSM,ASM ColorSellIN

```
IF [%Capai RSM,ASM SellIN (copy)_682576860310437889] >= 1 
  OR [%Capai RSM,ASM SellIN (copy)_682576860310437889] >= [Calculation_802485210386206722] THEN 
    "GREEN"
ELSEIF [%Capai RSM,ASM SellIN (copy)_682576860310437889] > 0.7 THEN 
    "YELLOW"
ELSE 
    "RED"
END
```

### #%Capai RSM Sell IN

```
{FIXED [monthyear],[REGIONNAME]:
if [%Capai SPV SellOut (copy)_1548112413979332608] >= 1 then 1 else 0 END
}
```

### #%Capai ASM Sell IN

```
{FIXED [monthyear],[BRANCHNAME]:
if [%Capai SPV SellOut (copy)_1548112413979332608] >= 1 then 1 else 0 END
}
```

### #%Capai SPV Sell IN

```
{FIXED [monthyear],[NAMAAREA]:
if [%Capai SPV SellOut (copy)_1548112413979332608 1] >= 1 then 1 else 0 END
}
```

### #%Capai Dist Sell IN

```
{FIXED [monthyear],[SELLINGPOINTNAME]:
if [%Capai SPV SellOut (copy)_1548112413979332608] >= 1 then 1 else 0 END
}
```

### #%Capai Dist Sell IN WD

```
{FIXED [monthyear],[SELLINGPOINTNAME]:
if [%Capai SPV SellOut (copy)_1548112413979332608] >= ([Calculation_802485210386206722]) then 1 else 0 END
}
```

### %RSM CapaiTarget Sell IN

```
SUM([#%Capai ASM Sell IN (copy)_491736849075879936])/(COUNTD([BRANCHNAME])*(DATEDIFF('month',DATE([Parameters].[Parameter 8]),DATE([Parameters].[Start_Date (copy)_971370148214833154]))+1))
```

### %ASM CapaiTarget Sell IN

```
SUM([#%Capai SPV Sell IN (copy)_1833246573115891714])/(COUNTD([BRANCHNAME])*(DATEDIFF('month',DATE([Parameters].[Parameter 8]),DATE([Parameters].[Start_Date (copy)_971370148214833154]))+1))
```

### %SPV CapaiTarget Sell IN

```
SUM([#%Capai ASM Sell IN (copy)_1531786845041741824])/(COUNTD([NAMAAREA])*(DATEDIFF('month',DATE([Parameters].[Parameter 8]),DATE([Parameters].[Start_Date (copy)_971370148214833154]))+1))
```

### %Dist CapaiTarget Sell IN WD

```
SUM([#%Capai Dist Sell IN (copy)_802485210475499539])
/
(COUNTD([SELLINGPOINTNAME])*(DATEDIFF('month',DATE([Parameters].[Parameter 8]),DATE([Parameters].[Start_Date (copy)_971370148214833154]))+1))
```


## Growth dashboards

### CM TotalRealSellIN Fix RSM

```
{FIXED [REGIONNAME],[Parameters].[Parameter 8] :
SUM(IF YEAR([monthyear]) = YEAR([Parameters].[Parameter 8]) 
AND MONTH([monthyear]) = MONTH([Parameters].[Parameter 8])
THEN [TOTALRealSellIn]
END)
}
```

### CM LY TotalRealSellIN Fix RSM

```
{FIXED [Calculation_304274452651835392],[Parameters].[Parameter 8] :
SUM(IF YEAR([monthyear]) = YEAR([Parameters].[Parameter 8]) - 1
AND MONTH([monthyear]) = MONTH([Parameters].[Parameter 8])
THEN [TOTALRealSellIn]
END)
}
```

### %Growth SellIN CM LY Fix RSM

```
SUM([CM TotalRealSellIN Fix ASM (copy)_2186497666772643840])/SUM([CM LY TotalRealSellIN Fix ASM (copy)_1833246573207883800])-1
```

### Growth Real Sell IN LYM VS CM over

```
(ZN(SUM([TOTALRealSellIn])) - LOOKUP(ZN(SUM([TOTALRealSellIn])), -12))
/LOOKUP(ZN(SUM([TOTALRealSellIn])), -12)
```

### LY RealIN

```
ZN(LOOKUP(SUM([TOTALRealSellIn]), -12))
```

### LY TargetRealIN

```
ZN(LOOKUP(SUM([TOTALTargetSellIn]), -12))
```

### %Capai ColorSellIN -1

```
IF [%Capai SellIN CM (copy)_632474323002138624] >= 1
   OR SUM([TOTALRealSellIn]) >= LOOKUP(SUM([TOTALRealSellIn]), -1)
THEN
    "GREEN"

ELSEIF [%Capai SellIN CM (copy)_632474323002138624] < 1
    AND MIN(DATEPART('year', [monthyear])) = DATEPART('year', TODAY())
THEN
    "YELLOW"

ELSE
    "RED"
END
```

### %Capai ColorSellIN LY

```
IF [Calculation_613334005326503939] >= [LY RealIN (copy)_613334005333868549] THEN 
    "GREEN"
ELSE 
    "RED"
END
```

### %Capai ColorSellOut

```
IF [Calculation_1278740820555309056] >= 1 THEN 
    "GREEN"
ELSEIF SUM([TOTALRealSellOut]) >= [LY RealIN (copy)_243475886122024962] THEN
    "GREEN"
ELSE 
    "YELLOW"
END
```

### Growth Real Sell IN LYM VS CM over Dynm Pos

```
ZN(
    IF [Growth Real Sell IN MoM (copy)_319474128433717248] > 0 THEN
        [Growth Real Sell IN MoM (copy)_319474128433717248]
    END
)
```

### Growth Real Sell IN LYM VS CM over Dynm Neg

```
ZN(
    IF [Growth Real Sell IN MoM (copy)_319474128433717248] < 0 THEN
        [Growth Real Sell IN MoM (copy)_319474128433717248]
    END
)
```


## Team counts and active distributors

### SE Excl SOA

```
IF CONTAINS([SLSNO], "SOA") THEN NULL ELSE [SLSNO] END
```

### SE Excl SOI

```
IF CONTAINS([SLSNO], "SOI") THEN NULL ELSE [SLSNO] END
```

### %AktifDist

```
SUM([Total Aktif])/SUM([TOTAL DIST])
```

### month filter dist aktif

```
//DATEPART('month',date([Select Tgl]))=[BULAN (Target RSM,ASM,SPV)]

IF ISNULL(DATETRUNC('month', [Bulan]))
OR ISNULL([Parameters].[Parameter 8]) 
OR DATETRUNC('month', DATE([Parameters].[Parameter 8])) = DATETRUNC('month', DATE(DATETRUNC('month', [Bulan])))
THEN TRUE
ELSE FALSE
END
```


## Daily upload

### color upload yes

```
IF ATTR({FIXED [KOTA]: MAX([Updated])} = 1) THEN "Upload"
ELSE "Not Upload"
END
```

### updated status

```
IF ISNULL([Updated]) THEN "NULL" ELSE "NOT NULL" END
```

### Update ZN

```
IF ISNULL([Updated]) THEN 0
ELSE 1
END
```

### color upload stok yes

```
IF ATTR({FIXED [KOTA (DailyUploadStockEskaINA.csv)]: MAX([Updated (DailyUploadStockEskaINA.csv)])} = 1) THEN "Upload"
ELSE "Not Upload"
END
```

### Month Filter Dist

```
//DATEPART('month',date([Select Tgl]))=[BULAN (Target RSM,ASM,SPV)]

IF ISNULL(DATETRUNC('month', [dates]))
OR ISNULL([Parameters].[Parameter 8]) 
OR DATETRUNC('month', DATE([Parameters].[Parameter 8])) = DATETRUNC('month', DATE(DATETRUNC('month', [dates])))
THEN TRUE
ELSE FALSE
END
```


## Filters

### Principal Filter

```
[principal]=[Parameters].[Parameter 3]
```

### Region Filter

```
[Parameters].[Parameter 1] = [REGIONNAME]
OR [Parameters].[Parameter 1] = "All"
```

### Area Filter

```
[Parameters].[Parameter 2] = [BRANCHNAME]
OR [Parameters].[Parameter 2] = "All"
```
