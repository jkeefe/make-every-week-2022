# Covid cases animation

[image]

I've been both awed and terrified by the transmissibility of Omicron and the speed at which it's spread. As the case curves hockey-sticked upward and the maps all turned red, I thought it'd be interesting to visualize the spread of this new twist on the virus.

So the other day, while doing some work mapping Covid-19 data by US counties, I realized it wouldn't take much to generate a map for each day of the pandemic ... and make those maps into a movie.

It almost seems like cheating to use a work project as one of my "Make Every Week" projects, but I'm lucky to have a job where creative tinkering is celebrated. When I shared a tinker-made movie of six months of case data with colleagues Kaeti Hinck and Sean O'Key, they thought it would make [a good data feature for CNN](https://www.cnn.com/2022/01/14/us/covid-cases-animation/index.html).

While truly a horrible topic — nearly every county is now reporting more than 100 cases per 100,000 people — the process of turning that case data into a movie was a worthy project. And I did it almost entirely from the command line (the text-only interface that is my Mac's "Terminal" program).

## Case data

The case data for every day in the pandemic, by county, is freely available [in the Github repository](https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_US.csv) maintained by the Johns Hopkins University Center for Systems Science and Engineering. I downloaded it from the command line using the [`curl`](https://curl.se/docs/manual.html) command.

The John's Hopkins data tracks daily _cumulative_ cases by county, so I had to do a little math to get the _new_ cases (the difference from the day before) and the 7-day rolling average for each day.

To get the rate per 100,000 people in each county, I needed the county populations, which are available from the US Census Bureau. I was fortunate to have those tables handy, since a bunch of us worked on them when the 2020 data came out. But you can also get them from the [Census website](https://data.census.gov/cedsci/table?t=Populations%20and%20People&g=0100000US%240500000&y=2020&tid=DECENNIALPL2020.P1) (which I've [downloaded](linktk), too.) The "P1_001N" column is the total population.

Tip: The "FIPS" code, which is the key to joining the Census data to the Johns Hopkins data, is the last five characters of the "GEO_ID" in the population table.

## Command-line cartography

Also available from the [Census website](https://www.census.gov/geographies/mapping-files/time-series/geo/cartographic-boundary.html) are the national county [shapefiles](https://www2.census.gov/geo/tiger/GENZ2020/shp/cb_2020_us_county_20m.zip).

With the data and the shapes by county, I can use the amazing [Mapshaper](https://mapshaper.org) tool's [commands](https://github.com/mbloch/mapshaper/wiki/Command-Reference) to build the maps. The `-i` command takes in the shapefile, the `-join` command marries the `.csv` with the shapefile, and the `-classify` command colors each county based on the case rate, allowing for custom colors and breaks, such as the `50,70,100,250` I used.

Fortunately the `-o` or `-output` command will save the map as an SVG if you use the `.svg` extension on the file name. This key, since SVGs can become images that can become frames of a movie.

I had to run a Mapshaper command for each day — to make a map a day — and I'm not good enough on the command line to make it run loops. But Mapshaper works [programmatically in Nodejs](https://github.com/mbloch/mapshaper/wiki/Using-mapshaper-programmatically#option-two-node-api), so I wrote a little script to do that.

## Command-line movie-making

With a folder full of SVGs, I used [Imagemagick Mogrify](https://imagemagick.org/script/mogrify.php) to batch-format them into PNGs and also adjust each image's "extent," or canvas, to the video-frame size I needed.

I used a couple of other Imagemagick tricks to [draw](https://stackoverflow.com/questions/23236898/add-text-on-image-at-specific-point-using-imagemagick/23238369) the dates on each image. And then I made a legend in Adobe Illustrator and saved it as a PNG, which I then overlaid on every map image using [Imagemagick composite](https://imagemagick.org/script/composite.php).

Finally, I used the command-line program [`ffmpeg`](https://www.ffmpeg.org/download.html) to turn my folder of [images into a video](https://hamelot.io/visualization/using-ffmpeg-to-convert-a-set-of-images-into-a-video/).

## Makefile making

Instead of trying to remember each command-line command as I go along, I've made a habit of writing each command into a [Makefile](https://makefiletutorial.com/). Then I can `make maps` or `make legends`, which then runs the commands to complete the task. It also give me a de-facto, executable log of what worked — which was essential, since I ended up making several iterations before the final version. 

In the end, my `make movie` command takes about five minutes to run ... and when it's done, I have the animation you can see [atop the story](https://www.cnn.com/2022/01/14/us/covid-cases-animation/index.html).

Six months from now, I hope to use the same code to show how Covid-19 disappeared into the background of our lives. 


