
# Scraping eBay: Step-by-Step Guide to Extracting Product Data with Python

<article>

Even though [Amazon](https://blog.datahut.co/challenges-that-make-amazon-data-scraping-so-painful/) is the leader in e-commerce marketplaces, eBay still holds a significant share in the online retail industry. For businesses selling online, monitoring prices on eBay can provide a competitive edge.

Stop wasting time on proxies and CAPTCHAs! ScraperAPI's simple API handles millions of web scraping requests, so you can focus on the data. Get structured data from Amazon, Google, Walmart, and more. Start your free trial today! 👉 [ScraperAPI](https://www.scraperapi.com/?fp_ref=coupons)

For data scientists, regularly scraping product data from eBay at scale is a complex challenge. This article walks you through how to scrape eBay using Python to track the prices of mobile phones effectively.

Imagine a scenario where you want to monitor the pricing trends of a mobile phone on eBay. You might also want to compare it with other phones under consideration and visualize the price ranges available for better insights.

In this guide, we’ll demonstrate how to scrape eBay for phone prices and analyze the data to make smarter decisions.

---

## How to Scrape eBay Product Data

This section outlines the step-by-step process for extracting product and price data from eBay.

### 1. Identifying Target Information

The first step in web scraping is identifying the target web page where the information resides. For eBay product listings, simply navigate to eBay, search for your desired product, and copy the URL of the results page. For example:

`https://www.ebay.com/sch/i.html?_from=R40&_nkw=galaxy+note+8&_sacat=0&_pgn=1`

Key URL parameters to note:
- **`nkw` (new keyword)**: Represents the product you are searching for.
- **`pgn` (page number)**: Indicates the page number of search results.

Changing `nkw` to "iPhone X" will return results for iPhone X, and altering `pgn` to 2 will display the second page of results.

---

### 2. Analyzing HTML Structure

Once you have your target URL, you need to understand the HTML layout of the page. This is essential for locating the desired data. Open the developer tools in your browser (`CTRL + SHIFT + I`) and inspect the elements of the page.

On eBay product listings, items are usually listed as HTML `<li>` elements with a common class name, such as `s-item`. Specific data points like product name and price can be identified by their respective classes:
- **Product name**: `s-item__title`
- **Product price**: `s-item__price`

---

### 3. Structuring the Data

After extracting the desired information, the next step is organizing it into a structured format. For this guide, we’ll create a table with two columns: product names and their corresponding prices.

---

### 4. Visualizing the Results (Optional)

Visualizing data can reveal trends and insights. For this example, we’ll use boxplots to compare the price distributions of Galaxy Note 8 and iPhone 8.

---

## Required Libraries and Setup

To implement this project, you’ll need Python, `pip` (Python's package installer), and the following libraries:
- **BeautifulSoup**: For scraping HTML content.
- **Pandas and NumPy**: For data manipulation and analysis.

### Installation Steps

1. **Installing Python and Pip**  
   Follow [this guide](https://realpython.com/installing-python/) to set up Python and Pip on your system.

2. **Installing Required Libraries**  
   Run the following commands:
   ```bash
   pip install beautifulsoup4
   pip install pandas
   pip install numpy
   ```

Once the setup is complete, you’re ready to start scraping!

---

## Python Code for Scraping eBay

Below, we’ll perform scraping for two products: Galaxy Note 8 and iPhone 8. The implementation for each product is repeated for clarity.

### Scraping eBay for Galaxy Note 8

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
from scipy import stats
import numpy as np

item_name = []
prices = []

for i in range(1, 10):
    ebay_url = f"https://www.ebay.com/sch/i.html?_from=R40&_nkw=note+8&_sacat=0&_pgn={i}"
    response = requests.get(ebay_url)
    soup = BeautifulSoup(response.text, "html.parser")

    listings = soup.find_all('li', attrs={'class': 's-item'})

    for listing in listings:
        name = listing.find('h3', attrs={'class': 's-item__title'})
        price = listing.find('span', attrs={'class': 's-item__price'})

        if name and price:
            item_name.append(name.text)
            prices.append(int(price.text.replace("$", "").replace(",", "")))

# Data cleanup
data_note_8 = pd.DataFrame({"Name": item_name, "Prices": prices})
data_note_8 = data_note_8[(np.abs(stats.zscore(data_note_8["Prices"])) < 3)]
print(data_note_8)
```

---

### Scraping eBay for iPhone 8

```python
item_name = []
prices = []

for i in range(1, 10):
    ebay_url = f"https://www.ebay.com/sch/i.html?_from=R40&_nkw=iphone+8&_sacat=0&_pgn={i}"
    response = requests.get(ebay_url)
    soup = BeautifulSoup(response.text, "html.parser")

    listings = soup.find_all('li', attrs={'class': 's-item'})

    for listing in listings:
        name = listing.find('h3', attrs={'class': 's-item__title'})
        price = listing.find('span', attrs={'class': 's-item__price'})

        if name and price:
            item_name.append(name.text)
            prices.append(int(price.text.replace("$", "").replace(",", "")))

# Data cleanup
data_iphone_8 = pd.DataFrame({"Name": item_name, "Prices": prices})
data_iphone_8 = data_iphone_8[(np.abs(stats.zscore(data_iphone_8["Prices"])) < 3)]
print(data_iphone_8)
```

---

## Visualizing Product Prices

Here’s how to visualize price distributions using boxplots:

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(10, 6))
plt.boxplot([data_note_8['Prices'], data_iphone_8['Prices']], labels=["Galaxy Note 8", "iPhone 8"])
plt.title("Price Comparison of Galaxy Note 8 and iPhone 8 on eBay")
plt.ylabel("Price ($)")
plt.show()
```

---

## Conclusion

In this tutorial, we demonstrated how to scrape eBay product data using Python and BeautifulSoup. We compared the prices of Galaxy Note 8 and iPhone 8, providing actionable insights through data visualization. Web scraping combined with data science can empower informed decisions, whether for businesses or personal use.

For larger-scale projects or assistance, consider using a professional tool like [ScraperAPI](https://www.scraperapi.com/?fp_ref=coupons) to handle complex scraping needs.

Start leveraging web scraping today to stay ahead in your market!

</article>
