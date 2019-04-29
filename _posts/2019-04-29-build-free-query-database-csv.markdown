---
layout: post
title:  "How to Build a Browsable Web User Interface From a CSV File For Free Without Writing a Line of Code"
date:   2019-04-29T09:09:00Z
---

These are the steps I used to take a csv file and turn it into browsable database for free using [Datasette](https://datasette.readthedocs.io/) and [Heroku](https://www.heroku.com/).
The data used is parking infraction data from the [City of Waterloo Open Data](http://data.waterloo.ca/).

Create a new directory and set up a python virtual environment. Download the data from [here](https://opendata-city-of-waterloo.opendata.arcgis.com/datasets/city-of-waterloo-bylaw-parking-infractions) and save it in the directory as waterloo_parking_infractions.csv.


Install the [Datasette](https://datasette.readthedocs.io/) and [csvs-to-sqlite](https://github.com/simonw/csvs-to-sqlite) packages:

```
pip install datasette
pip install csvs-to-sqlite
```

Convert the csv file to an sqlite database:

```
csvs-to-sqlite waterloo_parking_infractions.csv waterloo_parking_infractions.db
```

To run locally, run this command:

```
datasette serve waterloo_parking_infractions.db
```

You can then open a browser and go to http://127.0.0.1:8001 to browse the database.

To publish using Heroku, create an account and install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli):

Create a metadata.json file in the directory. This is optional, but allows us to display some information on the dataset we are sharing.

```
{
    "title": "City of Waterloo Bylaw Parking Infractions",
    "description": "Contains records pertaining to parking infractions within the City of Waterloo. Details of each infraction including date, street name, and issued fine are included.",
    "source": "City of Waterloo Open Data",
    "source_url": "https://opendata-city-of-waterloo.opendata.arcgis.com/datasets/city-of-waterloo-bylaw-parking-infractions"
}
```

Publish to Heroku. For more information see the docs [here](https://datasette.readthedocs.io/en/stable/publish.html):

```
datasette publish heroku waterloo_parking_infractions.db -n waterloo-parking-infractions -m metadata.json --extra-options="--config sql_time_limit_ms:5500"
```

View in browser [here](https://waterloo-parking-infractions.herokuapp.com/waterloo_parking_infractions-59806a7)

Now you can run SQL queries on the database in the browser. Here are a few sample queries.

* [View total fines given out by year](https://waterloo-parking-infractions.herokuapp.com/waterloo_parking_infractions-c7b5982?sql=select+strftime%28%27%25Y%27%2C+ISSUEDATE%29+as+year%2C+sum%28VIOFINE%29+as+total_fines+from+waterloo_parking_infractions%0D%0Agroup+by+year)

* [View total fines given out by day of week (0 is Sunday)](https://waterloo-parking-infractions.herokuapp.com/waterloo_parking_infractions-c7b5982?sql=select+strftime%28%27%25w%27%2C+ISSUEDATE%29+as+day_of_week%2C+sum%28VIOFINE%29+as+total_fines+from+waterloo_parking_infractions%0D%0Agroup+by+day_of_week)

* [View total fines given out by hour of day](https://waterloo-parking-infractions.herokuapp.com/waterloo_parking_infractions-c7b5982?sql=select+strftime%28%27%25H%27%2C+ISSUEDATE%29+as+hour%2C+sum%28VIOFINE%29+as+total_fines+from+waterloo_parking_infractions%0D%0Agroup+by+hour)

* [View the location/reason with the most total fines in 2018](https://waterloo-parking-infractions.herokuapp.com/waterloo_parking_infractions-c7b5982?sql=select+STREET%2C+REASON%2C+sum%28VIOFINE%29+as+total_fines+from+waterloo_parking_infractions%0D%0Awhere+ISSUEDATE+between+%272018-01-01%27+and+%272019-01-01%27%0D%0Agroup+by+STREET%2C+REASON%0D%0Aorder+by+total_fines+desc)

All in all, this makes for a quick free way to set up to do some exploratory analysis and easily share it with anyone.
