![datacleaning in Excel][]

# Lifebear Data Processing Script

## Overview

This repository contains Python scripts to process and clean the 3.6M_lifebear.csv dataset using pandas and re libraries. The script includes several steps for reading, cleaning, validating, and splitting large datasets. It performs various data processing tasks such as removing duplicates, validating email addresses, cleaning timestamps, and chunking large CSV files.



## Prerequisites

Make sure you have the following installed:

Python 3.x,
[pandas](https://pypi.org/project/pandas/) (pip install pandas) and
[re](https://pypi.org/project/regex/) (Regular Expressions) (comes with Python standard library)

```python
import pandas as pd
import re
```

## Usage
### 1. Read the Dataset:
The dataset 3.6M_lifebear.csv is read into a pandas DataFrame using memory-efficient parameters.

```python
file_path = '3.6M_lifebear.csv'
lifebear_initial = pd.read_csv(file_path, sep=';', low_memory=True)
```
### 2. Preview the Dataset:
To get an idea of the dataset without opening the entire file, we inspect the first and last 5 lines of the dataset, use the following commands:

```python
# First 5 lines
df_head = pd.read_csv(file_path, nrows=5)
print(df_head)

# Last 5 lines
df_tail = pd.read_csv(file_path, skiprows=lambda x: x < 5, nrows=5)
print(df_tail)
```
### 3. Identify Missing Values:
Identify whether any of the columns in the dataset have missing values and print the number of each for a further understanding of the dataset.
```python
print(lifebear_initial.isnull().sum())
```
### 4. Remove Duplicates
Here, we check to see duplicates that exist in both login_id and mail_address columns and extract those entries to a separate file.
```python
lifebear_duplicates = lifebear_initial[lifebear_initial.duplicated(subset=["login_id", "mail_address"], keep=False)]
lifebear_duplicates.to_csv("/content/lifebear_dup_garbage.csv", index=False)
lifebear_clean = lifebear_initial.drop_duplicates(subset=["login_id", "mail_address"], keep='first')
lifebear_clean.to_csv("/content/lifebear_clean.csv", index=False)
```

### 5. Validate Email Format
A function is used to check if the email addresses follow a valid format. The valid and invalid email datasets are stored in separate files.
```python
  # Expression to allow "-", "_", and "." in the email prefix
    email_regex = r'^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(email_regex, email) is not None

lifebear_clean['valid_email'] = lifebear_clean['mail_address'].apply(is_valid_email)

valid_emails = lifebear_clean[lifebear_clean['valid_email'] == True]
invalid_emails = lifebear_clean[lifebear_clean['valid_email'] == False]
```

### 6. Lowercase Conversion
Both login_id and mail_address columns are converted to lowercase.
```python
lifebear_valid_e = pd.read_csv("/content/lifebear_valid_e.csv")
lifebear_valid_e['login_id'] = lifebear_valid_e['login_id'].str.lower()
lifebear_valid_e['mail_address'] = lifebear_valid_e['mail_address'].str.lower()
lifebear_valid_e.to_csv("/content/lifebear_valid_e_update.csv", index=False)
```
### 7. Clean the "created_at" Column
The time portion is removed from the created_at column, retaining only the date. This was cleaned to standardize date data throughout the dataset to a DD/MM/YYYY format.
```python
lifebear_valid_e_update['created_at'] = pd.to_datetime(lifebear_valid_e_update['created_at']).dt.date
lifebear_valid_e_update.to_csv("/content/lifebear_valid_e_update.csv", index=False)
```
### 8. Strip Whitespace
This step removes any leading or trailing whitespace from all text columns.
```python
lifebear_valid_e_update = lifebear_valid_e_update.apply(lambda x: x.str.strip() if x.dtype == "object" else x)
lifebear_valid_e_update.to_csv("/content/lifebear_stripped.csv", index=False)
```

### 9.Chunk Large Files
Optionally, the final cleaned file could be chunked into smaller CSV files for easier analysis.
```python
def chunk_csv(input_file, chunk_size, output_prefix):
    chunk_num = 0
    for chunk in pd.read_csv(input_file, chunksize=chunk_size):
        output_file = f'{output_prefix}_{chunk_num}.csv'
        chunk.to_csv(output_file, index=False)
        chunk_num += 1
        print(f"Created {output_file}")

chunk_csv('/content/lifebear_stripped.csv', 600000, 'lifebear_cleaned')
```
## Files Generated
* lifebear_dup_garbage.csv: Contains the duplicate rows.
* lifebear_clean.csv: Contains the cleaned dataset after removing duplicates.
* lifebear_valid_e.csv: Contains rows with valid email addresses.
* lifebear_invalid_e.csv: Contains rows with invalid email addresses.
* lifebear_valid_e_update.csv: Contains valid emails after converting columns to lowercase and removing time from created_at.
* lifebear_stripped.csv: Contains the final version with stripped whitespace.
* lifebear_cleaned_*.csv: Contains chunked files for easier ingestion (if required).
## Contributing

Pull requests are welcome. For major changes, please open an issue first
to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

[MIT](https://choosealicense.com/licenses/mit/)
