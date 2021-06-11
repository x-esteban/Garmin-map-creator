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

## 2. Creating our map

Our objective is to display all our routes on a single map, which can be achieved quite easily now that our *.fit* files are nicely parsed. Our tool of choice will be **Folium**, since it supports a variety of [tiles](https://pypi.org/project/folium/0.1.4/) and built-in functions to make our maps more visually appealing. For this example I have chosen the tileset **OpenStreetMap**, as it provides a great deal of information on towns, roads and intersections.

To create a map with **Folium** we would usually need to pass the center coordinates, but in this case we will use the function *fit_bounds* to automatically center our map on the displayed routes with the according zoom level.

```
fit_map = folium.Map(tiles="openstreetmap") #Calling our tiles.

for i in point_list: #Using a loop to add every route to the map.
    try:
        folium.PolyLine(i, color='blue', weight=1.5, opacity=0.8).add_to(fit_map) #Tweaking line parameters.
    except:
        pass
    
fit_map.fit_bounds(fit_map.get_bounds()) #Fitting our map to the routes, at max zoom.
```

Once our map has finished rendering we can display it like so.

```
fit_map
```

We will be greeted with a map similar to this. While this is a static image, the real map is fully interactive and zoomable.

![Image](https://i.ibb.co/ZHXmsds/openstreetmap.png)

We can change the line color, width and transparency by modifying the parameters shown in the code above.

If we need to display the map elsewhere or simple save it, we can achieve it by this simple command:

```
fit_map.save(outfile= "<map_name>.html")
```

It will be stored in the same folder as our notebook.


## 3. Giving every ride a different color.

While the previous map achieved its purpose, it was quite difficult to make out every single ride unless we zoomed right in. This could be fixed by giving each ride a random color and displaying them on a dark, minimalist tileset such as the one provided by **CartoDB Dark Matter**.

First, let's import the *random* library to perform our random choices.

```
import random
```

Defining a list of colors for the routes. The dark ones have been left out since they don't pop on a black map.

```
color_list = ['red', 'blue', 'green', 'purple', 'orange', 'lightred', 'cadetblue', 'pink', 'lightblue', 'lightgreen']
```

Creating our map.

```
color_map = folium.Map(tiles="cartodbdark_matter")

for i in point_list:
    try:
        folium.PolyLine(i, color=random.choice(color_list), weight=1.5, opacity=1).add_to(color_map)
    except:
        pass
    
color_map.fit_bounds(color_map.get_bounds())
```

Displaying our map.

```
color_map
```
The result will be quite similar to this (with different routes, of course):

![Image](https://i.ibb.co/dGHjcxW/cartoDB1.png)

The different colors make our routes more easily distinguished, especially if we zoom in.

![Image](https://i.ibb.co/X7rZBxg/cartoDB2.png)
