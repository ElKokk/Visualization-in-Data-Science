Data Visualisation in Data Science

Prof Jan Aerts, Jelmer Bot


1. General idea



1.2. The data
The data that you will be using:
1.2.1. GPS tracking data
The first dataset contains GPS tracking data of a number of cars by employees of the GAStech corporation, over the span of 14
days.
Properties of the dataset:
• Data is downsampled to 1-minute resolution.
• Data contains information for 40 cars (identifiable by their car_id)
• Location is in latitude/longitude.
• Data is recorded for 14 days, starting at the 6th of the month until and including the 19th.
• Time is recorded in different redundant ways:
1
◦ day, hour, minute
◦ cumulative_minute is the minute since the start of that day. In other words, the cumulative_minute
for day 7 at 9:31am is (9*60+31=) 571
◦ cumulative_minute_total is the minute since the start of the first day (i.e. day "6"). In other words, the
cumulative_minute_total for day 7 at 9:31am is ((7-1)*24*60+9*60+31=) 9,211.
Sample document of GPS tracking data
{
  "key": "35642582",
  "car_id": 1,
  "day": 7,
  "hour": 13,
  "minute": 44,
  "cumulative_minute": 824,
  "long": 24.862910800857144,
  "lat": 36.05667007714286,
  "speed": 99.45714285714286,
  "cumulative_minute_total": 2264
}



1.2.2. Points of Interest


The second dataset concerns different (fictitious) points of interest on the island.
Properties of the dataset:
• There are 65 points of interest.
• They belong to one of 5 types: catering, domestic, housing, professional and other.
Sample document of points of interest
{
  "key": "67864964",
  "name": "Katerina's Cafe",
  "type": "catering",
  "lat": 36.054484865,
  "long": 24.899920025
}


1.2.3. Car stops
The final dataset concerns locations where a car stood still for at least 100 seconds.
Properties of the dataset:
• Time is measured in seconds. As a result, the maximum value for time will be 86,400.
• All entries start and end on the same day. This means that if a car, for example, stands still at a location from 8pm inthe evening until 5am the next morning, there will be 2 entries in the dataset: one standing still on the first day fromb8pm until midnight, and one standing still on the next day from midnight until 5am.
• The car key corresponds to the car_id key in the GPS tracking dataset.
• The name and location_type keys for each location correspond to the name and type keys in the points-ofinterest dataset.

{
  "key": "67866930",
  "car": 101,
  "start": {
  "day": 6,
  "time": 0
  },
  "end": {
  "day": 6,
  "time": 27361
  },
  "location": {
  "name": "GAStech",
  "location_type": "professional"
  }
}




3. Storyboard


3.1. Overview page: visual

• All GPS tracking data (across all days, for all cars)
◦ These locations should be in black and low opacity
• All points of interest, in a different colour per location type
• A dropdown box where you can select a particular car (see below.)
• The image should be 600x600 pixels.
Figure 1. Visual overview


3.2. Overview page: interaction

There should be two types of interaction (see figure):

• When you hover over a point of interest, it should show the location’s name in a tooltip.
• When you select a car in the dropdown box, two things should happen:
◦ All GPS locations of that car should be coloured red instead of black.
◦ A sentence should appear below the dropdown box, which includes a link to that car’s details (see below).



Figure 2. Interaction with the overview


3.3. Details page: visual


The details page for a car should contain the following:
• Your name, university and student number
• A link back to the overview page.
• A link to the next and the previous car.
• A slide bar that allows you to scan through the 14 days: the left end is minute 0 on the first day (day "6"); the right end is
minute 1,440 on the last day (day "19"), which corresponds to cumulative_minute_total of 20,160
• At the left, the same GPS tracking data is shown as in the overview, but only for this particular car.
◦ This image should be 300x300 pixels.
• At the right you show an overview of where that car was (i.e. "stood still") across the whole period that you have data for.
◦ Each day should be a new bar.
◦ The number of the day should be shown at the left.
◦ Different locations should be coloured by the type of that location (i.e. a different colour for professional,
housing, etc).
◦ You should add vertical lines for 0am, 6am, noon, 6pm and midnight, as well as their 24hr number at the
bottom.
◦ This image should be approximately the same size as the image on the left.



Figure 3. Visual details view


3.4. Details page: interaction


There should be two types of interaction (see figure):
• When you hover over a section in the image on the right, it should show the location’s name in a tooltip.
• When you change the value of the slider (which represents the complete time 14-day period), two things should happen:
◦ Any GPS positions that are recorded in a 30-min window around the slider value (i.e. 15 minutes before until 15
minutes after) should be shown in red and full opacity.
◦ A black bar (see white arrow in the image) should be shown in the right image, which indicates which day and
time the slider represents. When you reach the end of a certain day, it should move to the start of the next day.