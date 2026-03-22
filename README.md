# HarvestGuard-Africa

HarvestGuard Africa is a predictive analytics project analyzing climate, crop yield, and food security data across five African countries (2013–2024, with projections to 2027). It delivers insights via a Power BI dashboard to help decision-makers identify and act on climate-driven food security risks.

**The Five Datasets**

Five publicly available datasets were sourced, downloaded as CSV files, and consolidated into a single Microsoft Excel workbook:

**Dataset	Source	What It Contributed**

Staple Crop Yield (Maize, Rice, Cassava, Millet, Wheat)	FAOSTAT fao.org/faostat	Annual yield in kg/ha per country per crop 2013–2024. Baseline for Yield_Decline_% and FORECAST.LINEAR predictions

Rainfall & Temperature	World Bank Climate Portal climateknowledgeportal.worldbank.org	Annual rainfall (mm) and mean temperature (°C) per country. Used for Rainfall_Deficit_% and Temp_Deficit_%

Vegetation Health Index (VHI)	FAO GIEWS fao.org/giews/earthobservation	Dekadal (10-day) VHI score per country. Averaged to annual VHI_Score for Climate_Stress_Score formula

Food Security (Undernourishment)	FAO Food Security Data fao.org/faostat	Number of people undernourished (millions) per country per year. Used as Food_Security_Score in risk formula

Malnutrition (Stunting Prevalence)	UNICEF data.unicef.org/topic/nutrition	Stunting prevalence % per country per year. Used as Malnutrition_Rate_% in risk formula

**The Excel COMBINE MASTER**

All five datasets were cleaned and merged into a single COMBINE MASTER sheet in Microsoft Excel using the following step-by-step process:

Data Cleaning

•	Removed all redundant metadata columns from each CSV file (database codes, footnote flags, API reference fields)
•	Standardised country name spellings across all five sheets to enable consistent XLOOKUP and SUMPRODUCT matching
•	Filtered crop yield data to five target crops only: Maize, Rice, Cassava, Millet, Wheat
•	Filtered all datasets to the 2013–2024-time range for consistency
•	Used Power Query (Get & Transform) to unpivot the World Bank climate data from wide format (one column per year) to long format (one row per year) - matching the structure of the other datasets

The Seven Calculated Columns

Seven new columns were added to the COMBINE MASTER sheet, each building on the previous:

Column	Formula and Purpose

Rainfall_Deficit_%	= ((Rainfall_mm - AVERAGEIF (Country, [@Country], Rainfall_mm)) / AVERAGEIF (Country, [@Country], Rainfall_mm)) * 100 Measures how far each year's rainfall deviates from that country's own historical average. Negative = drought stress. Enables fair cross-country comparison.

Yield_Decline_%	= ((Yield_kg_ha - XLOOKUP (Country&"-"&Crop&"-2013", Key, Yield_kg_ha, 0)) / XLOOKUP(...)) * 100 Measures percentage change in crop yield against the 2013 baseline. Negative = production declining.

VHI_Score	=IFERROR(SUMPRODUCT((VHI_Sheet!Year=[@Year]) * VALUE(VHI_Sheet!VHI_SCORE)) / SUMPRODUCT((VHI_Sheet!Year=[@Year]) * 1), 0) Annual average Vegetation Health Index per country. SUMPRODUCT used to handle text-format VHI values without array entry.

Food_Security_Score	=IFERROR(SUMPRODUCT((Food_Security!Country=[@Country]) * (Food_Security!Year=[@Year]) * (Food_Security!Value)), 0) Number of undernourished people (millions) per country per year. SUMPRODUCT matches on both Country and Year simultaneously.

Malnutrition_Rate_%	=IFERROR(SUMPRODUCT((Stunting!Country=[@Country]) * (Stunting!Year=[@Year]) * VALUE(Stunting!Value)), 0) Stunting prevalence % per country per year from UNICEF data.

Climate_Stress_Score	=(ABS(Rainfall_Deficit_%) * 0.40) + (ABS(Temp_Deficit_%) * 0.35) + ((1 - VHI_Score) * 100 * 0.25) Weighted composite of three climate variables. Weights: Rainfall 40%, Temperature 35%, VHI 25%. Higher score = greater agricultural climate stress.

Food_Insecurity_Risk_Score	=(ABS(Percentage_CHG) * 0.40) + (Food_Security_Score * 0.35) + (Malnutrition_Rate_% * 0.25) Weighted composite of yield decline, food security, and malnutrition. Weights: Yield 40%, Food Security 35%, Malnutrition 25%. Higher score = greater food insecurity risk.

Crop_Risk_Flag	=IF(N2>60, "HIGH RISK", IF(N2>35, "MEDIUM RISK", "LOW RISK")) Converts the Food_Insecurity_Risk_Score into a plain-language decision label with conditional colour formatting: Red = HIGH, Orange = MEDIUM, Green = LOW.

**The PREDICTIONS Sheet**

A separate PREDICTIONS sheet was built using Excel's FORECAST.LINEAR function to project staple crop yields forward to 2025, 2026, and 2027 for all country-crop combinations. FORECAST.LINEAR applies ordinary least squares linear regression to the 12-year historical yield series (2013–2024) and extrapolates the identified trend forward. The PREDICTIONS sheet contains: predicted yield (kg/ha), actual 2024 yield, yield change versus 2024, percentage change versus 2024, and the FORECAST.LINEAR formula made visible in a dedicated reference column for full transparency[10]

=IFERROR (FORECAST.LINEAR(Predicted_Year, 'CROP YIELD’! $D$first: $D$last, 'CROP YIELD’! $C$first: $C$last), 0)

**The Power BI Dashboard**
The completed COMBINE MASTER and PREDICTIONS sheets were imported into Microsoft Power BI Desktop and connected as the data source for the interactive dashboard. The dashboard comprises three components:

Six KPI Cards

KPI Card	Value	Significance

Highest Risk Country -	Nigeria.	The country with the highest Food Insecurity Risk Score - the most urgent intervention priority

Average Climate Stress Score - 18.21.	The region-wide average climate burden on agriculture - a baseline for comparison across all countries

Country at high crop risk Flag- Nigeria. Only Nigeria currently classified HIGH RISK - confirms the crisis is concentrated and actionable

Worst Rainfall Deficit (%)	- (-39.87%)	The most extreme single-season rainfall shortage recorded - confirms climate volatility is already at crisis level in parts of Africa

Predicted Yield Change by 2027 -	(+3.45%)	The overall average yield trajectory - positive but masking serious crop-specific declines in individual countries

Most Affected Crop - 	Millet	The crop with the greatest projected decline - a critical finding given Millet's role as a primary subsistence crop for smallholder farmers

Six Charts

Chart	Type	What It Reveals


1	Rainfall Deficit Trend Over Time	Line Chart - Tracks rainfall deficit for all 5 countries 2013–2024. Reveals Uganda's extreme spike and the varying climate volatility patterns across the region

2	Average Climate Stress Score by Country	Bar Chart	Ranks all 5 countries by climate stress score. Identifies which countries face the highest agricultural climate burden

3	Food Insecurity Risk Score by Country	Column Chart	Ranks countries by Food Insecurity Risk Score. Directly identifies which country requires the most urgent food security intervention

4	Crop Risk Flag Summary Table	Table	Shows HIGH/MEDIUM/LOW RISK classification for every country-crop combination. A decision-ready action list for governments and food agencies

5	Predicted Yield by Country 2025–2027	Grouped Bar Chart	Shows projected yields for all 5 countries across 3 forecast years. Reveals which countries face production decline and which show resilience

6	Historical vs Predicted Yield	Combo Line Chart	Overlays actual historical yields with FORECAST.LINEAR projections. Shows where predicted trends diverge from recent performance

Country Button Slicer

A Country Button Slicer displays all five countries as clickable buttons connected to all six charts simultaneously. Selecting any country instantly filters the entire dashboard, allowing decision-makers to explore a single country's full climate and food security story in real time.


Please find attached link to powerbi dashboard 
https://uhasedughana-my.sharepoint.com/:u:/g/personal/mhomenu26pg_sph_uhas_edu_gh/IQD9kLCyH2cPSpjLtuuEyUW3AUpSdZngJCaWW4egvxr0Bbo?e=Sf0ATL
