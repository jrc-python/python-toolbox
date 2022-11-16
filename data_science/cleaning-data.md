# Cleaning Data in Python

- [Cleaning Data in Python](#cleaning-data-in-python)
  - [Data type constraints](#data-type-constraints)
    - [Numeric or categorical](#numeric-or-categorical)
  - [Finding out of range data](#finding-out-of-range-data)
    - [Numeric data range constraints](#numeric-data-range-constraints)
      - [Dropping out of range data](#dropping-out-of-range-data)
      - [Setting custom minimums and maximums](#setting-custom-minimums-and-maximums)
    - [Date range constraints](#date-range-constraints)
      - [Dropping values](#dropping-values)
      - [Hardcoding dates with upper limit](#hardcoding-dates-with-upper-limit)
  - [Uniqueness constraints](#uniqueness-constraints)
    - [Finding duplicates](#finding-duplicates)
    - [How to treat duplicated value](#how-to-treat-duplicated-value)
    - [Grouping and aggregating duplicates](#grouping-and-aggregating-duplicates)
    - [Another example of treating duplicates](#another-example-of-treating-duplicates)
  - [Membership constraints](#membership-constraints)
    - [Finding inconsistent categories](#finding-inconsistent-categories)
    - [Categorical variables](#categorical-variables)
      - [Uppercase or lowercase](#uppercase-or-lowercase)
      - [Trailing spaces](#trailing-spaces)
      - [Collapsing data into categories](#collapsing-data-into-categories)
        - [Using qcut - can lead to errors](#using-qcut---can-lead-to-errors)
        - [Using cut - better, more precise](#using-cut---better-more-precise)
        - [Mapping categories to fewer ones](#mapping-categories-to-fewer-ones)
        - [Another example of remapping categories](#another-example-of-remapping-categories)
    - [Removing titles and taking names](#removing-titles-and-taking-names)
    - [Keeping it descriptive](#keeping-it-descriptive)
    - [Uniform currencies](#uniform-currencies)
    - [Data integrity](#data-integrity)
    - [Completeness](#completeness)
      - [Missingness types](#missingness-types)
    - [Comparing strings](#comparing-strings)
    - [Generating pairs](#generating-pairs)
    - [Linking dataframes](#linking-dataframes)
---
## Data type constraints

```python
# Import CSV file and output header
sales = pd.read_csv('sales.csv')
sales.head(2)

# Get data types of columns
sales.dtypes

# Get DataFrame information
sales.info()

# Revenue column is a string followed by a dollar sign ($)
# If we sum it, it will concatenate the values
sales['Revenue'].sum()
# Example: 1213$567465$8346$

# Remove $ from Revenue column
sales['Revenue'] = sales['Revenue'].str.strip('$')

# Convert Revenue column to integer
sales['Revenue'] = sales['Revenue'].astype('int')

# Verify that Revenue is now an integer
assert ride_sharing['duration_time'].dtype == 'int'

# Convert to categorical
df['marriage_status'] = df['marriage_status'].astype('category')
```

### Numeric or categorical

Categoricals are a pandas data type corresponding to categorical variables in statistics. A categorical variable takes on a limited, and usually fixed, number of possible values (categories; levels in R). Examples are gender, social class, blood type, country affiliation, observation time or rating.

```python
# Before converting to categorical
df['marriage_status'].describe()
```
| DataFrame describe | |
| --- | --- |
| mean | 1.4 |
| std | 0.20 |
| min | 0.00 |
| 50% | 1.8 |
```python
# Convert to categorical
df['marriage_status'] = df['marriage_status'].astype('category')

# After converting to categorical
df.describe()
```
| DataFrame describe | |
| --- | --- |
| count | 241 |
| unique | 4 |
| top | 1 |
| freq | 120 |

## Finding out of range data

### Numeric data range constraints

```python
# Output movies with rating > 5
movies[movies['avg_rating'] > 5]
```

#### Dropping out of range data

```python
# Drop values using filtering
movies = movies[movies['avg_rating'] <= 5]

# Drop values using drop
# Inplace: if False, returns a copy. Otherwise, do operation inplace and return None.
movies.drop(movies[movies['avg_rating'] <= 5].index, inplace = True)

# Assert results
assert movies['avg_rating'].max() <= 5
```

#### Setting custom minimums and maximums

```python
# Convert avg_rating > 5 to 5
movies.loc[movies['avg_rating'] > 5, 'avg_rating'] = 5

# Assert results
assert movies['avg_rating'].max() <= 5
```

### Date range constraints

```python
import pandas as pd
import datetime as dt

# Output data types
user_signups.dtypes
```

| dtypes | |
| ---- | ---- |
| subscription_date | object |
| user_name | object |
| Country | object |

```python
# Convert to date
user_signups['subscription_date'] = pd.to_datetime(user_signups['subscription_date']).dt.date
```

#### Dropping values

```python
today_date = dt.date.today()

# Drop values using filtering
user_signups = user_signups[user_signups['subscription_date'] <= today_date]

# Drop values using drop
user_signups.drop(user_signups[user_signups['subscription_date'] > today_date].index, inplace = True)
```

#### Hardcoding dates with upper limit

```python
today_date = dt.date.today()

# Hardcoding future dates to today
user_signups.loc[user_signups['subscription_date'] > today_date, 'subscription_date'] = today_date

# Assert results
assert user_signups['subscription_date'].max().date() <= today_date
```

## Uniqueness constraints

### Finding duplicates

```python
# Get duplicates across all columns
duplicates = height_weight.duplicated()
height_weight[duplicates]
```

Important parameters for the duplicated method:
* subset: List of column names to check for duplication
* keep: Whether to keep **first** (`'first'`), **last** (`'last'`) or **all** (`False`) duplicate values.

```python
# Column names to check for duplication
column_names = ['first_name', 'last_name', 'address']

# Get duplicates across selected columns
duplicates = height_weight.duplicated(subset = column_names, keep = False)

# Output duplicate values
height_weight[duplicates].sort_values(by = 'first_name')
```

### How to treat duplicated value

Two important parameters for the .drop_duplicates method:
* subset: List of column names to check for duplication
* keep: Whether to keep **first** (`'first'`), **last** (`'last'`) or **all** (`False`) duplicate values.
* inplace: If `True`, drops duplicated rows directly inside DataFrame without creating new object

```python
# Drop duplicates
height_weight.drop_duplicates(inplace = True)
```

### Grouping and aggregating duplicates

```python
# Group by columns and produce statistical summaries
column_names = ['first_name', 'last_name', 'address']
summaries = {'height': max, 'weight': 'mean'}

height_weight = height_weight.groupby(by = column_names).agg(summaries).reset_index()

# Make sure aggregation is done
duplicates = height_weight.duplicated(subset = column_names, keep = False)
height_weight[duplicates].sort_values(by = 'first_name')
```

### Another example of treating duplicates

```python
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

## Membership constraints

### Finding inconsistent categories

`study_data`

| Name | birthday | blood_type |
| --- | --- | --- |
| John | 2017-01-12 | B- |
| Mary | 2018-09-05 | A+ |
| ... | ... | ... |
| Chloe | 2015-10-26 | <span style="color:red">Z-</style> |

`categories`

| blood_type | |
| --- | --- |
| 1 | O- |
| 2 | O+ |
| 3 | A- |
| 4 | A+ |
| 5 | B- |
| 6 | B+ |
| 7 | AB- |
| 8 | AB+ |

```python
# Find inconsistent data
inconsistent_categories = set(study_data['blood_type']).difference(categories['blood_type'])

# Get rows with inconsistent categories
inconsistent_rows = study_data['blood_type'].isin(inconsistent_categories)

# Drop inconsistent categories and get consistent data only
consistent_data = study_data[~inconsistent_rows]
```

### Categorical variables

| Type of data | Example Values | Numeric representation |
| ----------- | ----------- | ----------- |
| Marriage Status | `unmarried`, `married`| 0, 1 |
| Household Income Category | `0-20k`, `20-40k`, ... | 0, 1, ... |
| Loan Status | `default`, `payed`, `no_loan` | 0, 1, 2|

#### Uppercase or lowercase

```python
# Get marriage status column in Series
marriage_status = demographics['marriage_status']
marriage_status.value_counts()

# Get value counts on DataFrame
demographics.groupby('marriage_status').count()
```

| marriage_status | household_income | gender |
| --- | --- | --- |
| unmarried | 352 | 352 |
| married | 265 | 265 |
| MARRIED | 204 | 204 |
| UNMARRIED | 176 | 176 |

```python
# Capitalize
demographics['marriage_status'] = demographics['marriage_status'].str.upper()

# Lowercase
demographics['marriage_status'] = demographics['marriage_status'].str.lower()
```

#### Trailing spaces

`married `, ` married`, `unmarried`, ` unmarried`

```python
demographics['marriage_status'] = demographics['marriage_status'].str.strip()
```

#### Collapsing data into categories

##### Using qcut - can lead to errors

```python
import pandas as pd

group_names = ['0-200k', '200k-500k', '500k+']

demographics['income_group'] = pd.qcut(demographics['household_income'], q = 3, labels = group_names)

# Print income_group column
print(demographics['income_group', 'household_income'])
```

##### Using cut - better, more precise

```python
# Create categoriy ranges and names
ranges = [0, 200000, 500000, np.inf]
group_names = ['0-200k', '200k-500k', '500k+']

# Create income_group column
demographics['income_group'] = pd.cut(demographics['household_income'], bins=ranges, labels=group_names)

# Print income_group column
print(demographics['income_group', 'household_income'])
```

##### Mapping categories to fewer ones

```python
mapping = {
    'Microsoft': 'DesktopOS',
    'MacOS': 'DesktopOS',
    'Linux': 'DesktopOS',
    'Android': 'MobileOS',
    'iOS': 'MobileOS',
}

# Remap categories
devices['operating_system'] = devices['operating_system'].replace(mapping)

# Print operating_system column
devices['operating_system'].unique()
```

##### Another example of remapping categories

```python
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

```python
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

```python
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

```python
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

```python
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

```python
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

```python
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

categories = ['Corinthians', 'Palmeiras']

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

### Generating pairs

```python
import recordlinkage

# Create indexing object
indexer = recordlinkage.Index()

# Generate pairs blocked on state
indexer.block('state')
pairs = indexer.index(census_A, census_B)

# Create a Compare object
compare_cl = recordlinkage.Compare()

# Find exact matches for pairs of date_of_birth and state
compare_cl.exact('date_of_birth', 'date_of_birth', label='date_of_birth')
compare_cl.exact('state', 'state', label='state')

# Find similar matches for pairs of surname and address_1 using string similarity
compare_cl.string('surname', 'surname', threshold=0.85, label='surname')
compare_cl.string('address_1', 'address_1', threshold=0.85, label='address_1')

# Find matches
potential_matches = compare_cl.compute(pairs, census_A, census_B)
```

### Linking dataframes

```python
# Isolate matches with matching values for 3 or more columns
matches = potential_matches[potential_matches.sum(axis = 1) >= 3]

# Get indices for matching census_B rows only
duplicate_rows = matches.index.get_level_values(1)

# Find duplicates in census_B
census_B_duplicates = census_B[census_B.index.isin(duplicate_rows)]

# Find new rows in census_B
census_B_new = census_B[~census_B.index.isin(duplicate_rows)]

# Link the DataFrames
full_census = census_A.append(census_B_new)
```
