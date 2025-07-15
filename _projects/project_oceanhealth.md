---
layout: page
title: Effect of Different Factors on Plankton Levels in the Ocean
description: Study on the correlation between shipping lanes, nitrate levels, plankton levels, and whale activity.
img: assets/img/erd.jpg
importance: 1
category: work
related_publications: true
---

# Motivation

Plankton are organisms classified by their inability to move against tides and currents. Typically microscopic in size, they have been broadly categorized into two groups, phytoplankton (plants) and zooplankton (animals). Phytoplankton are found near the ocean surface and convert sunlight and carbon dioxide into energy and oxygen. Many zooplankton eat phytoplankton and in turn are eaten by larger marine life. For example, krill are a type of zooplankton and make up much of the diet of humpback whales. 

Both phytoplankton and zooplankton play an important role in the ocean ecosystem, and both are sensitive to changes in the environment. Changes in temperature, salinity, pH, and nutrient levels all have an effect on the plankton populations. Because of this, plankton can serve as a proxy for the health of the ocean ecosystem, as changes in plankton levels both indicate shifting ocean environmental factors as well as likely having an effect going up the food chain. 

The ocean serves as a large sink for greenhouse gases. For example, 30% of the carbon dioxide released into the atmosphere is absorbed by the ocean, which has decreased the surface pH of the water by 0.1. Another common greenhouse gas is nitrogen oxides, which can take the form of nitrate or nitrite. These chemical species are produced from ships as they burn fuel. Since 80% of international trade is on the ocean, it makes sense that common trade routes will have higher nitrogen oxide levels than the surrounding ocean, and since plankton are sensitive to changes in their environment, there may be differences in plankton levels. Larger marine organisms such as whales may have adapted behavior around shipping lanes as well. Through this project we hope to gain understanding on how shipping lanes alter and affect the ocean ecosystem. 

Studying and understanding the human effect on Earth is a priority as humanity's environmental impact grows as a globe-spanning civilization. This research may provide further insight into how we are harming the oceans, give another reason to find new energy sources for ocean travel that minimize damage to the ocean, and potentially remap shipping lanes to vary in ways that minimize areas of high ship density.

# Data

## Sources

Data for this project was compiled from multiple different sources. Nitrate and plankton levels were obtained from the World Ocean Database. The World Ocean Database is an ongoing project by the International Oceanographic Data and Information Exchange (IODE) and the World Data Service for Oceanography of the World Data System. It is hosted by the National Centers for Environmental Information (NCEI) which is a subset of the National Oceanic and Atmospheric Administration (NOAA). The goal of this 30 year project is to allow open and public access to the world’s largest collection of ocean profile data that is uniformly formatted and quality controlled. It includes measures for temperature, salinity, plankton, alkalinity, and pH, as well as concentrations of chemicals such as oxygen, nitrogen oxide species, and dissolved inorganic carbon.

Shipping density data was downloaded from the World Bank Group. This data consisted of a high resolution TIFF image, with each pixel representing shipping density over a 0.005 by 0.005 degree grid, which equates to around a 500m by 500m square near the equator. Because of the large number of pixels in this image, it was not feasible to extract every pixel value from this image. Instead, latitude and longitude coordinates were taken from the observations from the World Ocean Database containing nitrate and plankton levels, and a Python script was run to obtain the pixel value at each location. An important note is that this TIFF image did not record data above 85 degrees north and below -85 degrees south, so for locations that fall in this area no shipping data is recorded.

Phytoplankton concentration data was downloaded from NASA’s OceanColor database using a programmatic query to the OceanData API. Specifically, the request targeted Aqua MODIS Level 3 chlorophyll a data, monthly composite, at a 9 km resolution, spanning from July 2002 to March 2025. The raw .nc files were processed using xarray and pandas in Python to extract spatially gridded chlor_a values along with corresponding lat, lon, year, and month. For each time step, the grid was averaged at a 0.01 degrees resolution to reduce data size. Final data was saved to year-specific .csv files for easy integration into downstream analysis pipelines and uploaded to a PostgreSQL database for access by mapping and modeling workflows. The chlor_a variable is used as a proxy for phytoplankton health and abundance. These values were cleaned to remove extreme outliers and negative values (which are physically implausible). Data north of 85 degrees and south of -85 degrees latitude is missing due to satellite orbital limits and coverage.

Whale sighting data was obtained from OBIS-SEAMAP, focusing specifically on Balaenoptera acutorostrata and Balaenoptera musculus sightings from 1911 through 2024. This dataset includes the whale species, sighting location (latitude, longitude), observation date range, provider, and dataset reference ID. Each record includes a standardized csquare spatial identifier and a WKT polygon representing the area of observation. These were used to spatially map observations and relate them to chlorophyll levels. Data was cleaned to remove any rows missing latitude/longitude or with ambiguous date formats. Provider names were standardized to avoid duplication, and records with duplicate oid identifiers were deduplicated.

## Organization

All the data was uploaded to an online Postgres database hosted by Railway, and was organized in the following manner.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/erd.jpg" title="Entity Relationship Diagram" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Entity relationship diagram showing the compilation and organizational structure of the data.
</div>

Each observation from the World Ocean Database contained a Cast number, year, month, day, and latitude and longitude coordinates. These were split into separate tables with unique ID numbers. Units for each measurement were also extracted and put in their own table. Specific to the plankton data, measurement types were extracted and put in their own table, as well as two identifiers, wod_pgc and its_tsn, which contain information about trophic groups and genus respectively. The shipping density pixel values are in the denseboats table, where the id is the same as the coordinate id numbers from the coordinates table. All foreign key references are shown in the above ERD.

*Other constraints not shown in ERD*: In the `nitrite` table, columns `depth_unit_id`, `depth_uncertainty_id`, and `nitrite_unit_id` cannot contain NULL values. This is because for any measurement to have meaning, it must also contain its corresponding unit. The `nitrate_value` and `depth` columns also cannot contain NULL values, as a missing measurement is not useful. Likewise, in the `plankton` table, `measurement_type_id`, `plankton_unit_id`, `wod_pgc_id`, `its_tsn_id`, and `plankton_value` columns all cannot contain NULL values.
The chlorophyll_a data was organized by lat, lon, year, and month into wide-format .csv files and stored in a structured folder hierarchy. Each file represents a single year of monthly 9 km chlorophyll concentration grids. 

Whale sightings were converted to a `clean.csv` file containing key variables for geospatial and temporal matching: `species`, `longitude`, `latitude`, and `date_min`. The `geom_wkt` column was retained to preserve spatial resolution of the reported sighting box. 

All date fields were parsed to datetime objects to support time-series filtering and aggregation. Chlorophyll_a values and whale sightings were linked on the basis of spatial proximity and month year overlap.

