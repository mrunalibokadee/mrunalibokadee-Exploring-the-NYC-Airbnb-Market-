# mrunalibokadee-Exploring-the-NYC-Airbnb-Market-
1 Importing the Data

import pandas as pd
import numpy as np
import datetime as dt

# Load airbnb_price.csv, prices
prices = pd.read_csv("/content/airbnb_price.csv")

# Print the first five rows of each DataFrame
prices.head()

2. Clean the price column

# Remove whitespace and string characters from prices column
prices["price"] = prices["price"].str.replace(" dollars", "")

# Convert prices column to numeric datatype
prices["price"] = pd.to_numeric(prices["price"])

# Print 1st 5 rows
print(prices["price"].head())

# Print descriptive statistics for the price column
print(prices["price"].describe())

3. Calculating Average Price

prices.info()

# Subset prices for listings costing $0 named "free_listings"
free_listings = prices["price"] == 0
print(type(free_listings))
print(free_listings.shape)

# Update prices by removing all free listings from prices
# Similar to SQL's concept of "NOT IN"
prices = prices.loc[~free_listings]

# Calculate the average price and round to nearest 2 decimal places, avg_price
avg_price = round(prices["price"].mean(),2)

# Print the average price
print("The average price per night for an Airbnb listing in NYC is ${}.".format(avg_price))

4. comparing cost to the private rental market

prices.info()

prices.head()

# Add a new column to the prices DataFrame, price_per_month
prices["price_per_month"] = prices["price"] * 365 / 12
# print(type(prices["price_per_month"]))

# Calculate average_price_per_month
average_price_per_month = round(prices["price_per_month"].mean(),2)                     
                                                       
# Compare Airbnb and rental market
print("Airbnb monthly costs are ${}, while in the private market you would pay {}.".format(average_price_per_month, "$3,100.00"))

5. Timeframe are we working with

reviews.head()
reviews.info()
# Change the data type of the last_review column to datetime
# https://pandas.pydata.org/docs/reference/api/pandas.to_datetime.html
reviews["last_review"] = pd.to_datetime(reviews["last_review"])
print(type(reviews["last_review"]))

# Create first_reviewed, the earliest review date
first_reviewed = reviews["last_review"].dt.date.min()

# Create last_reviewed, the most recent review date
last_reviewed = reviews["last_review"].dt.date.max()

# Print the oldest and newest reviews from the DataFrame
print("The latest Airbnb review is {}, the earliest review is {}".format(last_reviewed, first_reviewed))

print(reviews.dtypes["last_review"])
reviews["last_review"].head()

6. Joining the dataframes

# Merge prices and reviews to create prices_and review
# https://pandas.pydata.org/docs/user_guide/merging.html
rooms_and_prices = pd.merge(prices, reviews, 
                            how="outer", 
                            on="listing_id")

# Drop missing values from airbnb_merged
rooms_and_prices.dropna(inplace=True)

# Check if there are any duplicate values
print("There are {} duplicates in the DataFrame.".format(rooms_and_prices.duplicated().sum()))

print(rooms_and_prices.info())
rooms_and_prices.head()

7. Analyzing listing prices by NYC borough

# Extract information from the nbhood_full column and store as a new column, borough
# Either use `.str.partition()` or `.str.split()`
rooms_and_prices["borough"] = rooms_and_prices["nbhood_full"].str.partition(",", expand=True)[0]

# Group by borough and calculate summary statistics
boroughs = rooms_and_prices.groupby("borough")["price"].agg(["sum", "mean", "median", "count"])

# Round boroughs to 2 decimal places, and sort by mean in descending order
boroughs = boroughs.round(2).sort_values("mean", ascending=False)

# Print boroughs
print(boroughs)

# Create labels for the price range, label_names
label_names = ["Budget", "Average", "Expensive", "Extravagant"]

# Create the label ranges, ranges
ranges = [0, 69, 175, 350, np.inf]

# Insert new column, price_range, into DataFrame
# Use `pd.cut` to segment and sort data values into bins
# Useful for going from a continuous variable to a categorical variable 
rooms_and_prices["price_range"] = pd.cut(rooms_and_prices["price"], bins=ranges, labels=label_names)

# Calculate borough and price_range frequencies, prices_by_borough
prices_by_borough = rooms_and_prices.groupby(["borough", "price_range"])["price_range"].agg("count")
print(prices_by_borough)

print(ranges)
print(type(ranges))
