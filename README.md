# World_Weather_Analysis

## Overview
At PLANMYTRIP, an internet related services company in the hotel and lodging industry, I was tasked with presenting data to optimize the customer search page. We used pandas dataframes, OpenWeatherMap API, random number generation, the citipy mododule, geoviews, and Geoapify API to randomly generate ideal travel destination for customers. We then found hotels near these cities to help craft the perfect vacation destinations for our customers.


### Purpose
We randomly generated 2000 pairs of latitude and longitude geocoordinates and found the nearest cities with the citipy module. Using pandas dataframes and geoviews, cities with weather conditions meeting the travel needs of customers were displayed on a global map. We made API calls using the geoapify API call system and found hotels nearby the customers preferred cities of choice. We then suggested four hotels to the customer, displayed these hotels on a travel itinerary map, and created an overlay for the route of the trip.

## Analysis

Using the NumPy module and the `random.uniform()function` we randomly generated 2000 latitudes and longitudes as geocoordinates.

`lats = np.random.uniform(-90,90,size=2000)`

`lngs = np.random.uniform(-180,180,size=2000)`

The lattitude and longitude lists were then zipped into a list of latitude and longitude pairs as tuples. This was accomplished using the zip function. The tuples were then converted to lists:

`lat_lngs = zip(lats,lngs)`

`coordinates = list(lat_lngs)`

We used the citipy moule and the `nearest_city()` function to get the closest cities near these coordinates. We added the generated cities to a list called `cities`:

`city = citipy.nearest_city(coordinate[0],coordinate[1]).city_name`



 `if city not in cities:`

&nbsp;&nbsp;&nbsp;&nbsp;`cities.append(city)`

Using the OpenWeatherMap API, a response of weather data from these cities was delivered in `.json` format. Our API calls navigated through dictionary and list data structures to retrieve the specific pieces of information we requested such as maximum temperature of a city, humidity, clouds, wind speed, and a general description of the current weather. This information was appended to a new dataframe, row by row, as each API call per city was made. A try and except structure was established so that geoocordinates without a nearby city would not end the code or ellicit a traceback. The resulting dataframe was created as summarized by the following code block:


`city_data = []`


`print("Beginning Data Retrieval     ")`

`print("-----------------------------")`

`record_count = 1`

`set_count = 1`

`for i, city in enumerate(cities):`
        
`# Group cities in sets of 50 for logging purposes, reduces the API call load on the servers`

&nbsp;&nbsp;&nbsp;&nbsp;`if (i % 50 == 0 and i >= 50):`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`set_count += 1`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`record_count = 1`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`time.sleep(60)`

&nbsp;&nbsp;&nbsp;&nbsp;`city_url = url + "&q=" + city.replace(" ","+")`
    
&nbsp;&nbsp;&nbsp;&nbsp;`print(f"Processing Record {record_count} of Set {set_count} | {city}")`

&nbsp;&nbsp;&nbsp;&nbsp;`# Add 1 to the record count`

&nbsp;&nbsp;&nbsp;&nbsp;`record_count += 1`

&nbsp;&nbsp;&nbsp;&nbsp;`# Run an API request for each of the cities`

&nbsp;&nbsp;&nbsp;&nbsp;`try:`
        
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city_weather = requests.get(city_url).json()`
        
        
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`# Parse out the latitude, longitude, max temp, humidity, cloudiness, wind, country, and weather description`
        
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city_lat = city_weather["coord"]["lat"]`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city_lng = city_weather["coord"]["lon"]`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city_max_temp = city_weather["main"]["temp_max"]`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city_humidity = city_weather["main"]["humidity"]`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city_clouds = city_weather["clouds"]["all"]`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city_wind = city_weather["wind"]["speed"]`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city_country = city_weather["sys"]["country"]`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city_description = city_weather["weather"][0]["description"]`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`# Convert the date to ISO standard.`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city_date = datetime.utcfromtimestamp(city_weather["dt"]).strftime('%Y-%m-%d %H:%M:%S')`

            
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`# Append the city information into the city_data list`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city_data.append({"City": city.title(),`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`"Lat": city_lat,`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`"Lng": city_lng,`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`"Max Temp": city_max_temp,`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`"Humidity": city_humidity,`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`"Cloudiness": city_clouds,`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`"Wind Speed": city_wind,`
                          
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`"Country": city_country,`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`"Current Description":city_description,`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`"Date": city_date})`
        
    
&nbsp;&nbsp;&nbsp;&nbsp;`# If an error is experienced, skip the city`

&nbsp;&nbsp;&nbsp;&nbsp;`except:`


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`print("City not found. Skipping...")`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`pass`

`# Indicate that the data retrieval is complete `

`print("-----------------------------")`

`print("Data Retrieval Complete      ")`

`print("-----------------------------")`

The resulting dataframe was then exported as a `.csv` file named `WeatherPy_Database.csv`. 

We then wrote code to ask the customer what maximum and minimum temperature they wanted to experience on their vacation.  A new dataframe was then constructed using the `.loc` function with conditionals to satisfy the customer's needs. This function was accomplished with:

`min_temp = float(input("What is the minimum temperature you would like for your trip? "))`

`max_temp = float(input("What is the maximum temperature you would like for your trip? "))`

`preferred_cities_df = city_data_df.loc[`

&nbsp;&nbsp;&nbsp;&nbsp;`(city_data_df["Max Temp"] <= max_temp) & (city_data_df["Max Temp"] >= min_temp)`

`]`

We then used an iterated loop to perform Geoapify API calls to get the nearby hotels for our customers preferred travel cities. 

`# Print a message to follow up the hotel search`

`print("Starting hotel search")`

`# Iterate through the hotel_df DataFrame`

`for index, row in hotel_df.iterrows():`

&nbsp;&nbsp;&nbsp;&nbsp;`# Get latitude and longitude from DataFrame.`

&nbsp;&nbsp;&nbsp;&nbsp;`latitude = hotel_df.loc[index,"Lat"]`

&nbsp;&nbsp;&nbsp;&nbsp;`longitude = hotel_df.loc[index,"Lng"]`
    
&nbsp;&nbsp;&nbsp;&nbsp;`# Add the current city's latitude and longitude to the params dictionary`

&nbsp;&nbsp;&nbsp;&nbsp;`params["filter"] = f"circle:{longitude},{latitude},{radius}"`

&nbsp;&nbsp;&nbsp;&nbsp;`params["bias"] = f"proximity:{longitude},{latitude}"`

    
&nbsp;&nbsp;&nbsp;&nbsp;`# Set up the base URL for the Geoapify Places API.`

&nbsp;&nbsp;&nbsp;&nbsp;`base_url = "https://api.geoapify.com/v2/places"`
    
&nbsp;&nbsp;&nbsp;&nbsp;`# Make request and retrieve the JSON data by using the params dictionary`

&nbsp;&nbsp;&nbsp;&nbsp;`name_address = requests.get(base_url, params=params)`
    
&nbsp;&nbsp;&nbsp;&nbsp;`# Convert the API response to JSON format`

&nbsp;&nbsp;&nbsp;&nbsp;`name_address = name_address.json()`
    
&nbsp;&nbsp;&nbsp;&nbsp;`# Get the first hotel from the results and store the name, if a hotel isn't found set the hotel name as "No hotel found".`

&nbsp;&nbsp;&nbsp;&nbsp;`try:`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`hotel_df.loc[index, "Hotel Name"] = name_address["features"][0]["properties"]["name"]`

&nbsp;&nbsp;&nbsp;&nbsp;`except (KeyError, IndexError):`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`# If no hotel is found, set the hotel name as "No hotel found".`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`hotel_df.loc[index, "Hotel Name"] = "No hotel found"`
        
&nbsp;&nbsp;&nbsp;&nbsp;`# Log the search results`

&nbsp;&nbsp;&nbsp;&nbsp;`print(f"{hotel_df.loc[index, 'City']} - nearest hotel: {hotel_df.loc[index, 'Hotel Name']}")`

The resulting dataframe from this iterated loop of API calls contained the all the hotel information we needed for our customers. A plot was subsequently generated and was stored in the `Vacation_Search` folder of this github repository. 

![WeatherPy_vacation_map](https://github.com/willmino/World_Weather_Analysis/blob/main/Vacation_Search/WeatherPy_vacation_map.png)

From our list of hotels, we picked four locaitons in Brazil we thought would be perfect for our customers. These cities were Rio Grande, Cidreira, and Araquari. These were all beautiful coastal cities. The `.loc` function with conditionals were used to construct 4 separate dataframes for each city which were then merged into a single dataframe.

`vacation_start = vacation_df.loc[vacation_df["City"] == "Rio Grande"]`

`vacation_end = vacation_df.loc[vacation_df["City"] == "Rio Grande"]`

`vacation_stop1 = vacation_df.loc[vacation_df["City"] == "Cidreira"]`

`vacation_stop2 = vacation_df.loc[vacation_df["City"] == "Araquari"]`

`vacation_stop3 = vacation_df.loc[vacation_df["City"] == "Castro"]`

`frames = [vacation_start, vacation_stop1, vacation_stop2, vacation_stop3,vacation_end]`

`itinerary_df = pd.concat(frames)`

Using another iterated loop to perform API calls, we generated all of the coordinates along the route of the customer's trip. We plotted the final vacation destinations and overlayed the trip route for our customer's final itinerary map.

`# Set parameters to trace the route`

`radius = 5000`

`params = {`

&nbsp;&nbsp;&nbsp;&nbsp;`"mode":"drive",`

&nbsp;&nbsp;&nbsp;&nbsp;`"apiKey": geoapify_key,`

`}`

`# Set an empty waypoints String variable`

`waypoints = ""`

`# Iterate through the route_df DataFrame to define the waypoints`

`for index, row in waypoints_df.iterrows():`

&nbsp;&nbsp;&nbsp;&nbsp;`waypoints = waypoints + str(row["Lat"]) + "," + str(row["Lng"]) + "|"`

&nbsp;&nbsp;&nbsp;&nbsp;`# Delete the last character from the string`

&nbsp;&nbsp;&nbsp;&nbsp;`waypoints = waypoints[:-1]`

&nbsp;&nbsp;&nbsp;&nbsp;`# Add the waypoints to the params dictionary`

&nbsp;&nbsp;&nbsp;&nbsp;`params["waypoints"] = waypoints`

`# Display the params dictionary`

`params`

`# Set up the base URL for the Geoapify Places API.`

`base_url = "https://api.geoapify.com/v1/routing"`

`# Make request and retrieve the JSON data by using the params dictionary`

`route_response = requests.get(base_url, params=params)`

`# Convert the API response to JSON format`

`route_response = route_response.json()`

`# Fetch the route's legs coordinates from the JSON reponse`

`legs = route_response["features"][0]["geometry"]["coordinates"]`

To specifically construct the trip route on the itinerary map, we iterated through the response json set to variable called `legs`. The longitudes and latitudes from every point of the route on the trip were added to a new dictionary called `route_df`.

`# Create and empty list to store the longitude of each step`

`longitudes = []`

`# Create and empty list to store the latitude of step`

`latitudes = []`



`# Loop through the legs coordinates to fetch the latitude and longitude for each step`

`for index in legs:`

&nbsp;&nbsp;&nbsp;&nbsp;`for coordinates in index:`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`longitudes.append(coordinates[0])`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`latitudes.append(coordinates[1])`

`# Create an empty DataFrame to store the steps' coordinates`

`route_df = pd.DataFrame()`

`# Add the steps' longitude and latitude from each step as columns to the DataFrame`

`route_df["longitude"] = longitudes`

`route_df["latitude"] = latitudes`

`route_df`


Then, `route_df` was incoporated into the final output code for the itinerary map display.

`# Configure the route path by using the GeoViews' Path function`

`route_path = gv.Path(route_df).opts(color="red",line_width=4,alpha=0.3)`

`route_path`


`route_map = waypoints_df.hvplot.points(`
&nbsp;&nbsp;&nbsp;&nbsp;`"Lng",`

&nbsp;&nbsp;&nbsp;&nbsp;`"Lat",`

&nbsp;&nbsp;&nbsp;&nbsp;`geo = True,`

&nbsp;&nbsp;&nbsp;&nbsp;`tiles = "OSM",`

&nbsp;&nbsp;&nbsp;&nbsp;`frame_width = 500,`

&nbsp;&nbsp;&nbsp;&nbsp;`frame_height = 500,`

&nbsp;&nbsp;&nbsp;&nbsp;`color = "City",`

&nbsp;&nbsp;&nbsp;&nbsp;`size = 300,`

&nbsp;&nbsp;&nbsp;&nbsp;`alpha = 0.6,`
    
&nbsp;&nbsp;&nbsp;&nbsp;`#by adding hover_cols, we can see the info associated with each city,`

&nbsp;&nbsp;&nbsp;&nbsp;`hover_cols = ["City", "Country","Current Description"]`

`)`

`composed_map = route_map*route_path`

`composed_map`

![WeatherPy_travel_map.png](https://github.com/willmino/World_Weather_Analysis/blob/main/Vacation_Itinerary/WeatherPy_travel_map.png)








