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

### Breakdown of Commercial
```python
import matplotlib.pyplot as plt
from matplotlib.ticker import FuncFormatter

# Define a function to format y-axis labels with commas and a dollar sign
def currency_format(x, pos):
    return "${:,.0f}".format(x)

# Calculate the average rates
average_commercial_rate = df_filtered['commercial_rate'].mean()
average_aetna_rate = df_filtered['Aetna HMO/PPO'].mean()
average_anthem_rate = df_filtered['Anthem HMO/PPO'].mean()
average_cigna_rate = df_filtered['Cigna HMO/PPO'].mean()
average_united_rate = df_filtered['United HMO / POS'].mean()

# Values to plot
rates = [average_commercial_rate, average_aetna_rate, average_anthem_rate, average_cigna_rate, average_united_rate]
rate_labels = ['Commercial', 'Aetna', 'Anthem', 'Cigna', 'United']

# Colors for each bar
colors = ['blue', 'blue', 'blue', 'blue', 'blue']

# Creating the bar chart
plt.figure(figsize=(10, 6))  # Set the figure size
plt.bar(rate_labels, rates, color=colors)

# Adding title and labels
plt.title('Breakdown of the Commercial Insurance Rates')
plt.ylabel('Avg Cost of Cataract Surgery')

# Add grid lines for better readability
plt.grid(axis='y', linestyle='--', alpha=0.7)

# Apply the currency_format function to the y-axis labels
plt.gca().yaxis.set_major_formatter(FuncFormatter(currency_format))

# Show the plot
plt.show()
```

### Commercial vs Medicare
```python
# Calculate the average (or any other representative value) of the rates
average_commercial_rate = df_filtered['commercial_rate'].mean()
average_medicare_rate = df_filtered['medicare_rate'].mean()

# Values to plot
rates = [average_commercial_rate, average_medicare_rate]
rate_labels = ['Commercial Rate', 'Medicare Rate']

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
