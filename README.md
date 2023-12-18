# Analysis of Sutter Health's Hospital Data

This repository contains the python code used in the analysis of Sutter Health's hospital data, focusing on the commercial and government rates. The full article related to this analysis can be found [here](https://chienho.substack.com/publish/post/139728370).

## Python Code Used in Analysis

### Reading in the Data

```python
import pandas as pd
import chardet

# Define local variable
file_path = 'data/941156621-1811946734_SUTTER-MEDICAL-CENTER-SACRAMENTO_standardcharges.csv'

# Detect encoding of CSV file
with open(file_path, 'rb') as file:
    encoding = chardet.detect(file.read())['encoding']

# Read the file with the detected encoding
df_loaded = pd.read_csv(file_path, skiprows=1, encoding=encoding, low_memory=False)
```

### Parsing Columns
```python
def find_columns_containing(df, search_string):
    """
    Finds and returns a list of column names in the DataFrame that contain the given 'search_string'.
    """
    return [column for column in df.columns if search_string in column]

columns_with_hmo_and_ppo = find_columns_containing(df_filtered, 'HMO/PPO')
columns_with_medicare = find_columns_containing(df_filtered, 'Medicare')

# Print the lists
print(f"Columns with HMO/PPO: {columns_with_hmo_and_ppo}")
print(f"Columns with Medicare: {columns_with_medicare}")
```

### Converting Data Types
```python
# Function to convert price columns from string to numeric
def convert_price_column(data_frame, column_name):
    data_frame.loc[:, column_name] = pd.to_numeric(data_frame[column_name].replace('[\$,]', '', regex=True), errors='coerce')
    return data_frame

# Combine all column lists into one
all_columns = columns_with_hmo_and_ppo + columns_with_medicare  + ['United HMO / POS']

# Loop through the combined list and convert each column
for column in all_columns:
    df_filtered = convert_price_column(df_filtered, column)

```

### Averaging Rates
```python
df_filtered.loc[:, 'commercial_rate'] = df_filtered[columns_with_hmo_and_ppo].mean(axis=1)
df_filtered.loc[:,'medicare_rate'] = df_filtered[columns_with_medicare].mean(axis=1)
```

### Breakdown of Commercial Insurance Rates
```python
import matplotlib.pyplot as plt
from matplotlib.ticker import FuncFormatter

# Define a function to format y-axis labels as currency (e.g., $1,000)
def currency_format(x, position):
    # x: value, position: unused here but required by FuncFormatter
    return "${:,.0f}".format(x)

# Calculate the average rates from a DataFrame named df_filtered
# .mean() calculates the mean of the specified column
average_commercial_rate = df_filtered['commercial_rate'].mean()
average_aetna_rate = df_filtered['Aetna HMO/PPO'].mean()
average_anthem_rate = df_filtered['Anthem HMO/PPO'].mean()
average_cigna_rate = df_filtered['Cigna HMO/PPO'].mean()
average_united_rate = df_filtered['United HMO / POS'].mean()

# Prepare data for plotting
# 'rates' holds the average rates for different insurers
# 'rate_labels' holds the names of the insurers
rates = [average_aetna_rate, average_anthem_rate, average_cigna_rate, average_united_rate]
rate_labels = ['Aetna', 'Anthem', 'Cigna', 'United']

# Create a figure for plotting with a specific size (width: 10, height: 6)
plt.figure(figsize=(10, 6))

# Plot a bar chart
# 'rate_labels' are on the x-axis and 'rates' are the heights of the bars
# All bars are colored blue
plt.bar(rate_labels, rates, color='blue')

# Add a title to the chart and labels to the y-axis
plt.title('Breakdown of Commercial Insurance Rates')
plt.ylabel('Avg Cost of Cataract Surgery')

# Add grid lines on the y-axis for better readability, with a dashed style
plt.grid(axis='y', linestyle='--', alpha=0.7)

# Format the y-axis labels as currency
plt.gca().yaxis.set_major_formatter(FuncFormatter(currency_format))

# Add a horizontal line across the chart representing the average commercial rate
# The line is gray and dotted
plt.axhline(y=average_commercial_rate, color='gray', linestyle='--')

# Add a label to the right side of the chart for the average commercial rate
# The label is positioned at the rightmost bar and aligned at the bottom
plt.text(len(rate_labels) - 1, average_commercial_rate, 'Avg Commercial Rate', color='gray', va='bottom', ha='right')

# Show the plot
plt.show()
```

### Comparison of Commercial and Medicare Rates
```python
average_commercial_rate = df_filtered['commercial_rate'].mean()
average_medicare_rate = df_filtered['medicare_rate'].mean()

# Values to plot
rates = [average_commercial_rate, average_medicare_rate]
rate_labels = ['Avg Commercial Rate', 'Avg Medicare Rate']

# Creating the bar chart
plt.bar(rate_labels, rates, color=['blue', 'green'])

# Adding title and labels
plt.title('Comparison of Commercial and Medicare Rates')
plt.ylabel('Avg Cost of Cataract Surgery')

# Add grid lines for better readability
plt.grid(axis='y', linestyle='--', alpha=0.7)

# Apply the currency_format function to the y-axis labels
plt.gca().yaxis.set_major_formatter(FuncFormatter(currency_format))

# Show the plot
plt.show()
```
