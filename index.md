# Generating maps with our Garmin data

If you own a **Garmin** you have access to a wide variety of riding/training data, but most of it cannot be visualized in bulk. This is specially true for geospatial data, but thankfully **Python** and some ingenuity can easily solve this issue.

A few steps must be followed to gather all the data needed for our maps: 

<br>

**1.** Visit [this link](https://support.garmin.com/es-ES/?faq=W1TvTPW8JZ6LfJSfK512Q8) and follow the instructions to request **Garmin** your personal data:
   
   
**2**. Wait a few hours and follow the link sent to your email. Download the contents.


**3.** Extract the downloaded files and navigate to the folder .../garmin/DI_CONNECT/DI-Connect-Fitness-Uploaded-Files .


**4.** Copy all *.fit* files in that folder, put them in a new folder called *fit* and copy it to the same folder as this notebook.

<br>

Done? <br>
Let's move to the next step.

## 1. Defining a parser function for *.fit* files

To visualize our *.fit* files in our maps we must first parse them to obtain both latitude and longitude for every instant of the ride.

Import the necessary libraries:

```
import fitdecode   
import os
import folium
```

Creating a parser function that will convert a *.fit* file into a list of tuples containing the latitude/longitude of every point in the ride.

```
def parser (file):
    points = []
    with fitdecode.FitReader(file) as fit: #Opening our file.
        for frame in fit:
            if isinstance(frame, fitdecode.FitDataMessage):
                try:
                    points.append(tuple([(frame.get_value('position_lat')/ ((2**32)/360)), #Appending the lat.
                                         (frame.get_value('position_long')/ ((2**32)/360))])) #Appending long.
                except:
                    pass
    return points #Returning our list of lat/long tuples.
```

Now that we have our function ready we will make use of *scandir* to parse every file in our */fit* directory.

```
with os.scandir('fit') as entries: #Scanning our directory.
    point_list = [] 
    for entry in entries: #Walking through every single file.
        point_list.append(parser(entry))
```

![Image](src)
