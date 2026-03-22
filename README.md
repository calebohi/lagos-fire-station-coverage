# Lagos Fire Station Coverage Analysis
## 📌 Overview
This project presents a spatial analysis of fire station coverage in Lagos using GIS, highlighting underserved areas and estimating the population outside a 5 km service radius.

Rapid access to fire services is critical in urban environments like Lagos. This project evaluates the accessibility of fire stations across the state by identifying areas beyond a 5 km radius of existing stations.

By combining spatial analysis with population data, the study highlights regions where emergency response coverage may be limited and estimates the number of people potentially affected.

## 🎯 Objectives
- Assess fire station accessibility across Lagos State
- Identify areas beyond a 5 km coverage radius
- Estimate the population located in uncovered areas
- Highlight spatial patterns of underserved regions

## 🗂️ Data Sources
- **Population Data:** WorldPop (2026 population estimates for Nigeria)  
  https://hub.worldpop.org/geodata/summary?id=74736  

- **Administrative Boundaries:** HDX (OCHA) – Nigeria Administrative Boundaries  
  https://data.humdata.org/dataset/cod-ab-nga  

- **Fire Station Locations:**  
  - OpenStreetMap (accessed via QuickOSM plugin in QGIS)  
  - Supplemented with manually collected locations from Google Maps and Google Earth Pro  

All datasets used in this project are included in the `/data` folder to support reproducibility.

## ⚙️ Methodology
### 1. Data Collection

Population data for Nigeria was obtained from the WorldPop database (2026 estimates). Administrative boundary data was sourced from HDX (OCHA), from which the Lagos State boundary and its Local Government Areas (LGAs) were extracted for the analysis.

Fire station locations were obtained from multiple sources to improve completeness and accuracy. OpenStreetMap data was accessed using the QuickOSM plugin in QGIS by querying features with the tag `amenity = fire_station` within the Lagos extent.

Additional fire station locations were identified and manually collected from Google Maps and Google Earth Pro to supplement and validate the OpenStreetMap dataset.
###  2. Data Preparation

The Lagos State boundary and its Local Government Areas (LGAs) were extracted from the national administrative boundary dataset obtained from HDX by querying for Lagos-specific features in the attribute table and exporting the selected features as a new layer.

All datasets were reprojected to a projected coordinate reference system (UTM Zone 31N) to ensure accuracy in distance-based analysis and maintain spatial consistency across all datasets.

Fire station data obtained from multiple sources required preprocessing before integration. OpenStreetMap features were retrieved as polygon geometries (ways), which were converted to point features using the **Centroid tool**.

Fire station locations collected from Google Maps were initially in CSV format. These were imported into QGIS and converted into a vector layer by exporting them as a GeoPackage file using the “Save Features As” function.

All fire station datasets were merged into a single unified layer using the “Merge Vector Layers” tool.

To remove duplicate entries, a spatial cleaning approach was applied. The merged layer was processed using the “Snap Geometries to Layer” tool with an appropriate tolerance (in meters) to align closely located points. This was followed by the “Delete Duplicate Geometries” tool, which removed overlapping or identical features, ensuring that each fire station was represented only once in the final dataset.

The population raster (WorldPop 2026 estimates) was clipped to the Lagos State boundary using the “Clip Raster by Mask Layer” tool, with the output projected to UTM Zone 31N to align with other datasets.
###  3. Accessibility Analysis (Buffer)

To assess fire station accessibility, a buffer analysis was performed around all fire station locations.

The “Buffer” tool in QGIS was used to generate a fixed-distance buffer of 5 km (5000 meters) around each fire station point, representing an approximate service coverage radius.

A segment value of 20 was used to produce smoother and more accurate circular buffers. The “Dissolve result” option was enabled to merge overlapping buffer zones into a single continuous coverage area, ensuring that shared service regions were not counted multiple times.

The resulting buffer layer represents areas with potential access to fire station services, while regions outside the buffer were considered underserved and used for further population analysis.

![Buffer Analysis](output/buffer_analysis.png)

### 4. Population Analysis
#### 4.1 Identifying Uncovered Areas

To determine areas without fire station coverage, the “Difference” tool in QGIS was used.

The Lagos State boundary was set as the input layer, while the 5 km buffer layer was used as the overlay. This operation subtracts the buffer (covered areas) from the Lagos boundary, resulting in polygons representing areas outside fire station coverage.

![Uncovered Areas - Difference Tool](output/difference_tool.png)
#### 4.2 Extracting Population Values

To quantify the population within uncovered areas, the “Zonal Statistics” tool was applied.

The uncovered areas layer was used as the input (zones), while the lagos population count raster (2026 estimates) served as the raster layer. The “Sum” statistic was selected to calculate the total population within each polygon.

This process generated a new attribute field containing population values for uncovered areas within each Local Government Area (LGA), enabling a spatial assessment of population exposure to limited fire station coverage.

![Zonal Statistics](output/zonal_statistics.png)

#### 4.3 Filtering Minimal Uncovered Areas

Some LGAs contained very small uncovered regions that were not significant enough to influence overall accessibility patterns. To avoid misrepresenting these areas and to maintain clarity in the analysis, LGAs with minimal uncovered coverage were excluded.

A threshold of **2 km²** was used to define significant uncovered areas.

To implement this:

- The **Field Calculator** tool was used to compute the area of uncovered regions.
- A new field named `area_km²` was created.
- The following expression was applied:

  `round($area / 1000000, 2)`

This converts area from square meters to square kilometers and rounds the values to two decimal places.

LGAs with uncovered areas less than or equal to 2 km² were classified as minimal and excluded from the main analysis. This step ensures that the analysis focuses on areas with meaningful service gaps rather than minor spatial artifacts.
#### 4.4 Population Classification and Mapping

To effectively communicate the spatial distribution of populations outside fire station coverage, the results from the zonal statistics were classified into meaningful population ranges.

A rule-based symbology approach was applied in QGIS to group LGAs based on the number of people living outside the 5 km fire station service radius. The classification thresholds were defined as follows:

- 100,000 – 250,000 people  
- 250,000 – 500,000 people  
- 500,000 – 1,000,000 people  
- Greater than 1,000,000 people  

These classes were selected to highlight variations in population exposure and to clearly distinguish between moderately and highly underserved areas.

LGAs identified as having minimal uncovered areas (≤ 2 km²) were symbolized separately to avoid visual clutter and ensure that the analysis focuses on regions with significant service gaps.

The final map was styled using a **graduated orange-to-red color scheme**, where darker shades represent higher population exposure. Fire station locations and their 5 km coverage buffers were overlaid to provide spatial context and enhance interpretability.

This visualization enables easy identification of high-risk areas and supports informed decision-making for emergency service planning and resource allocation.

## 📊 Key Findings

- Approximately **5.9 million people** in Lagos State live outside a 5 km radius of the nearest fire station, indicating significant gaps in emergency service accessibility.

- **Ojo and Ikorodu LGAs** were identified as the most affected areas, each with **over 1 million people** living outside fire station coverage.

- Notably, **Ojo LGA has no fire station within its boundary**, further increasing its vulnerability to fire-related incidents.

- The spatial distribution of underserved populations shows that fire station accessibility is uneven, with some densely populated areas lacking adequate coverage.

- A few LGAs — including **Ikeja, Surulere, Apapa, Ajeromi-Ifelodun, and Oshodi-Isolo** — recorded only **minimal uncovered areas (≤ 2 km²)**. In particular, **Ikeja and Surulere had extremely small uncovered extents (~0.02 km²)**, which are practically negligible in the context of this analysis.

- While some LGAs have only minimal uncovered regions, others experience substantial service gaps that could significantly impact emergency response times.

- These findings highlight the urgent need for improved fire station distribution and strategic planning to ensure better coverage across rapidly growing urban areas.

## 🗺️ Map Output

The final map presents the spatial distribution of fire station coverage across Lagos State, highlighting areas outside the 5 km service radius and the population affected within those regions.

LGAs are classified based on the number of people living outside fire station coverage, using a graduated orange-to-red color scheme where darker shades indicate higher population exposure.

Fire station locations and their corresponding 5 km buffer zones are overlaid to provide spatial context and clearly distinguish between covered and underserved areas.

![Lagos Fire Station Coverage Map](output/lagos-fire-station-coverage-map.png)

## ⚙️ Tools & Technologies

- **QGIS** – for spatial analysis, data processing, and map visualization  
- **QuickOSM Plugin (QGIS)** – for extracting fire station data from OpenStreetMap  
- **WorldPop** – for high-resolution population data (2026 estimates)  
- **HDX (OCHA)** – for administrative boundary data  
- **Google Maps & Google Earth Pro** – for supplementing and validating fire station locations  

**Spatial Analysis Techniques:**
- Buffer Analysis  
- Overlay Analysis (Difference)  
- Zonal Statistics  
- Data Cleaning and Deduplication  
- Rule-Based Classification and Cartographic Visualization

## 📌 Conclusion

This project demonstrates how geospatial analysis can be applied to assess emergency service accessibility and identify critical infrastructure gaps within an urban environment.

By integrating fire station locations with high-resolution population data, the analysis revealed that a significant portion of Lagos State’s population resides outside effective fire station coverage, with certain LGAs experiencing severe service deficits.

These findings highlight the importance of data-driven planning in improving emergency response systems and ensuring more equitable distribution of essential services across rapidly growing cities.

## 👤 Author
Ohi
