---
output: html_document
---
# NOAA Weather Treemaps - Visualizing data about Human and Physical damages caused by weather in the United States

## Synopsis

The objective of this study is to create a data visualization based on data analysis of NOAA weather events dataset, to support better understanding of which types of weather events across the United States are most harmful with respect to population health, and which ones are generating more economic losses to crops and properties across the nation.

The following R packages were used to read the data and build the visualizations:

```{r}
library(R.utils)
library(data.table)
library(dplyr)
library(plyr)
require(treemap)
require(grid)
```

## Data Processing

The data was downloaded from:
<https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2>.

After download, the data was unzipped for using the csv file in R:

```{r}
bunzip2("repdata-data-StormData.csv.bz2","repdata-data-StormData.csv", remove = FALSE, skip = TRUE)
```

Because of the size of the dataset, the author preferred to use "fread" to import the csv data more quickly:

```{r}
storm=fread("repdata-data-StormData.csv")
```

The warning from "fread" was investigated, and accordingly to reports from the community, "The file is read completely and correctly. The issue is that fread() is bad at counting the number of rows in specific cases (which this case falls under)."
(<https://github.com/Rdatatable/data.table/issues/1239>)

After that, the author created two auxiliary objects, "events" and "economic", to help calculating the health impacts(events) and economic impacts(economic).

```{r}
events=ddply(storm, .(EVTYPE),summarise,Fatalities = sum(FATALITIES),Injuries = sum(INJURIES))
economic=ddply(storm, .(EVTYPE),summarise,PROPDMG = sum(PROPDMG),CROPDMG = sum(CROPDMG))
```

## Results

To turn the data more visually atractive, I set two sets of treemaps from "treemap" package.

Each set is composed by four treemaps, to show how the impacts are distributed across the United States, and to show which are the most frequent events generating the impacts in health or economic impacts.  To put the treemaps together, the package "grid" was used.

If you want to investigate better the charts, an option is looking at "plot1.svg" and "plot2.svg" in this repo, zooming the pages to better explore the data, or using these scripts in R and generating your own files.  The full script is in "grid_treemaps.R", in this repo.

```{r}
vplayout <- function(x, y) viewport(layout.pos.row = x, layout.pos.col = y)
svg('plot1.svg', width = 16, height = 9)
grid.newpage()
pushViewport(viewport(layout = grid.layout(2, 2)))
treemap(storm,index="STATE",vSize="FATALITIES",vColor="FATALITIES",type="value",title = "Fatalities across US States", palette="RdGy",force.print.labels = TRUE,vp = vplayout(1,1))
treemap(storm,index="STATE",vSize="INJURIES",vColor="INJURIES",type="value",title = "Injuries across US States",palette="PRGn",force.print.labels = TRUE,vp = vplayout(1,2))
treemap(events,index="EVTYPE",vSize="Fatalities",vColor="Fatalities",type="value",title = "Fatalities by Weather Events", palette="RdPu",force.print.labels = TRUE,vp = vplayout(2,1))
treemap(events,index="EVTYPE",vSize="Injuries",vColor="Injuries",type="value",title = "Injuries by Weather Events",palette="RdBu",force.print.labels = TRUE,vp = vplayout(2,2))
dev.off()
```

```{r}
vplayout <- function(x, y) viewport(layout.pos.row = x, layout.pos.col = y)
svg('plot1.svg', width = 16, height = 9)
grid.newpage()
pushViewport(viewport(layout = grid.layout(2, 2)))
treemap(storm,index="STATE",vSize="PROPDMG",vColor="PROPDMG",type="value",title = "Property dmg across States", palette="PuRd",force.print.labels = TRUE,vp = vplayout(1,1))
treemap(storm,index="STATE",vSize="CROPDMG",vColor="CROPDMG",type="value",title = "Crops dmg across States", palette="BuGn",force.print.labels = TRUE,vp = vplayout(1,2))
treemap(economic,index="EVTYPE",vSize="PROPDMG",vColor="PROPDMG",type="value",title = "Property dmg by Events", palette="GnBu",force.print.labels = TRUE,vp = vplayout(2,1))
treemap(economic,index="EVTYPE",vSize="CROPDMG",vColor="CROPDMG",type="value",title = "Crops dmg by Events",palette="YlGn",force.print.labels = TRUE,vp = vplayout(2,2))
dev.off()
```
