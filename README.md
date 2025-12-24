# 🏏 Cricket Data Analytics: Web Scraping & Power BI ETL

## 📊 Complete Data Engineering Project

### 📖 Project Overview
This project showcases an advanced technical solution for ETL (Extract, Transform, Load) operations, sourcing sports statistics from the official ESPN Cricinfo website. The main objective was to automate batting data capture using **dynamic web scraping** in Power Query and create a robust analytical model with **DAX**.

---

## 🔄 Workflow

### **1. Data Extraction (Web Scraping with M)**

**Pattern-Based Scraping**: Using `Web.BrowserContents` to render dynamic content and extraction via specific CSS selectors (`TABLE.engineTable:nth-child(5)`).

**Parameterized Custom Function**:
```m
(ps as text) =>
let
    Source = Web.BrowserContents("https://stats.espncricinfo.com/...page="&ps&"..."),
    #"Extracted Table From Html" = Html.Table(Source, {...}),
    #"Filtered Rows" = Table.SelectRows(#"Extracted Table From Html", each true),
    #"Removed Columns" = Table.RemoveColumns(#"Filtered Rows",{"Column16"})
in
    #"Removed Columns"
```
This function enables **automatic iteration** across multiple web pages using the `ps` parameter.

---

### **2. Robust Data Architecture**

**Compressed Headers**: Header table stored as compressed binary to ensure schema consistency:
```m
Table.FromRows(
    Json.Document(
        Binary.Decompress(
            Binary.FromText("...", BinaryEncoding.Base64), 
            Compression.Deflate
        )
    )
)
```

**Iterative Invocation**: List generation `{1..3}` to automatically call the function across multiple pages.

---

### **3. Transformation Pipeline (Data Cleaning)**

Comprehensive cleaning process in **15 sequential steps**:

| Transformation | M Technique | Purpose |
|----------------|-------------|---------|
| **Consolidation** | `Table.Combine({Headers, Query1})` | Schema and data union |
| **Promotion** | `Table.PromoteHeaders` | First row to headers conversion |
| **Strict Typing** | `Table.TransformColumnTypes` | Int64/Decimal conversion |
| **Null Cleaning** | `Table.ReplaceValue("-","0")` | Empty value replacement (15 iterations) |
| **Critical Localization** | `{{"Ave", type number}}, "en-US"` | Decimal handling independent of regional settings |

#### **Key Technical Note**: 
The `"en-US"` configuration in `Ave` and `SR` columns ensures the report is **Production-Ready** and works correctly on systems with Spanish/Latin American regional settings (where comma is used instead of decimal point). This prevents parsing errors when the report is shared across international teams.

---

## 📐 Data Model Output

### **Final Data After ETL:**
- **Volume**: 106 rows × 15 columns
- **Key Columns**: 
  - `Player` (text): Player name
  - `Mat` (integer): Matches played
  - `Inns` (integer): Innings batted
  - `NO` (integer): Not outs
  - `Runs` (integer): Total runs scored
  - `HS` (text): Highest score
  - `Ave` (decimal): Batting average
  - `BF` (integer): Balls faced
  - `SR` (decimal): Strike rate (runs per 100 balls)
  - `100` (integer): Centuries scored
  - `50` (integer): Half-centuries scored
  - `0` (integer): Ducks (dismissed for zero)
  - `4s` (integer): Fours hit
  - `6s` (integer): Sixes hit

### **Data Quality Achieved:**
✅ Strict data types (Int64 for counts, Decimal for averages)  
✅ Null values eliminated (converted to 0 for mathematical calculations)  
✅ Referential integrity guaranteed through validated schema  
✅ Culture-agnostic decimal handling for international deployment  

---

## 🧮 Featured DAX Measures

### **1. Weighted Average Runs**
```dax
Avg Runs Weighted = 
DIVIDE(
    SUM('Cricket_Data'[Runs]),
    SUM('Cricket_Data'[Inns]),
    0
)
```
**Purpose**: Calculates actual average runs per innings, handling division by zero.

---

### **2. Top Performers (Dynamic Ranking)**
```dax
Player Rank = 
RANKX(
    ALL('Cricket_Data'[Player]),
    [Total Runs],
    ,
    DESC,
    DENSE
)
```
**Purpose**: Generates dense ranking of players by total runs, ideal for Top N visualizations.

---

### **3. Categorized Strike Rate**
```dax
SR Category = 
SWITCH(
    TRUE(),
    'Cricket_Data'[SR] >= 140, "Aggressive",
    'Cricket_Data'[SR] >= 100, "Balanced",
    'Cricket_Data'[SR] >= 70, "Conservative",
    "Slow"
)
```
**Purpose**: Tactical segmentation of players by playing style.

---

### **4. Boundary Efficiency (Composite Measure)**
```dax
Boundary Efficiency % = 
VAR TotalBalls = SUM('Cricket_Data'[BF])
VAR BoundaryBalls = (SUM('Cricket_Data'[4s]) + SUM('Cricket_Data'[6s]))
RETURN
    DIVIDE(BoundaryBalls, TotalBalls, 0) * 100
```
**Purpose**: Advanced KPI measuring what percentage of balls faced resulted in boundary hits (4s or 6s).

---

### **5. Player Consistency (Standard Deviation)**
```dax
Player Consistency = 
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

## 📁 Project Architecture (PBIP vs PBIX)

### **Why PBIP Instead of PBIX?**

This project uses **PBIP (Power BI Project)** format for data engineering reasons:

✅ **Granular Version Control**: Git can track specific changes in JSON/plain text files  
✅ **Simultaneous Collaboration**: Multiple developers can work without merge conflicts  
✅ **Effective Code Review**: DAX and M changes are human-readable in pull requests  
✅ **Component Separation**: Report design and data model are stored independently  

### **Repository Structure:**
```
WebScraping_criquet/
├── .gitignore                          # Excludes Power BI temp files
├── WebScraping_criquet.pbip            # Main project file
├── WebScraping_criquet.pbix            # Compiled version (optional, for distribution)
├── WebScraping_criquet.Report/         # Visual definitions
└── WebScraping_criquet.SemanticModel/  # Data model and M transformations
    ├── definition/
    │   ├── tables/                     # Table definitions
    │   ├── relationships/              # Relationship definitions
    │   └── expressions/                # DAX measures
    └── queries/                        # Power Query M scripts
```

---

## 🛠️ Technical Stack

| Layer | Technology | Usage |
|-------|------------|-------|
| **Extraction** | M (Power Query) | Dynamic web scraping with parameterized functions |
| **Transformation** | M Language | 15-step ETL pipeline with data cleaning and type casting |
| **Modeling** | DAX | Calculated measures, rankings, and advanced KPIs |
| **Visualization** | Power BI Desktop | Interactive dashboards and reports |
| **Version Control** | Git + PBIP | Code change traceability and collaboration |

---

## 🎯 Key Technical Decisions

### **1. Modular M Architecture**
- Custom function with parameter (`ps as text`) for scalable pagination
- Binary-compressed headers to maintain schema integrity across iterations
- Sequential transformation pipeline (15 steps) for data quality assurance

### **2. Localization Strategy**
- Implemented `"en-US"` locale for decimal columns to ensure cross-regional compatibility
- Prevents parsing errors when reports are deployed in European or Latin American systems

### **3. Advanced DAX Analytics**
- Statistical measures (standard deviation) for player performance analysis
- Dynamic rankings using `RANKX` with `DENSE` mode
- Composite KPIs combining multiple metrics (boundary efficiency)

### **4. DataOps with PBIP**
- Decomposed binary format into version-controllable components
- Enabled collaborative development with Git-based workflows
- Facilitated code reviews of M queries and DAX measures

---

## 💡 Key Interview Talking Points

**When discussing this project, highlight:**

1. **"I implemented a modular ETL architecture in M"**  
   Parameterized function + compressed headers + sequential cleaning pipeline demonstrates understanding of scalable data engineering patterns.

2. **"I applied culture-agnostic data transformation"**  
   The `"en-US"` localization ensures production readiness across international deployments.

3. **"I used advanced DAX for statistical analysis"**  
   From dynamic rankings to standard deviation calculations, showcasing analytical depth beyond basic aggregations.

4. **"I chose PBIP for enterprise DataOps practices"**  
   Enables team collaboration, change tracking, and audit trails in data development workflows.

---

## 📊 Dashboard Preview



---

## 🚀 How to Use This Project

1. **Clone the repository**:
```bash
   git clone https://github.com/Patricoders23/WebScraping_criquet.git
```

2. **Open in Power BI Desktop**:
   - Double-click `WebScraping_criquet.pbip` to open the project
   - Alternatively, open the compiled `WebScraping_criquet.pbix` file

3. **Refresh Data**:
   - Click "Refresh" in Power BI to fetch the latest data from ESPN Cricinfo
   - Note: Web scraping may be rate-limited by the source website

4. **Explore the Model**:
   - Navigate to "Model View" to see table relationships
   - Check "Data View" to see the cleaned dataset
   - Review DAX measures in the "Measure" pane

---

## 📝 Future Enhancements

- [ ] Add incremental refresh for historical data
- [ ] Implement error handling for failed web requests
- [ ] Create predictive models for player performance
- [ ] Add more advanced visualizations (scatter plots, violin charts)
- [ ] Automate data refresh with Power BI Service/Gateway

---

## 📧 Contact

**Patricia García**  
Data Science & Analytics Professional | Petroleum Engineer  

🔗 LinkedIn: [linkedin.com/in/patri-data-engineering](https://www.linkedin.com/in/patri-data-engineering)  
💻 GitHub: [@Patricoders23](https://github.com/Patricoders23)  
📍 Location: Madrid, Spain  



---

## 🙏 Acknowledgments

- Data source: [ESPN Cricinfo](https://www.espncricinfo.com/)
- Power Query M documentation: [Microsoft Learn](https://learn.microsoft.com/en-us/powerquery-m/)
- DAX patterns: [SQLBI](https://www.sqlbi.com/)

---

*Last updated: December 2024*
