Data Visualisation in Data Science

Prof Jan Aerts, Jelmer Bot


1. General idea
For the integrated project, we ask you to develop a sveltekit application that includes different visuals.
1.1. Background information on the data
NOTE This is only for your information, and you should not try to answer the questions mentioned in this section.
The data is derived from the 2021 VAST (Visual Analytics in Science and Technology) competition. In this competition,
participants were asked to identify aberrant behaviour by employees of a (fictitious) company named GAStech which was
involved in environment pollution on a (fictitious) Greek island. This included getting a handle on where employees were
during a two-week period.
Here is a simplified tourist map of the region of the island where everything takes place, showing some (but not all) roads and
points of interest:
For this integrated assignment, we make parts of that data available to you.
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
The URL for this dataset is https://vda-lab.github.io/assets/vast2021_gps_coordinates.json
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
The URL for this dataset is https://vda-lab.github.io/assets/vast2021_businesses.json
1.2.3. Car stops
The final dataset concerns locations where a car stood still for at least 100 seconds.
Properties of the dataset:
• Time is measured in seconds. As a result, the maximum value for time will be 86,400.
• All entries start and end on the same day. This means that if a car, for example, stands still at a location from 8pm in
the evening until 5am the next morning, there will be 2 entries in the dataset: one standing still on the first day from
2
8pm until midnight, and one standing still on the next day from midnight until 5am.
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
The URL for this dataset is https://vda-lab.github.io/assets/vast2021_carstops.json
NOTE
You can choose to load the data directly from these websites, or download them within your project. The
choice is yours.
3
2. Setting up
You can either work on your own machine, or use stackblitz. In either case, what you will submit to us will be a zip-file of your
code base (see below).
When developing on your own machine, see the sveltekit website (kit.svelte.dev) for instructions on how to set things up. At the
time of writing this document, this means running npm create svelte@latest my-app, but please check yourself.
When using stackblitz, we will not provide you with a template. At the time of writing this document, you can create a new
sveltekit application by going to http://node.new/sveltekit. This will create a demo application instead of an empty shell, so you
will have to remove some files. These are (among others):
• all files as well as the about directory underneath the routes folder
• the lib/images folder
To download your stackblitz codebase as a zip-file, use the "Download Project" button as shown in the figure below:
NOTE
We cannot guarantee that these instructions are still correct when you start your project. So please set things
up as soon as possible.
IMPORTANT You are responsible for your code. Take precautions so that you do not accidentily loose/delete it.
4
3. Storyboard
Below we describe the specific visuals and interactions that we request from you. In the "Evaluation" section, we will describe
very specifically what components you will be graded on.
3.1. Overview page: visual
As shown in the figure, you should create a page showing:
• Your name, university and student number
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
5
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
6
Figure 3. Visual details view
3.4. Details page: interaction
There should be two types of interaction (see figure):
• When you hover over a section in the image on the right, it should show the location’s name in a tooltip.
• When you change the value of the slider (which represents the complete time 14-day period), two things should happen:
◦ Any GPS positions that are recorded in a 30-min window around the slider value (i.e. 15 minutes before until 15
minutes after) should be shown in red and full opacity.
◦ A black bar (see white arrow in the image) should be shown in the right image, which indicates which day and
time the slider represents. When you reach the end of a certain day, it should move to the start of the next day.
NOTE
There will obviously only be red points in the plot with GPS locations if the black bar in the plot on the
right is near a gap between the sections (because the car is driving at that moment).
7
4. Evaluation
You will receive points for the different aspects of this project:
• In the overview page:
◦ show all GPS positions: 1
◦ show all businesses (incl colour): 1
◦ highlight a single car’s positions when selected: 1
◦ details on hover for a point of interest: 1
◦ working link to the details page: 1
• In the details page:
◦ show GPS postions for only that car: 1
◦ time slider highlights that time period as red points: 1
◦ show the time blocks in the right image: 2
◦ axis added on the time blocks: 1
◦ use different categorical colours for those blocks based on the type of the point of interest: 0.5
◦ details on hover for a block: 0.5
◦ black bar on the time blocks that follows the location in the slider: 1
◦ images are next to each other instead of one below the other: 1
You can receive a bonus point if you use the bootstrap (https://getbootstrap.com/) library to improve the layout (e.g. with
navigation bar).
IMPORTANT
The bonus point will only be awarded if all of the other aspects listed above are completed! In case
you would already have a perfect score of 20/20, this bonus point would obviously not have any
effect.
This amounts to 13/20. The other 7 points are based on the designs that you submitted earlier in the course.
4.1. What to hand in?
An assignment will be created on Toledo/Blackboard where you will submit 2 files:
• a zip-file containing the code in your repository.
◦ Please remove the contents of the node_modules directory before zipping the repository.
◦ We should be able to unzip the file, run npm install (which installs all dependencies), and npm run dev
to get your interface.
• a short video (no sound; in .mov or .mp4 format) of max 30 seconds and max 15MB showing the following (in this
order):
1. on the overview page:
a. You hover over a point of interest (which should show that location’s name)
b. You select a car from the dropdown box (which should show the relevant GPS positions in red)
c. You click on the link going to the details page
2. on the details page:
8
a. You hover over two blocks in the image on the right to show that you get a tooltip
b. You move the slider slowly to the right (which should show that some of the GPS locations become red
and then blue again, as well as the black bar moving across the sections on the right.) In order to keep
the size of teh recording small, you do not have to go over the complete length of the slider. However, the
video should contain at least one transition between 2 days.
9
5. Interaction with the teaching team
The teaching team is of course still available for questions. These should however be focussing on clarification of the
assignment.
For technical questions, please check the teaching material at https://vda-lab.gitlab.io/datavis-technologies/, the exercises, and
google; do not as the teaching team. There is one exception: data loading, because that is a prerequisite for all the aspects of the
evaluation. If you need us to help you load the data itself, we will do so but there will be 1 point subtracted from your score.
10