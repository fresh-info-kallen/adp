---
output:
  html_document:
    keep_md: yes
  pdf_document: default
---


# Description of Imputation Methodology

# Section 1 - Overview 

This R Markdown document will demonstrate the calculation of imputation for each of the below measures:

- $Available\_Stay\_Unit\_Nights$
- $Occupied\_Nights$
- $Guest\_Nights$
- $Guest\_Arrivals$
- $Domestic\_Nights$ (and by extension, International Nights, which is $Guest\_Nights - Domestic\_Nights$)

We will first inspect the first 6 rows of the data at the start of the imputation process. Where no data for a particular measure is provided, this is indicated by `NA`. For these properties, there is a mix of responses and non responses for each the data fields shown (non imputed values). 

The imputation process will compute an imputation to fill each `NA` with an imputed value, in the order of columns as per the table below (left to right). The monthly capacity of each property is always known. This is done by computing an operational ratio, and multiplying the ratio with the value of the field preceding the field for which an imputation is being computed. 

For example, to calculate an imputation for `available_stay_unit_nights` for Executive Residence Boutique Hotel, we first calculate an operational ratio given by non-imputed data from donor properties, and multiply that ratio with the 
value of the field `monthly_capacity` for Executive Residence Boutique Hotel. 

This hotel is also missing data for `occupied_nights` and so this field will need to be computed. As per the previous imputation, an operational ratio is calculated and is multiplied with the imputed value of the preceding field, `available_stay_unit_nights`




```r
table_properties_head
```

<div style="border: 1px solid #ddd; padding: 5px; overflow-x: scroll; width:100%; "><table class=" lightable-classic table table-striped table-hover table-condensed table-responsive" style="font-family: Cambria; width: auto !important; margin-left: auto; margin-right: auto; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> name </th>
   <th style="text-align:right;"> fresh_id </th>
   <th style="text-align:left;"> rto </th>
   <th style="text-align:left;"> property_type </th>
   <th style="text-align:right;"> monthly_capacity </th>
   <th style="text-align:right;"> non_imputed_available_stay_unit_nights </th>
   <th style="text-align:right;"> non_imputed_occupied_nights </th>
   <th style="text-align:right;"> non_imputed_guest_nights </th>
   <th style="text-align:right;"> non_imputed_guest_arrivals </th>
   <th style="text-align:right;"> non_imputed_domestic </th>
   <th style="text-align:right;"> non_imputed_international </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Queenstown Park Boutique Hotel </td>
   <td style="text-align:right;"> 1706 </td>
   <td style="text-align:left;"> Queenstown RTO </td>
   <td style="text-align:left;"> Hotels (over 20 units) </td>
   <td style="text-align:right;"> 660 </td>
   <td style="text-align:right;"> 660 </td>
   <td style="text-align:right;"> 369 </td>
   <td style="text-align:right;"> 727 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> 727 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Executive Residence Boutique Hotel </td>
   <td style="text-align:right;"> 8440 </td>
   <td style="text-align:left;"> Dunedin RTO </td>
   <td style="text-align:left;"> Hotels (over 20 units) </td>
   <td style="text-align:right;"> 720 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Greenlane Suites </td>
   <td style="text-align:right;"> 434 </td>
   <td style="text-align:left;"> Auckland RTO </td>
   <td style="text-align:left;"> Hotels (over 20 units) </td>
   <td style="text-align:right;"> 750 </td>
   <td style="text-align:right;"> 750 </td>
   <td style="text-align:right;"> 632 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Awaroa Lodge </td>
   <td style="text-align:right;"> 7054 </td>
   <td style="text-align:left;"> Nelson Tasman RTO </td>
   <td style="text-align:left;"> Hotels (over 20 units) </td>
   <td style="text-align:right;"> 780 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Peppers Parehua Martinborough </td>
   <td style="text-align:right;"> 5806 </td>
   <td style="text-align:left;"> Wairarapa RTO </td>
   <td style="text-align:left;"> Hotels (over 20 units) </td>
   <td style="text-align:right;"> 840 </td>
   <td style="text-align:right;"> 840 </td>
   <td style="text-align:right;"> 685 </td>
   <td style="text-align:right;"> 1384 </td>
   <td style="text-align:right;"> 810 </td>
   <td style="text-align:right;"> 1350 </td>
   <td style="text-align:right;"> 34 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Wai Ora Lakeside Spa Resort </td>
   <td style="text-align:right;"> 731 </td>
   <td style="text-align:left;"> Rotorua RTO </td>
   <td style="text-align:left;"> Hotels (over 20 units) </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 900 </td>
   <td style="text-align:right;"> 366 </td>
   <td style="text-align:right;"> 920 </td>
   <td style="text-align:right;"> 240 </td>
   <td style="text-align:right;"> 920 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
</tbody>
</table></div>

# Section 2 - Preprocessing 

#### 2.1 Calculating Z Score

#### 2.2 Removing Unduly influential points 



# Section 3 - Imputation Process

#### 3.1 - Derive Donor Property Set

This first step involves finding the set of other comparable properties for each property. The set of comparable properties will consist of either

- properties sharing the same property type within the same RTO, or 
- properties sharing the same property type within New Zealand.

The former is used when there are sufficient respondents who have submitted data for a given attribute. If there are insufficient respondents, data from all respondents across the country sharing the same property type is used for imputation. This is done to protect the confidentiality of respondents, and also to increase the diversity of responses and to reduce the risk of one or two properties exerting undue influence on the imputation calculation. 

When calculating the imputation for any given field, the donor property set is determined by :

- Property Type  

- RTO - If there are three or more active and responding properties of the same type within the same RTO (including property A), then the donor set will consist of these properties (excluding property A); otherwise the donor set will consist of all properties of the same property type across New Zealand. 

- Whether a property has submitted data for a specified field. This field depends on the field which is being imputed. For example, when calculating the imputation for the field $Available\_Stay\_Unit\_Nights$ for hotels in the Auckland RTO, the donor property set consists of all hotels in Auckland RTO which have submitted data for  $Occupied\_Nights$. If there are less than three proprerties which fulfill this criteria, then the donor property set will instead consist of hotels across New Zealand.

| Field to be imputed             | Field to determine donor|
|---------------------------------|-------------------------|
| $Available\_Stay\_Unit\_Nights$ | $Occupied\_Nights$      |
| $Occupied\_Nights$              | $Occupied\_Nights$      |
| $Guest\_Nights$                 | $Guest\_Arrivals$       |
| $Guest\_Arrivals$               | $Guest\_Arrivals$       |
| $Domestic\_Nights$              | $Domestic\_Nights$      |


The following code examples will demonstrate the process for calculating the ratio which is used to impute the field  $Occupied\_Nights$; This code is then generalised to apply for all of the imputation calculations. 

We first need to count the number of properties which fulfill the three matching criteria as defined above. 


```r
df_grouped = properties%>%
  group_by_at(c("rto", "property_type")) %>%
  summarise(
    
    # Count properties with at least and at least one occupied night - This is used to determine `donor_set_type`
    count_of_donors_occupancy = sum( (non_imputed_occupied_nights > 0), na.rm = T),
     )%>%
  ungroup()
 
df_grouped %>%
  filter(rto == "Bay of Plenty RTO") %>%
  kbl() %>%
  kable_classic(full_width = F, html_font = "Cambria") %>%
  kable_styling(bootstrap_options = c("striped", "hover", "condensed", "responsive")) %>%
  scroll_box(width = "100%")
```

<div style="border: 1px solid #ddd; padding: 5px; overflow-x: scroll; width:100%; "><table class=" lightable-classic table table-striped table-hover table-condensed table-responsive" style="font-family: Cambria; width: auto !important; margin-left: auto; margin-right: auto; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> rto </th>
   <th style="text-align:left;"> property_type </th>
   <th style="text-align:right;"> count_of_donors_occupancy </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Backpackers (over 20 units) </td>
   <td style="text-align:right;"> 3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Holiday parks and campgrounds </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Hotels (over 20 units) </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Motels and serviced apartments (6-20 units) </td>
   <td style="text-align:right;"> 17 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Motels and serviced apartments (over 20 units) </td>
   <td style="text-align:right;"> 5 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Other accommodation (over 5 units) </td>
   <td style="text-align:right;"> 1 </td>
  </tr>
</tbody>
</table></div>

We then proceed to name the donor property set. Based on the counts shown above, we must confidentialise the rto + property groups which have less than three properties. These groups will have group type `donor_set_type = 'property_type'` as they consist of properties from across the country sharing the same property type. Groups which have counts of three or greater will consist of properties only within the same rto, and so have `donor_set_type = rto_and_property_type` 


```r
# Determine comp set type and name
df_grouped = df_grouped %>%
  mutate(
    donor_set_type = ifelse(
      count_of_donors_occupancy >= 3,
      "rto_and_property_type",
      "property_type"),
    
    donor_set_name = ifelse(
      count_of_donors_occupancy >= 3,
      paste(rto, property_type, sep = ""),
      property_type)
  )
```





```r
table_df_grouped
```

<div style="border: 1px solid #ddd; padding: 5px; overflow-x: scroll; width:100%; "><table class=" lightable-classic table table-striped table-hover table-condensed table-responsive" style="font-family: Cambria; width: auto !important; margin-left: auto; margin-right: auto; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> rto </th>
   <th style="text-align:left;"> property_type </th>
   <th style="text-align:right;"> count_of_donors_occupancy </th>
   <th style="text-align:left;"> donor_set_type </th>
   <th style="text-align:left;"> donor_set_name </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Backpackers (over 20 units) </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:left;"> rto_and_property_type </td>
   <td style="text-align:left;"> Bay of Plenty RTOBackpackers (over 20 units) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Holiday parks and campgrounds </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:left;"> rto_and_property_type </td>
   <td style="text-align:left;"> Bay of Plenty RTOHoliday parks and campgrounds </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Hotels (over 20 units) </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:left;"> property_type </td>
   <td style="text-align:left;"> Hotels (over 20 units) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Motels and serviced apartments (6-20 units) </td>
   <td style="text-align:right;"> 17 </td>
   <td style="text-align:left;"> rto_and_property_type </td>
   <td style="text-align:left;"> Bay of Plenty RTOMotels and serviced apartments (6-20 units) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Motels and serviced apartments (over 20 units) </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:left;"> rto_and_property_type </td>
   <td style="text-align:left;"> Bay of Plenty RTOMotels and serviced apartments (over 20 units) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Other accommodation (over 5 units) </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> property_type </td>
   <td style="text-align:left;"> Other accommodation (over 5 units) </td>
  </tr>
</tbody>
</table></div>

Please see the plots in the appendix section if you are interested in seeing a visualisation of the results of this step for a select group of properties. 

#### 3.2 - Compute Operational Ratios for each possible donor group

For each `donor_set_name`, we will compute an operational ratio which will be used as an imputation ratio for all accomodation providers within this group (where an imputation is required).


###### Example: Occupancy Rate: 

In this example, the occupancy rate is being calculated. This operational ratio will be used to impute the `occupied\_nights_` for any properties in this group where data has not been provided.

The numerator represents the total number of occupied nights and the denominator represents the total number of available stay unit nights across all the properties belonging to `donor_set_name`.

$$r= \frac{
\sum_{i \in\{i | {occupied\_nights}_{i} > 0 \}}^N occupied\_nights_{i}
}{
\sum_{i \in\{i | {occupied\_nights}_{i} > 0 \}}^N available\_stay\_unit\_nights_{i}
}$$

where $i$ represents the $i-th$ donor property, and $N$ is the total number of properties within the donor property set.  


```r
df_grouped = properties%>%
  group_by_at(c("rto", "property_type")) %>%
  summarise(
    
    # Count properties with at least one occupied night - This is used to determine `donor_set_type`
    count_of_donors_occupancy = sum(non_imputed_occupied_nights > 0, na.rm = T),
    
    # Numerator
      sum_non_imputed_available_stay_unit_nights = sum(non_imputed_available_stay_unit_nights[(non_imputed_available_stay_unit_nights > 0) & (non_imputed_occupied_nights > 0)], na.rm = T),
    
    # Denominator
    sum_monthly_capacity = sum(monthly_capacity[(non_imputed_available_stay_unit_nights > 0) & (non_imputed_occupied_nights > 0)], na.rm = T)
     )%>%
  ungroup()

# Determine comp set type and name
df_grouped = df_grouped %>%
  mutate(
    donor_set_type = ifelse(
      count_of_donors_occupancy >= 3,
      "rto_and_property_type",
      "property_type"), 
    
    donor_set_name = ifelse(
      count_of_donors_occupancy >= 3,
      paste(rto, property_type, sep = ""),
      property_type)
  )

# Confidentialise

# For donor_set_type == 'property_type'
aggregations_comp_set_property.df <- df_grouped %>%
  group_by(donor_set_name = rto) %>%
  summarise(
    sum_non_imputed_available_stay_unit_nights = sum(sum_non_imputed_available_stay_unit_nights), 
    sum_monthly_capacity = sum(sum_monthly_capacity)
  )

imputation_per_property_type.df = inner_join(
  x = df_grouped %>% select(donor_set_name, donor_set_type, rto, property_type, count_of_donors_occupancy),
  y = aggregations_comp_set_property.df,
  by = c("donor_set_name" = "donor_set_name")
)

# For donor_set_type == 'rto_and_property_type'
imputation_per_rto_and_property_type.df = df_grouped %>%
  filter(donor_set_type == "rto_and_property_type")

imputation_schedule_occupancy = rbind(imputation_per_property_type.df, imputation_per_rto_and_property_type.df)

# Compute Ratio
imputation_schedule_occupancy$ratio_occupancy = imputation_schedule_occupancy$sum_non_imputed_available_stay_unit_nights / imputation_schedule_occupancy$sum_monthly_capacity

imputation_schedule_occupancy  %>%
  filter(rto == "Bay of Plenty RTO") %>%
  kbl() %>%
  kable_classic(full_width = F, html_font = "Cambria") %>%
  kable_styling(bootstrap_options = c("striped", "hover", "condensed", "responsive")) %>%
  scroll_box(width = "100%")
```

<div style="border: 1px solid #ddd; padding: 5px; overflow-x: scroll; width:100%; "><table class=" lightable-classic table table-striped table-hover table-condensed table-responsive" style="font-family: Cambria; width: auto !important; margin-left: auto; margin-right: auto; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> rto </th>
   <th style="text-align:left;"> property_type </th>
   <th style="text-align:right;"> count_of_donors_occupancy </th>
   <th style="text-align:right;"> sum_non_imputed_available_stay_unit_nights </th>
   <th style="text-align:right;"> sum_monthly_capacity </th>
   <th style="text-align:left;"> donor_set_type </th>
   <th style="text-align:left;"> donor_set_name </th>
   <th style="text-align:right;"> ratio_occupancy </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Backpackers (over 20 units) </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 4688 </td>
   <td style="text-align:right;"> 5220 </td>
   <td style="text-align:left;"> rto_and_property_type </td>
   <td style="text-align:left;"> Bay of Plenty RTOBackpackers (over 20 units) </td>
   <td style="text-align:right;"> 0.8980843 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Holiday parks and campgrounds </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:right;"> 36053 </td>
   <td style="text-align:right;"> 40860 </td>
   <td style="text-align:left;"> rto_and_property_type </td>
   <td style="text-align:left;"> Bay of Plenty RTOHoliday parks and campgrounds </td>
   <td style="text-align:right;"> 0.8823544 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Motels and serviced apartments (6-20 units) </td>
   <td style="text-align:right;"> 17 </td>
   <td style="text-align:right;"> 5977 </td>
   <td style="text-align:right;"> 6720 </td>
   <td style="text-align:left;"> rto_and_property_type </td>
   <td style="text-align:left;"> Bay of Plenty RTOMotels and serviced apartments (6-20 units) </td>
   <td style="text-align:right;"> 0.8894345 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Motels and serviced apartments (over 20 units) </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 4077 </td>
   <td style="text-align:right;"> 4920 </td>
   <td style="text-align:left;"> rto_and_property_type </td>
   <td style="text-align:left;"> Bay of Plenty RTOMotels and serviced apartments (over 20 units) </td>
   <td style="text-align:right;"> 0.8286585 </td>
  </tr>
</tbody>
</table></div>

| Field                      | Description                                                                                                                                                                                  | Formula                                                                                                                                                                                              |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Available Stay Unit Nights | Stay units made available for short-term guests multiplied by the number of days of availability. This includes occupied and unoccupied stay units, and those located for managed isolation. | $$r= \frac{ \sum_{i \in\{i \| {available\_stay\_unit\_nights}_{i} > 0 \}} available\_stay\_unit\_nights_{i} }{ \sum_{i \in\{i \| {available\_stay\_unit\_nights}_{i} > 0 \}} monthly\_capacity_{i} }$$ |
| Occupied Nights            | Stay units occupied by short-term guests during the month. This includes managed isolation guests.                                                                                           | $$r= \frac{ \sum_{i \in\{i \| {occupied\_nights}_{i} > 0 \}} occupied\_nights_{i} }{ \sum_{i \in\{i \| {occupied\_nights}_{i} > 0 \}} available\_stay\_unit\_nights_{i} }$$                            |
| Guest Nights               | Cumulative number of nights spent short-term guests. Equivalent to one guest spending one night in a property. This includes managed isolation guests.                                       | $$r= \frac{ \sum_{i \in\{i \| {guest\_nights}_{i} > 0 \}} guest\_nights_{i} }{ \sum_{i \in\{i \| {guest\_nights}_{i} > 0 \}} occupied\_nights_{i} }$$                                                  |
| Guest Arrivals             | Number of individual short-term guests checking in during the reference month. This includes managed isolation guests.                                                                       | $$r= \frac{ \sum_{i \in\{i \| {guest\_arrivals}_{i} > 0 \}} guest\_arrivals_{i} }{ \sum_{i \in\{i \| {guest\_arrivals}_{i} > 0 \}} guest\_nights_{i} }$$                                               |
| Domestic Nights            | Nights spent by guests that were identified as being from New Zealand.                                                                                                                       | $$r= \frac{ \sum_{i \in\{i \| {domestic\_nights}_{i} > 0 \}} domestic\_nights_{i} }{ \sum_{i \in\{i \| {domestic\_nights}_{i} > 0 \}} guest\_nights_{i} }$$                                            |

## All Ratios

###### Available Stay Unit Nights

Stay units made available for short-term guests multiplied by the
number of days of availability. This includes occupied and unoccupied
stay units, and those located for managed isolation.

$$r= \frac{
\sum_{i \in\{i | {available\_stay\_unit\_nights}_{i} > 0 \}} available\_stay\_unit\_nights_{i}
}{
\sum_{i \in\{i | {available\_stay\_unit\_nights}_{i} > 0 \}} monthly\_capacity_{i}
}$$

###### Occupied Nights 

Stay units occupied by short-term guests during the month. This
includes managed isolation guests.

$$r= \frac{
\sum_{i \in\{i | {occupied\_nights}_{i} > 0 \}} occupied\_nights_{i}
}{
\sum_{i \in\{i | {occupied\_nights}_{i} > 0 \}} available\_stay\_unit\_nights_{i}
}$$


###### Guest Nights 

Cumulative number of nights spent short-term guests. Equivalent to
one guest spending one night in a property. This includes managed
isolation guests.

$$r= \frac{
\sum_{i \in\{i | {guest\_nights}_{i} > 0 \}} guest\_nights_{i}
}{
\sum_{i \in\{i | {guest\_nights}_{i} > 0 \}} occupied\_nights_{i}
}$$

###### Guest Arrivals

Number of individual short-term guests checking in during the
reference month. This includes managed isolation guests.

$$r= \frac{
\sum_{i \in\{i | {guest\_arrivals}_{i} > 0 \}} guest\_arrivals_{i}
}{
\sum_{i \in\{i | {guest\_arrivals}_{i} > 0 \}} guest\_nights_{i}
}$$

###### Domestic Nights

Nights spent by guests that were identified as being from New
Zealand.

$$r= \frac{
\sum_{i \in\{i | {domestic\_nights}_{i} > 0 \}} domestic\_nights_{i}
}{
\sum_{i \in\{i | {domestic\_nights}_{i} > 0 \}} guest\_nights_{i}
}$$

The below function computes an imputation schedule for a particular imputation measure. 


```r
compute_imputation_schedule <- function(
  df, 
  numerator_field, 
  denominator_field, 
  imputation_name,
  imputation_comp_set_field
  ){
  
  df_grouped = df %>% 
    # filter_at(vars(numerator_field), ~ .>0) %>%
    group_by_at(c("rto", "property_type")) %>%
    summarise_(
      count = interp(
        ~sum((numerator_var > 0) & (imputation_comp_set_var > 0), na.rm = T), 
        numerator_var = as.name(numerator_field),
        imputation_comp_set_var = as.name(imputation_comp_set_field)
        ),
      
      numerator = interp(~sum(numerator_var[(numerator_var > 0) & (imputation_comp_set_var > 0)], na.rm = T), 
                         imputation_comp_set_var = as.name(imputation_comp_set_field),
                         numerator_var = as.name(numerator_field)
                         ),
      
      denominator = interp(~sum(denominator_var[(numerator_var > 0) & (imputation_comp_set_var >0)], na.rm = T), 
                           imputation_comp_set_var = as.name(imputation_comp_set_field), 
                           denominator_var = as.name(denominator_field),
                           numerator_var = as.name(numerator_field))
      ) %>% ungroup()
    
  # Determine comp set type and name
  df_grouped = df_grouped %>%
    mutate(
      donor_set_type = ifelse(
        count >= 3,
        "rto_and_property_type",
        "property_type"),
      
      donor_set_name = ifelse(
        count >= 3,
        paste(rto, property_type, sep = ""),
        property_type)
    )
  # Confidentialise
  aggregations_comp_set_property.df <- df_grouped %>%
  group_by(donor_set_name = property_type) %>%
  summarise(
    numerator = sum(numerator),
    denominator = sum(denominator)
  )

  imputation_per_property_type.df = inner_join(
    x = df_grouped %>%
      select(donor_set_name, donor_set_type, rto, property_type, count),
    y = aggregations_comp_set_property.df,
    by = c("donor_set_name" = "donor_set_name")
  )

  imputation_per_rto_and_property_type.df = df_grouped %>%
    filter(donor_set_type == "rto_and_property_type")

  # Row bind the imputation schedules for both `rto_and_property_type` and either `rtop` or `proprety_type`
  imputation_schedule = rbind(imputation_per_property_type.df, imputation_per_rto_and_property_type.df)

  imputation_schedule = imputation_schedule %>%
    rename_at("donor_set_type", ~paste(imputation_name, "donor_set_type", sep = "_")) %>%
    rename_at("donor_set_name", ~paste(imputation_name, "donor_set_name", sep = "_")) %>%
    rename_at("count", ~ paste(imputation_name, "imputation_donors_count", "count", sep = "_")) %>%
    
  # Calculate Ratio 
  mutate(ratio = numerator/denominator) %>%
    
  # Rename Columns
  rename_at("numerator", ~ paste(imputation_name, "sum", numerator_field,  sep = "_")) %>%
  rename_at("denominator", ~ paste(imputation_name,"sum", denominator_field,  sep = "_")) %>%
  rename_at("ratio", ~paste(imputation_name, "ratio", sep = "_"))

  return(imputation_schedule)
}
```



```r
# Our five imputation schedules are as follows:
imputation_available_stay_unit_nights = compute_imputation_schedule(
  df = properties,
  numerator_field = "non_imputed_available_stay_unit_nights",
  denominator_field = "monthly_capacity",
  imputation_name = "available_stay_unit_nights",
  imputation_comp_set_field = "non_imputed_occupied_nights"
  # property_type = "rto"
)

imputation_occupied_nights = compute_imputation_schedule(
  df = properties,
  numerator_field = "non_imputed_occupied_nights",
  denominator_field = "non_imputed_available_stay_unit_nights",
  imputation_name = "occupied_nights",
  imputation_comp_set_field = "non_imputed_occupied_nights"
)

imputation_guest_nights = compute_imputation_schedule(
  df = properties,
  numerator_field = "non_imputed_guest_nights",
  denominator_field = "non_imputed_occupied_nights",
  imputation_name = "guest_nights",
  imputation_comp_set_field = "non_imputed_guest_arrivals"
)

imputation_guest_arrivals = compute_imputation_schedule(
  df = properties,
  numerator_field = "non_imputed_guest_arrivals",
  denominator_field = "non_imputed_guest_nights",
  imputation_name = "guest_arrivals",
  imputation_comp_set_field = "non_imputed_guest_arrivals"
)

imputation_domestic_nights = compute_imputation_schedule(
  df = properties,
  numerator_field = "non_imputed_domestic",
  denominator_field = "non_imputed_guest_nights",
  imputation_name = "domestic_nights",
  imputation_comp_set_field = "non_imputed_domestic"
)
```

#### 3.3 -  Calculating the Imputation

The below code demonstrates how the operational ratio is used to compute an imputation for `occupied_nights`. In this scenario, property 1318 is a property in the Bay of Plenty RTO with property type `Holiday parks and campgrounds.` We look up the property and the rto in the respective imputation schedule for `occupied_nights`. Since there are 6 properties with the same property type within the Bay of PLenty RTO, the set of donors will consist of 5 properties (excluding property 1318).

To compute the imputed occupied nights, we simply multiply the operational ratio with with the `occupied_nights` of property 1318:

$$Occupied\_Nights_{1318} = r \times occupied\_nights_{1318} $$

where

$$r= \frac{
\sum_{i \in\{i | {guest\_nights}_{i} > 0 \}} guest\_nights_{i}
}{
\sum_{i \in\{i | {guest\_nights}_{i} > 0 \}} occupied\_nights_{i}
}$$



```r
filtered.df = properties%>%
  filter(fresh_id == 1318) %>%
  select(fresh_id, rto, property_type, non_imputed_guest_nights, non_imputed_occupied_nights)

# Lookup the rto and property type in the imputation comp set table
filtered.df = left_join(
  x = filtered.df,
  y = imputation_guest_nights,
  by = c("rto", "property_type")
)

# Multiply the ratio by the respective field
filtered.df$imputed_guest_nights = filtered.df$guest_nights_ratio * filtered.df$non_imputed_occupied_nights

# Final Imputed Value 
filtered.df$imputed_guest_nights
```

```
## [1] 8230.709
```

```r
kbl(filtered.df)%>%
   kable_classic(full_width = F, html_font = "Cambria") %>%
  kable_styling(bootstrap_options = c("striped", "hover", "condensed", "responsive")) %>%
  scroll_box(width = "100%")
```

<div style="border: 1px solid #ddd; padding: 5px; overflow-x: scroll; width:100%; "><table class=" lightable-classic table table-striped table-hover table-condensed table-responsive" style="font-family: Cambria; width: auto !important; margin-left: auto; margin-right: auto; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:right;"> fresh_id </th>
   <th style="text-align:left;"> rto </th>
   <th style="text-align:left;"> property_type </th>
   <th style="text-align:right;"> non_imputed_guest_nights </th>
   <th style="text-align:right;"> non_imputed_occupied_nights </th>
   <th style="text-align:left;"> guest_nights_donor_set_name </th>
   <th style="text-align:left;"> guest_nights_donor_set_type </th>
   <th style="text-align:right;"> guest_nights_imputation_donors_count_count </th>
   <th style="text-align:right;"> guest_nights_sum_non_imputed_guest_nights </th>
   <th style="text-align:right;"> guest_nights_sum_non_imputed_occupied_nights </th>
   <th style="text-align:right;"> guest_nights_ratio </th>
   <th style="text-align:right;"> imputed_guest_nights </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> 1318 </td>
   <td style="text-align:left;"> Bay of Plenty RTO </td>
   <td style="text-align:left;"> Holiday parks and campgrounds </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> 3150 </td>
   <td style="text-align:left;"> Bay of Plenty RTOHoliday parks and campgrounds </td>
   <td style="text-align:left;"> rto_and_property_type </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 12414 </td>
   <td style="text-align:right;"> 4751 </td>
   <td style="text-align:right;"> 2.612924 </td>
   <td style="text-align:right;"> 8230.709 </td>
  </tr>
</tbody>
</table></div>









# Appendix

#### Properties in Bay of Plenty RTO

The scatter plot below shows the relationship between available monthly capacity and occupancy rates for accommodation providers within the Bay of Plenty RTO for the month of September 2020. A larger shape indicates that there are more accomodation providers within the donor set. 



```r
p  = imputation_schedule_occupancy %>%
    filter(rto == "Bay of Plenty RTO") %>%
    rename(`Monthly Capacity` = sum_monthly_capacity, count = count_of_donors_occupancy) %>%
    ggplot(aes(x = `Monthly Capacity`, y = ratio_occupancy, col = property_type)) +
    geom_point(aes(shape = donor_set_type, size = count)) +
    geom_text_repel(aes(label = property_type)) +
    ggtitle("Occupancy Ratio vs Monthly Capacity of Bay of Plenty Accomodation Providers")

p
```

![](imputation_calculation_for_mbie_files/figure-html/unnamed-chunk-15-1.png)<!-- -->


#### All Hotels in New Zealand


The below scatter plot shows the count of hotels within each RTO, against the total monthly capacity of hotels within the RTO. RTOs with more than three hotels (indicated by the red dashed line) have `compset_type = rto_and_property_type`, while RTOs with less than three hotels will have `compset_type = rto`As expected, the big attractions such as Auckland, Wellington and Queenstown have the highest capacity and will have plenty of donors. 



```r
imputation_schedule_occupancy %>%
    mutate(rto = str_replace(imputation_schedule_occupancy$rto, " RTO", "")) %>%
    filter(property_type == "Hotels (over 20 units)") %>%
    rename(`Monthly Capacity` = sum_monthly_capacity) %>%

    ggplot(aes(x = `Monthly Capacity`, y = count_of_donors_occupancy, col = donor_set_type)) +
    geom_point(aes(size = count_of_donors_occupancy)) +
    ggtitle("Occupancy Ratio vs Monthly Capacity of NZ Hotels") +
    geom_hline(aes(yintercept =3), linetype = 'dashed', col = 'red') +
    geom_text_repel(aes(label = rto))
```

![](imputation_calculation_for_mbie_files/figure-html/unnamed-chunk-16-1.png)<!-- -->





