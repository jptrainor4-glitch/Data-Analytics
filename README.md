Conclusion:
Top 5 results:

Sensor ID | Lat \ Long              |  Top-10s  |  200m Radius, no outliers gap score
25        	-37.824018	144.956044	   4	         0.655102
24        	-37.818880	144.954492	   4	         0.416327
133        	-37.815314	144.952278	   4	         0.344898
5	          -37.818742	144.967877	   2           0.520408
41	        -37.816686	144.966897	   2	         0.500000


How to read the gap score: each sensor gets a demand rank (how busy it is compared with the others) and a supply rank (how many bar seats are nearby compared with the others). Both ranks run from 0 to 1. The gap score is demand minus supply, so +1 means busiest with the least competition and −1 means quiet with a lot of seats.

Final results are based on four top-10s: supply was measured four ways: within 200m and within 300m, each with and without outlier venues (stadiums, function centres, theatre bars). A sensor in all four top-10s doesn't depend on any one of those choices.

How to red this repo:

01. bar_cleaning.ipynb	Cleans and flags the CLUE bar seat data
02. foot_traffic.ipynb	Turns hourly counts into typical Early/Late traffic per sensor	TODO sensor CSV name
03. gap_analysis.ipynb	Joins the two, scores gap scores, ranks, maps

Data Soucre:
Bar, tavern and pub patron capacity (CLUE) -	City of Melbourne Open Data	Census year 2024: 304 Property IDs (raw file has 5,304 rows)
Pedestrian Counting System, hourly counts -	City of Melbourne Open Data	Oct 2024 to Oct 2026, 4pm to 11pm, CBD box padded ~300m: 100 sensors (478,706 rows)

Method and key decisions:
Bar cleaning: missing Lat/Long filled with the mean for each Property ID. Outlier venues are flagged, not dropped. Filtered to 2024, the census year closest to the sensor window.

Foot traffic:
Early/Late split at 19:00. The 17:00 commuter peak has faded by 19:00, so Late (19:00 to 22:00) is the bar-relevant window.
Median first. Counts are right-skewed, and busier, outlier nights (~2.6% of dataset) were kept rather than removed.
Mean of medians across hours and seasons. This gives every hour and season an equal say, so sensors that miss late hours or winter months aren't biased. Weak sensors (<90% complete) are flagged, not dropped. Each one is the only reading for its patch.

Gap analysis:
Seats summed within 200m and 300m of each sensor, using haversine.
2 sensors outside the bar data's extent were dropped, because their zero seats reflect missing coverage, not a real gap.
Demand = 70% Late rank + 30% Early rank. Late trade is a bar's core market. Ranks put both periods on the same scale, so Early's bigger volume doesn't dominate.

Limitations:
Transit-heavy traffic. Southern Cross and Flinders St are stations, and the sensors count bodies, not intent. Many people are walking through to a train. Check dwell time on the ground.
Crown. The sensor sits far enough from the complex to count as its own space, but there is little structures built around it, so it may not be a viable site. Separately, filling Lat/Long with the Property ID means condensing all crowns venues into one geographic point skewing many venues into one specific space on the map.
Static seats. 2024 seat counts are compared with 2025–26 traffic, which assumes supply hasn't changed. Update when a new CLUE year is released.
Clustering (Hotelling's law, agglomeration). Bars group together on purpose. A high seat count can mean proven demand rather than saturation, and an empty area may be empty for a reason. The score finds under-served traffic, not a proven market.
Short window. Two years of sensor data, and one census year of bars by choice (CLUE goes back to 2002).
Bounding box. The bar data's coverage was approximated with a min/max lat/lon rectangle. The City of Melbourne boundary isn't a rectangle.
70/30 is a judgement call, not fitted to trade data.
