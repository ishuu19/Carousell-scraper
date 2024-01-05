# Carousell Data Scraper and Cleaning

## Scrapping Carousell Data

### Script Explanation:

#### 1. Setting Up Selenium and Navigating to the URL:
   - The script utilizes the Selenium library to automate web browser interaction.
   - The given URL is the Carousell collectibles search page.

   ```python
   import csv
   from selenium import webdriver
   from selenium.webdriver.common.by import By
   from selenium.webdriver.support.ui import WebDriverWait
   from selenium.webdriver.support import expected_conditions as EC

   url = "https://www.carousell.com.hk/search/collectibles?addRecent=true&canChangeKeyword=true&includeSuggestions=true&searchId=Zuq97c&t-search_query_source=direct_search"

   # Set up the Selenium WebDriver
   driver = webdriver.Chrome()  # You may need to adjust the path if using a different WebDriver

   # Navigate to the URL
   driver.get(url) ```
#### 2. Defining Functions for Data Extraction:
   - The script defines a function `extract_listing_info` to extract information (title, price, date, and condition) from each listing on the page.

   ```python
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

       return {"Title": title, "Price": price, "Date": date, "Condition": condition}```
#### 3. Clicking "Show more results" and Extracting Initial Listings:
   - The script defines a function `click_show_more` to click the "Show more results" button.
   - It waits for the page to load and extracts information from the initial set of listings.

   ```python
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
       all_listings.append(extract_listing_info(listing))```
#### 4. Clicking "Show more results" Until Unavailable and Saving to CSV:
   - The script clicks "Show more results" until the button is no longer available.
   - It saves the scraped data to a CSV file.

   ```python
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

   print(f"Data has been saved to {csv_file_path}")```
## Cleaning Carousell Data

### Script Explanation:

#### 1. Importing Libraries and Reading CSV Data:
   - The script uses Pandas for data cleaning and analysis.
   - It reads the previously scraped Carousell data from the CSV file.

   ```python
   import pandas as pd
   df = pd.read_csv("carousell_data.csv")
   len(df)```
#### 2. Converting 'Date' Column to Datetime:
   - The script defines a function `convert_to_datetime` to convert relative date values (e.g., '16 minutes ago', '1 day ago') to datetime objects.

   ```python
   from datetime import datetime, timedelta

   # Assuming df['Date'] contains strings like '16 minutes ago', '1 day ago', etc.
   # Convert to datetime
   def convert_to_datetime(value):
       if pd.isna(value):
           return pd.NaT
       elif 'minute' in value:
           converted_date = datetime.now() - timedelta(minutes=int(value.split()[0]))
           print(f"{value} -> {converted_date}")
           return converted_date
       elif 'hour' in value:
           converted_date = datetime.now() - timedelta(hours=int(value.split()[0]))
           print(f"{value} -> {converted_date}")
           return converted_date
       elif 'day' in value:
           converted_date = datetime.now() - timedelta(days=int(value.split()[0]))
           print(f"{value} -> {converted_date}")
           return converted_date
       elif 'month' in value:
           converted_date = datetime.now() - timedelta(days=30 * int(value.split()[0]))
           print(f"{value} -> {converted_date}")
           return converted_date
       elif 'year' in value:
           converted_date = datetime.now() - timedelta(days=365 * int(value.split()[0]))
           print(f"{value} -> {converted_date}")
           return converted_date```
#### 3. Applying Date Conversion and Checking Frequency:
   - The script applies the conversion function to the 'Date' column and checks the frequency of each date.

   ```python
   # Apply the conversion function to the 'Date' column
   df['Date'] = df['Date'].apply(convert_to_datetime)

   # Check the frequency
   frequency = df['Date'].value_counts()
   print(frequency)```
#### 4. Visualizing Time Range Frequency:
   - The script uses Plotly Express to create a bar chart visualizing the frequency of time ranges.

   ```python
   import pandas as pd
   import plotly.express as px
   from datetime import datetime

   # Assuming df['Date'] contains datetime values
   # Convert 'Date' column to datetime format
   df['Date'] = pd.to_datetime(df['Date'], errors='coerce')

   # Replace NaT values with a default date (e.g., the maximum date in the DataFrame)
   df['Date'] = df['Date'].fillna(df['Date'].max())

   # Define bins for the time ranges
   bins = [0, 6, 12, 18, 24, 30, 36, 42, 48, float('inf')]
   labels = ['1-6 months', '7-12 months', '13-18 months', '19-24 months', '25-30 months', '31-36 months', '37-42 months',
             '43-48 months', '48+ months']

   # Calculate the time difference in months
   df['Time Difference'] = ((datetime.now() - df['Date']).dt.days / 30).astype(int)

   # Create a new column 'Time Range' to store the categorized values
   df['Time Range'] = pd.cut(df['Time Difference'], bins=bins, labels=labels, right=False)

   # Calculate the frequency of each time range
   frequency = df['Time Range'].value_counts().sort_index()

   # Convert the frequency series to a dictionary
   frequency_dict = frequency.to_dict()

   # Create a DataFrame from the dictionary
   df_plotly = pd.DataFrame(list(frequency_dict.items()), columns=['Time Range', 'Frequency'])

   # Create a bar chart using Plotly Express
   fig = px.bar(df_plotly, x='Time Range', y='Frequency', text='Frequency', title='Time Range Frequency',
                labels={'Frequency': 'Frequency Count', 'Time Range': 'Time Range'})

   # Customize the layout to show values above the bars
   fig.update_traces(textposition='outside', texttemplate='%{text}', textfont_size=12)

   # Show the plot
   fig.show()```
Feel free to customize the script to your specific needs! Happy scraping!
