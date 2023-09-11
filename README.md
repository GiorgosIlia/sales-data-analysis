# Sales Data Analysis

This Jupyter notebook contains Python code for analyzing sales data using the pandas library and visualizing the results using Matplotlib. The code is divided into sections, each focusing on different aspects of the sales data analysis.

## Getting Started

Before running the code, make sure you have the necessary Python libraries installed, such as pandas, glob, and Matplotlib. You can install them using the following command:

```python
pip install pandas glob2 matplotlib
```

## Loading and Combining Data

The code starts by loading multiple CSV files containing sales data, located in the specified directory, and combines them into a single DataFrame.

```python
import pandas as pd
import glob
import os

# Specify the path to the directory containing CSV files
path = "\\Users\\jegen\Desktop\\visualStudio\\data_science\\Sales_Data"
files = os.path.join(path, "Sales*.csv")
files = glob.glob(files)

# Combine all CSV files into a single DataFrame
df = pd.concat(map(pd.read_csv, files), ignore_index=True)
```

## Data Preprocessing

### Extracting Month from Order Date

The code extracts the month from the "Order Date" column and removes rows with invalid date entries.

```python
df["Month"] = df["Order Date"].str[0:2]
df = df[df["Month"] != "Or"]
df["Month"] = df["Month"].astype("int32")
```

### Calculating Total Cost

The total cost for each order is calculated by multiplying "Price Each" and "Quantity Ordered."

```python
df["Price Each"] = df["Price Each"].astype("float64")
df["Quantity Ordered"] = df["Quantity Ordered"].astype("int32")
df["Total Cost"] = df["Price Each"] * df["Quantity Ordered"]
```

## Data Analysis and Visualization

### Monthly Sales

The code groups the data by month and calculates the total sales for each month, then visualizes the results using a bar chart.

```python
import matplotlib.pyplot as plt

results = df.groupby("Month").sum()
months = range(1, 13)
names = plt.bar(months, results["Total Cost"])
plt.ylabel("Price in USD ($)")
plt.xlabel("Months")
plt.show()
```

### Top Selling Cities

The code extracts city and state information from the "Purchase Address" column, groups the data by city, and calculates the total sales for each city. It then visualizes the results using a bar chart.

```python
def get_city(address):
    return address.split(",")[1]

def get_state(address):
    return address.split(",")[2].split(" ")[1]

df["City"] = df["Purchase Address"].apply(lambda x: get_city(x) + " (" + get_state(x) + ")")

final = df.groupby("City").sum()
final.drop(columns=['Month', 'Quantity Ordered'])

cities = [city for city, final in final.groupby("City")]

names = plt.bar(cities, final["Total Cost"])
plt.xticks(cities, rotation="vertical")
plt.ylabel("Sales")
plt.xlabel("Cities")
plt.show()
```

### Sales by Hour

The code extracts the hour from the "Order Date" column and visualizes the number of sales for each hour using a line chart.

```python
def get_hours(hours):
    yolo = hours.split(":")[0]
    return yolo.split(" ")[1]

df["Time"] = df["Order Date"].apply(lambda x: get_hours(x))

hour = [time for time, df in df.groupby("Time")]

plt.plot(hour, df.groupby(["Time"]).count())
plt.grid()
plt.ylabel("Sales")
plt.xlabel("Time")
plt.show()
```

### Products Sold Together

The code identifies products that are sold together by grouping rows with the same "Order ID" and then counting pairs of products that are frequently sold together. It visualizes the most common product pairs using a bar chart.

```python
sold_together = df[df["Order ID"].duplicated(keep=False)]
sold_together["Grouped"] = sold_together.groupby("Order ID")["Product"].transform(lambda x: "," .join(x))

from itertools import combinations
from collections import Counter

count = Counter()

for row in sold_together["Grouped"]:
    rowlist = row.split(",")
    count.update(Counter(combinations(rowlist, 2)))

count.most_common(20)
```

### Top Selling Products

The code calculates and visualizes the top-selling products based on the quantity ordered and the average price.

```python
sold_most = df.groupby("Product").sum()
sold_most = sold_most[["Quantity Ordered"]]

products = [product for product, df in sold_most.groupby("Product")]
quantity_ordered = sold_most["Quantity Ordered"]

names = plt.bar(products, quantity_ordered)
plt.xticks(products, rotation="vertical")
plt.grid()
plt.ylabel("Sales")
plt.xlabel("Products")
plt.show()

prices = df.groupby("Product").mean()["Price Each"]

fig, ax1 = plt.subplots()
ax2 = ax1.twinx()
ax1.bar(products, quantity_ordered, color="g")
ax2.plot(products, prices, 'b-')
ax1.set_xlabel('Products')
ax1.set_ylabel('Sales', color='g')
ax2.set_ylabel('Price Each', color='b')
ax1.set_xticklabels(products, rotation="vertical")
plt.show()
```

## Conclusion

This Jupyter notebook provides a comprehensive analysis of sales data, including monthly sales trends, top-selling cities, sales by hour, products sold together, and top-selling products. You can use this code as a starting point for your own sales data analysis and visualization projects.
