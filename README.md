# Carousell Data Scraper and Cleaning
This script allows you to scrape data from Carousell, a popular online marketplace, and perform data cleaning on the scraped data. The scraping is done using the Selenium library, which automates web browser interaction. The cleaned data is then analyzed and visualized using the Pandas and Plotly libraries.

# Scrapping Carousell Data
### Script Explanation:
#### Setting Up Selenium and Navigating to the URL

The script starts by importing the necessary libraries and setting up Selenium with the Chrome WebDriver.
The given URL is the Carousell collectibles search page.
python
Copy
import csv
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

url = "https://www.carousell.com.hk/search/collectibles?addRecent=true&canChangeKeyword=true&includeSuggestions=true&searchId=Zuq97c&t-search_query_source=direct_search"

# Set up the Selenium WebDriver
driver = webdriver.Chrome()  # You may need to adjust the path if using a different WebDriver

# Navigate to the URL
driver.get(url)
```

 Defining Functions for Data Extraction

The script defines a function extract_listing_info to extract information (title, price, date, and condition) from each listing on the page.
python

def extract_listing_info(listing):
    # Extracting title, price, date, and condition information
    title_element = listing.find_element(By.CSS_SELECTOR, "p.D_oW.D_nU.D_oX.D_pb.D_pf.D_pi.D_pk.D_pg.D_po")
    price_element = listing.find_element(By.CSS_SELECTOR, "p.D_oW.D_nU.D_oX.D_pb.D_pe.D_pi.D_pl.D_pn")
    date_element = listing.find_element(By.CSS_SELECTOR, "p.D_oW.D_nS.D_oX.D_pb.D_pe.D_pi.D_pk.D_qw.D_pp")
    condition_element = listing.find_element(By.CSS_SELECTOR, "p.D_oW.D_nS.D_oX.D_pb.D_pe.D_pi.D_pk.D_po")

    title = title_element.text
    price = price_element.get_attribute("title")
    date = date_element.text
    condition = condition_element.text

    return {"Title": title, "Price": price, "Date": date, "Condition": condition}
```

Clicking "Show more results" and Extracting Initial Listings

The script defines a function click_show_more to click the "Show more results" button.
It waits for the page to load and extracts information from the initial set of listings.
python
Copy
# Function to click "Show more results" button
def click_show_more():
    try:
        show_more_button = driver.find_element(By.CLASS_NAME, "D_oo.D_oJ.D_oA.D_ow.D_oN.D_IL")
        show_more_button.click()
        return True
    except:
        return False

# Wait for the page to load
wait = WebDriverWait(driver, 10)
wait.until(EC.presence_of_element_located((By.CLASS_NAME, "D_qe")))

# Extract information from the initial set of listings
all_listings = []
listings = driver.find_elements(By.CLASS_NAME, "D_qe")
for listing in listings:
    all_listings.append(extract_listing_info(listing))
```

Clicking "Show more results" Until Unavailable and Saving to CSV

The script clicks the "Show more results" button until it's no longer available.
It saves the scraped data to a CSV file.
python
Copy
# Click "Show more results" button until it's no longer available
while click_show_more():
    wait.until(EC.invisibility_of_element_located((By.CLASS_NAME, "D_oo.D_oJ.D_oA.D_ow.D_oN.D_IL")))
    listings = driver.find_elements(By.CLASS_NAME, "D_qe")
    for listing in listings:
        all_listings.append(extract_listing_info(listing))

# Save the data to a CSV file
csv_file_path = "carousell_data.csv"
csv_columns = ["Title", "Price", "Date", "Condition"]

with open(csv_file_path, "w", newline="", encoding="utf-8") as csv_file:
    writer = csv.DictWriter(csv_file, fieldnames=csv_columns)
    writer.writeheader()
    for listing in all_listings:
        writer.writerow(listing)

# Close the browser
driver.quit()

print(f"Data has been saved to`carousell_data.csv`.")
Data Cleaning
Script Explanation:
Loading the Scraped Data

The script starts by importing the necessary libraries and loading the scraped data from the CSV file into a Pandas DataFrame.
python
Copy
import pandas as pd

# Load the scraped data from the CSV file
df = pd.read_csv("carousell_data.csv")
Cleaning the Data

The script performs data cleaning operations on the loaded DataFrame, such as removing duplicates, handling missing values, and converting data types if necessary.
python
Copy
# Remove duplicate rows
df.drop_duplicates(inplace=True)

# Handle missing values
df.dropna(subset=["Title", "Price", "Date", "Condition"], inplace=True)

# Convert data types if necessary
df["Price"] = df["Price"].str.replace("$", "").astype(float)
df["Date"] = pd.to_datetime(df["Date"])
```

Analyzing and Visualizing the Cleaned Data

The script provides some examples of analyzing and visualizing the cleaned data using the Pandas and Plotly libraries.
python
Copy
# Example: Calculate average price by condition
avg_price_by_condition = df.groupby("Condition")["Price"].mean()
print(avg_price_by_condition)

# Example: Create a bar chart of average price by condition
import plotly.express as px

fig = px.bar(df, x="Condition", y="Price", barmode="group")
fig.show()
```

Saving the Cleaned Data

The script saves the cleaned data to a new CSV file.
python
Copy
# Save the cleaned data to a new CSV file
cleaned_csv_file_path = "cleaned_carousell_data.csv"
df.to_csv(cleaned_csv_file_path, index=False)
```
Usage
Install the required libraries:

shell
Copy
pip install selenium pandas plotly
```

Run the scraping script:

shell
Copy
python scrape_carousell.py
```

The script will scrape data from Carousell and save it to a CSV file named `carousell_data.csv`.

Run the data cleaning script:

shell
Copy
python clean_data.py
```

The script will load the scraped data, perform data cleaning operations, analyze and visualize the cleaned data, and save it to a new CSV file named `cleaned_carousell_data.csv`.
Make sure to adjust the file paths and WebDriver setup according to your specific environment. Additionally, remember to handle potential errors and exceptions based on your requirements.
