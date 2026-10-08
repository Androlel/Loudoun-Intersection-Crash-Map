# Loudoun-Intersection-Crash-Map
A map built using QGIS that compares crash rates for a total of 145 signalized and unsignalized intersections on two way median divided roads in part of Loudoun County. Used trying to figure out if certain unprotected intersections were as dangerous as they appeared. 

![alt text](https://github.com/Androlel/Loudon-Intersection-Crash-Map/blob/main/Signals.png)

## Findings
The result of this experiment showed that on average, intersections with traffic signals resulted in a higher likelihood of crashes than those without. However, this is not a comprehensive conclusion that can prove traffic signals are a more dangerous way of mediating rather lights are installed on more dangerous intersections as per the limitations below.
All crash rates are in rate per million vehicles. 
| Group | n | Mean | Median |
|---|---|---|---|
| Signalized | 54 | 0.92 | 0.83 | 
| Unsignalized | 91 | 0.32 | 0.18|

## Data
- Crash records: VDOT, 2020-2025 https://www.virginiaroads.org/maps/1a96a2f31b4f4d77991471b6cabb38ba/about
- Traffic Volume / ADT: VDOT, 2025 https://www.virginiaroads.org/datasets/558fde82244b440294d831a3f815338f_0/explore?location=37.988939%2C-79.498365%2C7
- Intersections: https://www.virginiaroads.org/datasets/ad843128b9904e4ab87bd3570f7975b2_0/explore?location=38.882669%2C-77.261779%2C14
- Roads: https://www.virginiaroads.org/datasets/lrs-route-master/about
- CRS: ESPG: 2283 - NAD83 / Virginia North (ftUS)
- Census Districts: https://geohub-loudoungis.opendata.arcgis.com/datasets/LoudounGIS::loudoun-2020-census-designated-places/explore?location=38.993493%2C-77.425861%2C13

## Methodology
I defined the study by selecting census tracts in Loudoun County was suburban and contained many of these types of developments. This selection process was somewhat arbitrary but I wanted to focus on areas like Ashburn that were almost entirely suburban developed landscaped that included large 4-lane roads with two lane wide medians as part of their construction. I chose the census districts of Brambleton, Loudoun Valley Estates, Moorefield, Broadlands, Ashburn, Belmont, One Loudoun, and Kincora as the areas for this study to take place. I then used the LRS route master to tightly define a list of two-way roads with medians to use for the case study. This excluded highways and major thoroughfare roads such as VA-7, since I wanted to focus on roads that had residential access. I then selected the intersections which clipped with these roads as the base intersections. I also had intersection data from 2023, meaning I was not accounting for lights installed after then. I manually added lights that appeared on satellite then split the data so that intersections with these new lights would only have accidents after 2023. However these would not be the final intersections as the traffic volume data was not fully comprehensive and often lacked residential vehicle rate data. 

To filter the appropriate crashes I selected all crashes within the intersection (150 feet buffer), filtered by the ones that were labeled as in intersection, and added the total count to each intersection so that the end result was a 150 foot radius circle containing all necessary data. I also combined each intersection that overlapped since most two land divided intersections contained points for each division crossing leading to many intersections with over three points. This did include some intersections that were joining others, but I largely ignored since the crash intersection filter worked largely as a backup definition. But with manual filtration as well I was able to come up with a list of intersections on two-way divided roads that met the criteria I was seeking. 

Unfortunately this is when I decided to find vehicle travel data, which was wildly incomplete meaning I had to redefine many of my intersections. So I clipped all ADT data within my intersections, merged all roads into one leg by removing the other side of the median, and joined all the ADT data with the crash intersections for a final dataset. Below is the calculation used to truncate the ADT data.
```
SELECT int_id, SUM(m) AS adt_sum, COUNT(*) AS adt_roads
FROM (
  SELECT int_id,
         rtrim("Route Name", 'NSEWB') AS br,
         MAX(ADT) AS m
  FROM aadt_legs2
  GROUP BY int_id, br
)
GROUP BY int_id
```

Once I had the data I calculated several fields to find the final crash rate. And got my final result
Note: I only multiplied the years by 5 beacuse there were only a couple records for 2025, they were kept but ideally should have been removed.
```
- entering_veh	      Integer	    "adt_sum" / 2
- mev	              Decimal 	    ("entering_veh" * 365.0 * 5) / 1000000.0 
- crash_rate	      Decimal 	    CASE WHEN "mev" > 0 THEN "crash_total" / "mev" END
```

## Cartography
Built in QGIS 3.34. Both layers use identical classification so the two groups are directly comparable:
- Color: graduated on crash_rate, 7 manual classes at 0.5 intervals (0–3.5). Separate hue families distinguish signalized from unsignalized.
- Size: graduated on crash_rate, 2–12 mm, Flannery-corrected (exponent 0.57) to compensate for perceptual underestimation of circle area.
- Feature counts shown per class in the legend.

## Limitations
Since this was my introductory GIS project I had a lot of limitations for this study. First was the fact I based all of the vehicle travel rate of 2025 when the crash rate spans multiple years. I did this so I can accurately gauge average crash data and get a rough estimate but I did not remember the amount of traffic shrinkage present in 2020 and 2021 due to COVID. Not to mention the area is growing and developing a lot so daily travel rates may be drastically different or nearly nonexistent on some roads.

The ADT data is the most limiting and damaging things here. To actually have something to work with I included a lot of ADT data that was missing a minor/residential part of the intersection. Obviously this means the final crash rate is higher than reported, though on those roads it did not seem to affect the final result too much likely due to the fact that all of these intersections reported a low crash rate. Not to mention the number of possible intersections was nearly halved from incomplete or missing ADT data.

The type of road and layout differs wildly between the defined roads. Some roads (Ryan Rd) are more of the type I was initially aiming for while others (Waxpool Rd) are more of reigonal thouroghfares. There are also several intersections in industrial areas that while still data, are not answering the specific question I initially saw. The project was inspired off roads such as Ryan road where medians are large and connecting to residential areas so the addition of industrial roads sways away from the focal point of the study for additional data.

Signal installations are often due to higher volume of traffic or higher crash history meaning they aren't random meaning the intersections are often measures of two fundamentally unequal populations especially when considering residential changes. Also assuming all new lights contained all 2024 and 2025 accidents is a stretch and definitely produced some inaccurate data. I am also sure that the data got merged in the process though most if not all of this addition was lost due to ADT gaps.

Intersections themselves were a buffer, meaning there were several cases of multiple intersections appearing as one and crashes outside of the main intersection being counted. This was largely solved via manually filtering but there is the possibility of some data slipping through the cracks. Also some of the ADT intersections were just the number of roads passing through, which caused some initial hiccups but could be filtered accordingly. There was a lot of manual overriding in this project, nearly each step had manual overrides due to incomplete or extra data so there is bound to be something that slipped through the cracks. 

Finally this study is small and localized to a fault, it is not comprehensive of all of these road types and is bound for local conditions to take effect over the results rather than the road types themselves. 

## Legal notice
Per VDOT: under Title 23 United States Code Section 407, this crash information cannot be used in discovery or as evidence in a Federal or State court proceeding, or considered for other purposes in any action for damages against VDOT or the Commonwealth of Virginia arising from any occurrence at the location identified.

## Reproducing
```Loudon Intersection Crash Map Final.gpkg``` contains the intersection points with computed fields, with layer styles embedded. Open it in QGIS 3.34 or later; the symbology loads as the default style.

## Author
Andrew Sivak - andrew.r.sivak@gmail.com
