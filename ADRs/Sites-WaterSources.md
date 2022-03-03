# Decision record 

# Title 
Supporting many-to-many relationships between water sources and sites for site-specific time series data

## Status
This change has been implemented in February 2022 into the database and APIs calls

## Context
We needed this change to support unique cases for managing water use time series data that we didn’t anticipate earlier. This data is particularly interesting to the USGS National Water Use Program. USGS is planning on using this WaDE data for their National Water Use Census. 

Needed relationships between sites, Water Sauces, Amounts:
1. A site (city) may receive water from many different sources. 
2. A water source may supply many sites.
3. For each site, there is a time series of reported water use per water source 

The example here is California and Texas water use data that are reported at the city (user) level as aggregate of surface or groundwater time series. The existing design supported one water source (e.g., groundwater) suppling one or many sites but a site can only receive water from one source. We need the same water source to supply many sites and also a site to be supplied by multiple water sources. It may sound obvious that this many to many design was needed. However, based on our experience working on WaDE 1.0, this case didn’t show up. The main reason we noticed this case is that states like CA and TX have aggregated their water sources into ~three major types (e.g., surface water, groundwater, reuse). So the source is no longer a single groundwater well that supplied a city but rather it is an aggregate groundwater source that supplies all the cities in the state. States' don’t always collect groundwater use per source type, so this aggregation is a way around not knowing the exact origin of groundwater.


## Decision

We have implemented this design change by adding a new bridge table between Sites and Water Sources called: WaterSourceBridge_Sites_fact. 
https://schema.westernstateswater.org/diagrams/2_SiteSpecificAmounts.html


## Consequences

1. While this new bridge table solves the issue we faced before, it does have a consequence. An amount value in the VariableAmounts_fact couldn’t not be traced to one water source when many supply the city through its site. As a solution, we are storing the WaterSourceID	within the VariableAmounts_fact to help in tracking the exact water source per amount. This is not the best generalized design but it works for us given that it will be very expensive to resign the entire database. Future work for WaDE 3.0 may revise this design 

2. This design change is intended for site specific Variable Amounts data but it also does affect water allocation (water rights) data. The good news is that in water rights data, one site seems to be always related to one water source (e.g., river, aquifer, reservoir)--a water right is given for either surface water or groundwater at-a-time, not together. If we find multiple water sources attached to one water right, then we need to add WaterSourceID to the AllocationAmounts_fact table to keep track of them. 


# Template by Michael Nygard

This is the template in [Documenting architecture decisions - Michael Nygard](http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions).
You can use [adr-tools](https://github.com/npryce/adr-tools) for managing the ADR files.

In each ADR file, write these sections:
