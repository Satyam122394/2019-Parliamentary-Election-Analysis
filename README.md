
# Analysis of 2019 Parliamentary Elections of India

The analysis aims to address some of the **contentious questions** regarding the electoral process and its outcomes as mentioned in the forthcoming section. It seeks to shed light on these issues by relying on data from the latest parliamentary general elections held in 2019. 

Ultimatey, the outcomes of the analysis are to be visualized on an interactive **Power BI dashboard** integrated with a map of constituencies and prepare a detailed final **Project report** which includes reasons and hihlights.

## Questions for Analysis

1. What is the **overall voter turnout rate** in elections and its break up among the genders? Are there reasons for a low voter turnout rate if any and what are its consequences? What reforms could enhance electoral participation?

2. Do **elected representatives** genuinely possess the majority **mandate of both the electorate and voters**? If not, what factors contribute to this, how does it impact the electoral outcomes?

3. What is the **gender composition** of the elected representatives and contestants? Does the Indian polity adequately represent all genders? If not, what are the reasons, how can it become more inclusive?

4. What is the **age distribution of the Indian politicians**, and what is the level of the youth participation in politics?

## Data Sources

Constituency Wise Details and Voter Turnout details of 2019 Elections from Election Commission of India website.
* https://eci.gov.in/files/file/13539-33-constituency-wise-detailed-result/
* https://eci.gov.in/files/file/13579-13-pc-wise-voters-turn-out/

Map files for Power BI dashboard are taken from
- https://github.com/datameet/maps/tree/master/parliamentary-constituencies

## Creating Mapfiles for Power BI

The map files from the source, contained the map of India divisioned into its electoral constituencies and following operations were carried out. 
- The shape files were loaded onto the **QGIS software** and the names of the constituencies were changed to remove duplicates and to match it with the names from the ECI website.
- The updated shape file has been saved from QSIS software, which created the following filetypes **.shp, .shx, .dbf, .prj, .qmd, .cpg**.
- The above files are imported into **https://mapshaper.org/** and reshaped the map to reduce the file size. The reduced map has been exported as **TopoJSON** file which can be used in Power BI readily.
  
## Data Cleaning & Transformation in Excel

The following operations were carried out on the excel files downloaded from the Election Commission of India website.
- Removed unnecessary header rows.
- Formatted the data as a table.
- Carried out a **vlookup** to verify whether names of the constituencies matches the names in database file (.dbf) of the updated map created from the QGIS.
- Changed the **data types** to percentages at relevant places.
  
## Importing Excel data into Power BI

The following excel files were imported into Power BI thorugh the data source option.
1. List of Constituencies created from the database file (.dbf) of the map.
2. Constituency Wise details data.
3. Gender wise Voter Turnout data.
   
## Data Cleaninng, Exploration and Transformation in Power Query

The below transformation operations were carried out
- The files are validaeted using the **Data Preview tools** in View ribbon.
- **Removed empty columns** from the tables.
- Created a copy of constituency wise details table and **aggregated the data based on the highest votes recieved** to create a table with only the details of winning candidates of every constituency as **constituency wise shares**

## Data Modelling

Created a data model by connecting the tables with each other looking like a **Snowflake Schema** with **Constituency Name** as the **Primary Key** and ensured that the model is **normalized** to the extent possible.

## DAX - Measures, Calculated Columns and Calculated Tables

### Calculated Tables

- Creted a table for the winning candidates from constituency wise details table

~~~
Winning Candidates = 
FILTER(
    'Constituency wise details',
    'Constituency wise details'[Winner] = "YES"
)
~~~

### Calculated Columns

- Calculated column created in Constituencies table to segreggate states and union territories.

~~~
STATE/UT = 
    IF(
        Constituencies[ST_NAME] IN {"DELHI", "ANDAMAN & NICOBAR", "CHANDIGARH", "DADRA & NAGAR HAVELI", "DAMAN & DIU", "LAKSHADWEEP", "PUDUCHERRY"},
     "UNION TERRITORY",
     "STATE"
    )
~~~

- Added columns in constituency wise details table to identify winners and group the age columns into bing for creating histogramns

~~~
Winner = 
IF(
    'Constituency wise details'[PC_NAME] = RELATED('Constituency wise shares'[PC_NAME]) &&
    'Constituency wise details'[ TOTAL ] = RELATED('Constituency wise shares'[HIGHEST VOTES]),
    "YES",
    "NO"
)
~~~

-  Added columns in constituency wise shares table to calculate vote share and electorate share from the vote count and grouped percentage bins columns for histogramns

~~~
WINNER'S ELECTORATE SHARE = 
'Constituency wise shares'[HIGHEST VOTES]/'Constituency wise shares'[TOTAL ELECTORS]

WINNER'S VOTE SHARE = 
'Constituency wise shares'[HIGHEST VOTES]/'Constituency wise shares'[TOTAL VOTES]
~~~

### Measures

A Mesasures table was created to store all the calculated measure.
The following measures were created

- **For Dynamic Headings**
~~~
Constituency Name Heading = 
    IF(
        HASONEVALUE(Constituencies[PC_NAME]),
        VALUES(Constituencies[PC_NAME]),
        "ALL CONSTITUENCIES"
    )

State Name Heading = 
IF(
    HASONEVALUE(Constituencies[ST_NAME]),
    VALUES(Constituencies[ST_NAME]),
    "ALL STATES AND UTS"
)
~~~

- **Voter Count by gender**
~~~
Female Voters = 
SUM(
    'Constituency wise voter turnout'[ FEMALE ]
)

Male Voters = 
SUM(
    'Constituency wise voter turnout'[ MALE ]
)

Third Gender Voters = 
SUM(
    'Constituency wise voter turnout'[ THIRD GENDER ]
)
~~~

- **Median age of the nominees and elected members**
~~~
Median Age of Nominations = 
MEDIAN(
    'Constituency wise details'[ AGE ]
)


Median age of winners = 
MEDIAN(
    'Winning Candidates'[ AGE ]
)
~~~

- **Count of states and UTs**
~~~
No. of States = 
CALCULATE(
    DISTINCTCOUNT(Constituencies[ST_NAME]),
    Constituencies[STATE/UT] = "STATE"
) + 0 


No. of UTs = 
CALCULATE(
    DISTINCTCOUNT(Constituencies[ST_NAME]),
    Constituencies[STATE/UT] = "UNION TERRITORY"
) + 0
~~~


- **Ages of oldest and youngest winners**
~~~
Oldest Winner = 
MAX(
    'Winning Candidates'[ AGE ]
)

Youngest Winner = 
MIN(
    'Winning Candidates'[ AGE ]
)
~~~

- **Total Electorate and voters**
~~~
Total Electorate = 
SUM(
    'Constituency wise shares'[TOTAL ELECTORS]
)


Total Voters = 
SUM(
    'Constituency wise shares'[TOTAL VOTES] 
)
~~~

- **Vote share received by the contestants by gender**
~~~
Vote Share of Men = 
[Votes to Men]/[Total Voters]


Vote Share of Third Gender = 
[Votes to Third Gender]/[Total Voters]


Vote Share of Women = 
[Votes to Women]/[Total Voters]
~~~

- **Voter Turnout**
~~~
Voter Turnout = 
[Total Voters]/[Total Electorate]
~~~

- **Votes received by contestants of each Gender**
~~~
Votes to Men = 
SUMX(
    FILTER(
        'Constituency wise details',
        'Constituency wise details'[ SEX ] = "MALE"),
    'Constituency wise details'[ TOTAL ]
)+0


Votes to Third Gender = 
SUMX(
    FILTER(
        'Constituency wise details',
        'Constituency wise details'[ SEX ] = "THIRD"),
    'Constituency wise details'[ TOTAL ]
)+0


Votes to Women = 
SUMX(
    FILTER(
        'Constituency wise details',
        'Constituency wise details'[ SEX ] = "FEMALE"),
    'Constituency wise details'[ TOTAL ]
)+0
~~~

- **Elected members mandate**
~~~
Wins with below 50% Electorate Share = 
COUNTROWS(
FILTER(
    'Constituency wise shares',
    'Constituency wise shares'[WINNER'S ELECTORATE SHARE] < 0.5
)) + 0


Wins with below 50% Voters = 
COUNTROWS(
FILTER(
    'Constituency wise shares',
    'Constituency wise shares'[WINNER'S VOTE SHARE] < 0.5
)) + 0
~~~

## Visualization

The report was divided into three pages as shown in the attached **"Power BI Dashboard Screenshots"** file. In addition to this, a screenrecodring has been attached for better appreciation of the features of the dashboard.

The following features were used in the visualizations
- Controlled Interactions
- Hierarchials drilling of data up and down 
- Calculated y-axis range for the bar and column charts
- Custom maps
- Custom tooltips
- Page Navigation buttons
- Conditionally formatted colors at relevant places
- Dynamic headings and text boxes
- Bookmarks and buttons

## Analysis Outcomes

Please refer Analysis report for the outcomes of the ananlysis
