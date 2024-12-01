# India_Mart_WebScraping-
This script automates web scraping using Selenium to extract product and seller information from IndiaMART. The scraped data is then structured into a pandas DataFrame and saved as Excel files. Additionally, it can be shared on LinkedIn and GitHub for professional purposes using HTTP requests and API calls.

## Here is your code broken into pieces with their importance and explanation:

---

### **1. Import Libraries**
```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException, NoSuchElementException
import pandas as pd
import time
```

**Purpose:**  
These imports provide all the necessary tools for:  
- **WebDriver setup:** Initialize and manage Chrome browser automation (`webdriver`, `Service`, `Options`).  
- **Element interaction:** Locate and interact with page elements (`By`, `WebDriverWait`, `expected_conditions`).  
- **Exception handling:** Manage timeout and missing element errors (`TimeoutException`, `NoSuchElementException`).  
- **Data storage and delays:** Store scraped data in a structured format using pandas and introduce delays using `time`.

---

### **2. Define the Scraping Function**
```python
def scrape_and_save_data(url, driver_path, step_size=2):
```

**Purpose:**  
This function automates the scraping process and saves the data.  
- `url`: The webpage to scrape.  
- `driver_path`: Path to ChromeDriver executable for browser control.  
- `step_size`: Number of records to skip while saving subsets of data.

---

### **3. WebDriver Setup**
```python
chrome_options = Options()
service = Service(driver_path)
driver = webdriver.Chrome(service=service, options=chrome_options)
driver.get(url)
```

**Purpose:**  
Sets up the Chrome browser using Selenium.  
- `chrome_options`: Custom browser options (e.g., headless mode, disable extensions).  
- `driver.get(url)`: Navigates to the specified URL.  

---

### **4. Wait for Initial Elements to Load**
```python
try:
    WebDriverWait(driver, 20).until(EC.presence_of_all_elements_located((By.CLASS_NAME, "cardlinks")))
    WebDriverWait(driver, 20).until(EC.presence_of_all_elements_located((By.CSS_SELECTOR, "span.elps.elps1")))
except TimeoutException:
    print("Error: Elements did not load in time.")
    driver.quit()
    return
```

**Purpose:**  
Ensures essential elements (like product names and addresses) are loaded before scraping begins.  
- Uses `WebDriverWait` with a timeout of 20 seconds to wait for specific elements (`cardlinks` and `span.elps.elps1`).  
- Handles `TimeoutException` to exit gracefully if elements don't load.

---

### **5. Initialize Data Storage**
```python
product_names = []
product_links = []
seller_names = []
seller_addresses = []
```

**Purpose:**  
Creates empty lists to store scraped data for products, links, sellers, and addresses.

---

### **6. Define the Scraping Logic**
```python
def scrape_data():
    try:
        products = driver.find_elements(By.CLASS_NAME, "cardlinks")
        addresses = driver.find_elements(By.CSS_SELECTOR, "span.elps.elps1")

        for i, product in enumerate(products):
            product_name = product.text.strip()
            product_link = product.get_attribute('href')

            try:
                seller_name = product.find_element(By.XPATH, ".//following-sibling::a").text.strip()
            except NoSuchElementException:
                seller_name = "N/A"

            seller_address = addresses[i].text.strip() if i < len(addresses) else 'N/A'

            product_names.append(product_name)
            product_links.append(product_link)
            seller_names.append(seller_name)
            seller_addresses.append(seller_address)
    except Exception as e:
        print(f"Error during scraping: {e}")
```

**Purpose:**  
Extracts product and seller details from the webpage.  
- Iterates through products and their related elements (`cardlinks` and `elps`).  
- Handles missing seller names using `NoSuchElementException`.  
- Appends the data to the respective lists.

---

### **7. Pause for Manual Interaction**
```python
print("Please log in and click the 'Show More' button manually. Press Enter to continue...")
input()
```

**Purpose:**  
Allows manual login and interaction, if required, before automated scraping continues.

---

### **8. Define Scrolling and Loading Logic**
```python
def scroll_and_load():
    last_height = driver.execute_script("return document.body.scrollHeight")
    while True:
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        time.sleep(3)

        scrape_data()

        new_height = driver.execute_script("return document.body.scrollHeight")
        if new_height == last_height:
            print("No more data to load.")
            break
        last_height = new_height
```

**Purpose:**  
Handles dynamic content loading by scrolling the page.  
- Scrolls to the bottom and waits for new data to load.  
- Breaks the loop when no additional data loads (`last_height == new_height`).

---

### **9. Save Data**
```python
df = pd.DataFrame({
    'Product Name': product_names,
    'Product Link': product_links,
    'Seller Name': seller_names,
    'Seller Address': seller_addresses
})
df.to_excel('korean_scraped_data.xlsx', index=False)
```

**Purpose:**  
Converts the scraped data into a structured pandas DataFrame and saves it as an Excel file.

---

### **10. Save Subset of Data**
```python
indices = list(range(0, len(df), step_size))
specific_records = df.iloc[indices].reset_index(drop=True)
nth_records = df.iloc[step_size-1::step_size].reset_index(drop=True)

specific_records.to_excel("korean_products.xlsx", index=False)
nth_records.to_excel("korean_seller.xlsx", index=False)
```

**Purpose:**  
Saves specific records (every Nth record) to separate Excel files for detailed analysis.

---

### **11. Close the Browser**
```python
driver.quit()
```

**Purpose:**  
Closes the Chrome browser instance to release resources.

---

### **12. Function Invocation**
```python
scrape_and_save_data(
    url=url_korean,  # Replace with the actual URL
    driver_path=r'C:\Users\data_architect\Downloads\chromedriver-win64 (1)\chromedriver-win64\chromedriver.exe',
    step_size=2
)
```

**Purpose:**  
Executes the scraping function with the specified URL, ChromeDriver path, and step size.  
- `url_korean`: Replace with the actual URL to scrape.  
- `driver_path`: Path to ChromeDriver executable.  
- `step_size`: Controls the interval for saving subsets of records.
