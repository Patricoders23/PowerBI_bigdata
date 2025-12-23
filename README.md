📖 Project Overview
This project showcases an advanced technical solution for ETL (Extract, Transform, Load) operations, sourcing sports statistics from the official ESPN Cricinfo website. The main objective was to automate batting data capture using dynamic web scraping in Power Query and create a robust analytical model with DAX.

🔄 Workflow
1. Data Extraction (Web Scraping with M)

Pattern-Based Scraping: Using Web.BrowserContents to render dynamic content and extraction via specific CSS selectors (TABLE.engineTable:nth-child(5)).
Parameterized Custom Function:

m(ps as text) =>
let
    Source = Web.BrowserContents("https://stats.espncricinfo.com/...page="&ps&"..."),
    #"Extracted Table From Html" = Html.Table(Source, {...}),
    ...
in
    #"Removed Columns"
This function enables automatic iteration across multiple web pages using the ps parameter.
2. Robust Data Architecture

Compressed Headers: Header table stored as compressed binary to ensure schema consistency:

mTable.FromRows(Json.Document(Binary.Decompress(Binary.FromText("...", BinaryEncoding.Base64), Compression.Deflate)))

Iterative Invocation: List generation {1..3} to automatically call the function across multiple pages.

3. Transformation Pipeline (Data Cleaning)
Comprehensive cleaning process in 15 sequential steps:
TransformationM TechniquePurposeConsolidationTable.Combine({Headers, Query1})Schema and data unionPromotionTable.PromoteHeadersFirst row to headers conversionStrict TypingTable.TransformColumnTypesInt64/Decimal conversionNull CleaningTable.ReplaceValue("-","0")Empty value replacementCritical Localization{{"Ave", type number}}, "en-US"Decimal handling independent of regional settings
Key Technical Note: The "en-US" configuration in Ave and SR columns ensures the report is Production-Ready and works correctly on systems with Spanish/Latin American regional settings (where comma is used instead of decimal point).

📐 Data Model Output
Final Data After ETL:

Volume: 106 rows × 15 columns
Key Columns:

Player (text): Player name
Mat (integer): Matches played
Runs (integer): Total runs
Ave (decimal): Batting average
SR (decimal): Strike Rate
100, 50, 0: Centuries, half-centuries, and ducks
4s, 6s: Four and six-run hits



Data Quality Achieved:
✅ Strict data types (Int64 for counts, Decimal for averages)
✅ Null values eliminated (converted to 0 for mathematical calculations)
✅ Referential integrity guaranteed through validated schema

🧮 Featured DAX Measures
1. Weighted Average Runs
daxAvg Runs Weighted = 
DIVIDE(
    SUM('Cricket_Data'[Runs]),
    SUM('Cricket_Data'[Inns]),
    0
)
Purpose: Calculates actual average runs per innings, handling division by zero.

2. Top Performers (Dynamic Ranking)
daxPlayer Rank = 
RANKX(
    ALL('Cricket_Data'[Player]),
    [Total Runs],
    ,
    DESC,
    DENSE
)
Purpose: Generates dense ranking of players by total runs, ideal for Top N visualizations.

3. Categorized Strike Rate
daxSR Category = 
SWITCH(
    TRUE(),
    'Cricket_Data'[SR] >= 140, "Aggressive",
    'Cricket_Data'[SR] >= 100, "Balanced",
    'Cricket_Data'[SR] >= 70, "Conservative",
    "Slow"
)
Purpose: Tactical segmentation of players by playing style.

4. Boundary Efficiency (Composite Measure)
daxBoundary Efficiency % = 
VAR TotalBalls = SUM('Cricket_Data'[BF])
VAR BoundaryBalls = (SUM('Cricket_Data'[4s]) + SUM('Cricket_Data'[6s]))
RETURN
    DIVIDE(BoundaryBalls, TotalBalls, 0) * 100
Purpose: Advanced KPI measuring what percentage of balls faced resulted in boundary hits (4s or 6s).

5. Player Consistency (Variance)
daxPlayer Consistency = 
VAR AvgScore = [Avg Runs Weighted]
VAR Variance = 
    SUMX(
        'Cricket_Data',
        POWER('Cricket_Data'[Runs] - AvgScore, 2)
    ) / COUNT('Cricket_Data'[Runs])
RETURN
    SQRT(Variance)
```
**Purpose**: Calculates standard deviation to identify players with predictable vs. volatile performance.

---

### 📁 Project Architecture (PBIP vs PBIX)

**Why PBIP Instead of PBIX?**

This project uses **PBIP (Power BI Project)** format for data engineering reasons:

✅ **Granular Version Control**: Git can track specific changes in JSON/plain text files  
✅ **Simultaneous Collaboration**: Multiple developers can work without merge conflicts  
✅ **Effective Code Review**: DAX and M changes are human-readable in pull requests  

**Repository Structure:**
```
WebScraping_criquet/
├── .gitignore                          # Excludes Power BI temp files
├── WebScraping_criquet.pbip            # Main project file
├── WebScraping_criquet.pbix            # Compiled version (distribution)
├── WebScraping_criquet.Report/         # Visual definitions
└── WebScraping_criquet.SemanticModel/  # Data model and M transformations

🛠️ Technical Stack
LayerTechnologyUsageExtractionM (Power Query)Dynamic web scraping with parameterized functionsTransformationM + Regex15-step ETL pipeline with data cleaningModelingDAXCalculated measures, rankings, and advanced KPIsVisualizationPower BI DesktopInteractive dashboardsVersion ControlGit + PBIPCode change traceability

💡 Key Interview Points
When asked about this project, highlight:

"I implemented a modular architecture in M": Parameterized function + compressed headers + sequential cleaning pipeline.
"I applied data localization (en-US)": Ensures the report works on any operating system without decimal parsing errors.
"I used DAX for advanced analytics": From dynamic rankings to statistical metrics like standard deviation.
"I chose PBIP for DataOps": Facilitates collaborative work and change traceability in enterprise data teams.
