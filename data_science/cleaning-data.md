# Cleaning Data in Python

## Common data problems

### Type of data
```
# Print the information of ride_sharing
print(ride_sharing.info())

# Print summary statistics of user_type column
print(ride_sharing['user_type'].describe())
```

### Summing strings and concatenating numbers
```
# Strip duration of minutes
ride_sharing['duration_trim'] = ride_sharing['duration'].str.strip('minutes')

# Convert duration to integer
ride_sharing['duration_time'] = ride_sharing['duration_trim'].astype('int')

# Write an assert statement making sure of conversion
assert ride_sharing['duration_time'].dtype == 'int'

# Print formed columns and calculate average ride duration 
print(ride_sharing[['duration','duration_trim','duration_time']])
print(ride_sharing['duration_time'].mean())
```

### Data constraints
```
# Convert tire_sizes to integer
ride_sharing['tire_sizes'] = ride_sharing['tire_sizes'].astype('int')

# Set all values above 27 to 27
ride_sharing.loc[ride_sharing['tire_sizes'] > 27, 'tire_sizes'] = 27

# Reconvert tire_sizes back to categorical
ride_sharing['tire_sizes'] = ride_sharing['tire_sizes'].astype('category')

# Print tire size description
print(ride_sharing['tire_sizes'].dtype)
```

### Invalid dates
```
# Convert ride_date to date
ride_sharing['ride_dt'] = pd.to_datetime(ride_sharing['ride_date']).dt.date

# Save today's date
today = dt.date.today()

# Set all in the future to today's date
ride_sharing.loc[ride_sharing['ride_dt'] > today, 'ride_dt'] = today

# Print maximum of ride_dt column
print(ride_sharing['ride_dt'].max())
```

### Finding duplicates
```
# Find duplicates
duplicates = ride_sharing.duplicated('ride_id', keep = False)

# Sort your duplicated rides
duplicated_rides = ride_sharing[duplicates].sort_values('ride_id')

# Print relevant columns of duplicated_rides
print(duplicated_rides[['ride_id','duration','user_birth_year']])
```

### Treating duplicates
```
# Drop complete duplicates from ride_sharing
ride_dup = ride_sharing.drop_duplicates()

# Create statistics dictionary for aggregation function
statistics = {'user_birth_year': 'min', 'duration': 'mean'}

# Group by ride_id and compute new statistics
ride_unique = ride_dup.groupby('ride_id').agg(statistics).reset_index()

# Find duplicated values again
duplicates = ride_unique.duplicated(subset = 'ride_id', keep = False)
duplicated_rides = ride_unique[duplicates == True]

# Assert duplicates are processed
assert duplicated_rides.shape[0] == 0
```

### Finding consistency
```
# Print categories DataFrame
print(categories)

# Print unique values of survey columns in airlines
print('Cleanliness: ', airlines['cleanliness'].unique(), "\n")
print('Safety: ', airlines['safety'].unique(), "\n")
print('Satisfaction: ', airlines['satisfaction'].unique(), "\n")
```

### Inconsistent categories
```
# Print unique values of both columns
print(airlines['dest_region'].unique())
print(airlines['dest_size'].unique())
```

### Remapping categories
```
# Create ranges for categories
label_ranges = [0, 60, 180, np.inf]
label_names = ['short', 'medium', 'long']

# Create wait_type column
airlines['wait_type'] = pd.cut(airlines['wait_min'], bins = label_ranges, labels = label_names)

# Create mappings and replace
mappings = {
    'Monday':'weekday',
    'Tuesday':'weekday',
    'Wednesday': 'weekday',
    'Thursday': 'weekday',
    'Friday': 'weekday',
    'Saturday': 'weekend',
    'Sunday': 'weekend'
}

airlines['day_week'] = airlines['day'].replace(mappings)
```

### Removing titles and taking names
```
# Replace "Dr." with empty string ""
airlines['full_name'] = airlines['full_name'].str.replace("Dr.","")

# Replace "Mr." with empty string ""
airlines['full_name'] = airlines['full_name'].str.replace("Mr.","")

# Replace "Miss" with empty string ""
airlines['full_name'] = airlines['full_name'].str.replace("Miss","")

# Replace "Ms." with empty string ""
airlines['full_name'] = airlines['full_name'].str.replace("Ms.","")

# Assert that full_name has no honorifics
assert airlines['full_name'].str.contains('Ms.|Mr.|Miss|Dr.').any() == False
```

### Keeping it descriptive
```
# Store length of each row in survey_response column
resp_length = airlines['survey_response'].str.len()

# Find rows in airlines where resp_length > 40
airlines_survey = airlines[resp_length > 40]

# Assert minimum survey_response length is > 40
assert airlines_survey['survey_response'].str.len().min() > 40

# Print new survey_response column
print(airlines_survey['survey_response'])
```

### Uniform currencies
```
# Find values of acct_cur that are equal to 'euro'
acct_eu = banking['acct_cur'] == 'euro'

# Convert acct_amount where it is in euro to dollars
banking.loc[acct_eu, 'acct_amount'] = banking.loc[acct_eu, 'acct_amount'] * 1.1

# Unify acct_cur column by changing 'euro' values to 'dollar'
banking.loc[acct_eu, 'acct_cur'] = 'dollar'

# Assert that only dollar currency remains
assert banking['acct_cur'].unique() == 'dollar'
```

### Data integrity
```
# Store fund columns to sum against
fund_columns = ['fund_A', 'fund_B', 'fund_C', 'fund_D']

# Find rows where fund_columns row sum == inv_amount
inv_equ = banking[fund_columns].sum(axis = 1) == banking['inv_amount']

# Store consistent and inconsistent data
consistent_inv = banking[inv_equ]
inconsistent_inv = banking[~inv_equ]

# Store consistent and inconsistent data
print("Number of inconsistent investments: ", inconsistent_inv.shape[0])
```

### Completeness

```
import missingno as msno
import matplotlib.pyplot as plt

# Print number of missing values in dataframe
print(dataframe.isna().sum())

# Visualize missingness
msno.matrix(dataframe)
plt.show()

# Isolate missing and complete values aside
missing = dataframe[dataframe['column'].isna()]
complete = dataframe[~dataframe['column'].isna()]

# Describe missing and complete sets
missing.describe()
complete.describe()

# Visualize sorted missingness to find correlation between columns
sorted_data = dataframe.sort_values(by = 'column')
msno.matrix(sorted_data)
plt.show()

# Drop missing values
values_dropped = dataframe.dropna(subset = ['column'])
values_dropped.head()

# Replace missing values
value_mean = dataframe['column'].mean()
data_imputed = dataframe.fillna({'column': value_mean})
data_imputed.head()
```

#### Missingness types
- Missing completely at random (MCAR)
    - No systematic relationship between missing data and other values
    - Possibly caused by data entry errors when inputting data
- Missing at random (MAR)
    - Systematic relationship between missing data and other **observed** values
    - Example: missing ozone data for high temperatures
- Missing not at random (MNAR)
    - Systematic relationship between missing data and other **unobserved** values
    - Example: missing temperature values for high temperatures


### Comparing strings
**Edit distance**: a string metric, i.e. a way of quantifying how dissimilar two strings (e.g., words) are to one another. Steps required to turn a string into another.

```
from thefuzz import fuzz, process

# Result ranges from 0 (no similarity) to 100 (exactly equal)
fuzz.WRation('String1', 'String2')

string = 'Corinthians Timão vs São Paulo Tricolor'
choices = pd.Series([
    'Timão vs Tricolor',
    'Tricolor vs Timão',
    'Verdão vs Corinthians',
    'Galo vs Peixe'
])

process.extract(srting, choices, limit = 2)

print(results['team'].unique())

# Many variations, can't fix with .replace()
# Example: Corinthians, Corintians, Corinthian, Corintia, Corinthia

categories
# 0 Corinthians
# 1 Palmeiras

# For each correct category
for team in categories['team']:
    # Find potential matches in teams with typoes
    matches = process.extract(team, results['team'], limit = results.shape[0])
    # For each potential match
    for potential_match in matches:
        # If high similarity score
        if potential_match[1] >= 80:
            results.loc[results['team'] == potential_match[0], 'team'] = team
```
