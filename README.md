# Created by Andrius Daley
import datetime
import numpy as np
import pandas as pd
import os

from math import radians, sin, cos, sqrt, asin


# from geographiclib.geodesic import Geodesic


def haversine(lat1input, lon1input, lat2input, lon2input):
    r = 6372.8  # Earth radius in km

    dlat = radians(lat2input - lat1input)
    dlon = radians(lon2input - lon1input)
    lat_1 = radians(lat1input)
    lat_2 = radians(lat2input)

    a = sin(dlat / 2) ** 2 + cos(lat_1) * cos(lat_2) * sin(dlon / 2) ** 2
    c = 2 * asin(sqrt(a))

    return r * c  # Returns values in km unit

def conversion_km_to_meter(distance_km_input):
    distance_meter = distance_km_input * 0.001

    return distance_meter


def conversion_km_to_feet(distance_km_input):
    distance_feet_converted = distance_km_input * 3280.84

    return distance_feet_converted


def conversion_km_to_miles(distance_km_input):
    distance_miles = distance_km_input * 0.621371

    return distance_miles


def conversion_seconds_to_hours(slicedDeltaTimeInt_input):
    time_hours = slicedDeltaTimeInt_input * 0.000277778

    return time_hours


def conversion_mph_to_knots(speed_mph):
    speed_knots = speed_mph * 0.868976

    return speed_knots


# Adjust to display full fields
desired_width = 320
pd.set_option('display.width', desired_width)
np.set_printoptions(linewidth=desired_width)
pd.set_option('display.max_columns', 22)
pd.set_option('display.precision', 8)

# Initialize Files in Folder
filesInFolder = []

# Input Pathway
input_pathway = r'C:\Users\Andrius\Desktop\GIS Data\Kayaking\Text'

# for loop to loop through kayak data in folder
for filename in os.listdir(input_pathway):
    if filename.endswith('.txt'):
        filesInFolder.append(filename)

        # Open CSV file
        expedition_file = input_pathway + "\\" + filename
        df = pd.read_csv(expedition_file, sep=',')
        print(expedition_file)
        # Initialize index values
        i = 0  # First Loop - Obtain Point 2 Data
        u = 0  # Second Loop - Obtain Point 1 Data

        # Initialize float and integer variables
        total_distance_miles = 0.0
        total_time_hours = 0

        for l2, r2 in df.iterrows():  # Loop skips over first line to obtain point 2 data
            if i == 0:
                i += 1
                continue
            else:
                # print(" i = {}".format(i))

                lat2 = r2['Latitude']
                lon2 = r2['Longitude']
                time2 = r2['time']

                if time2[-3:] != "+00":
                    time2 = time2 + "+00"

                # print("time2 = {0} and type = {1}".format(time2,type(time2)))
                
                # Convert time2 string to datetime Object
                time2object = (datetime.datetime.strptime(time2, "%Y/%m/%d %H:%M:%S+00"))

                for l1, r1 in df.iloc[u:].iterrows():  # Second Loop begins to obtain point 1 data (1 step behind
                    # loop l2)
                    lat1 = r1['Latitude']
                    lon1 = r1['Longitude']
                    time1 = r1['time']

                    if time1[-3:] != "+00":
                        time1 = time1 + "+00"

                    # Convert time1 string to datetime Object
                    time1object = (datetime.datetime.strptime(time1, "%Y/%m/%d %H:%M:%S+00"))

                    deltaTime = time2object - time1object
                    deltaTimeStr = str(deltaTime)
                    slicedDeltaTimeStr = deltaTimeStr[5:7]

                    slicedDeltaTimeInt = int(slicedDeltaTimeStr)

                    timeHours = conversion_seconds_to_hours(slicedDeltaTimeInt)
                    if timeHours == 0.0:
                        timeHours += 0.0001

                    total_time_hours += timeHours

                    distance_km = haversine(lat1, lon1, lat2, lon2)

                    distance_feet = conversion_km_to_feet(distance_km)

                    distance_miles = conversion_km_to_miles(distance_km)

                    total_distance_miles += distance_miles

                    kayakingSpeed_mph = distance_miles / timeHours

                    kayakingSpeed_Knots = conversion_mph_to_knots(kayakingSpeed_mph)

                    df.at[l1, "Speed_mph"] = kayakingSpeed_mph
                    df.at[l1, "Speed_Knots"] = kayakingSpeed_Knots

                    i += 1
                    u += 1
                    break

        path = r'C:\Users\Andrius\Desktop\GIS Data\Processed_Kayak_Data'

        filename_striped = filename[:-4]
        expedition_location = filename_striped[11:]
        expedition_date = filename[:10]
        print(expedition_location)
        print(expedition_date)
        print(df)
        print("Total Distance (miles): {0}\nTotal Time (hours): {1}".format(total_distance_miles, total_time_hours))

        df.to_csv(os.path.join(path, expedition_date + "_" + expedition_location + "_" + "Processed_V1.csv"))
print(filesInFolder)
print(len(filesInFolder))
