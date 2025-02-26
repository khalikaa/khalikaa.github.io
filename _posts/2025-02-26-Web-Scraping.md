---
title: Simple Web Scraping Tutorial with Python & BeautifulSoup
image:
    path: "/assets/img/web-scraping.png"
---

The internet is filled with valuable data, from product prices and news articles to stock market trends and social media insights. But manually copying and pasting data from web pages is time-consuming and inefficient. This is where web scraping comes in.

Web scraping is the process of automatically extracting data from websites. It allows you to gather information quickly and efficiently, making it useful for research, business intelligence, and even personal projects. In this tutorial, we will explore how to perform simple web scraping using **BeautifulSoup**, a powerful and beginner-friendly Python library for parsing HTML and extracting useful data.

## **How Web Scraping Works**
![Web Scraping Step](/assets/img/web-scraping-step.png)
Web scraping follows a structured process to extract and store data efficiently. Here’s how it works:

**1. Extracting Data from Web Pages** <br>
Every website is built using HTML, which structures its content. When you visit a webpage, your browser renders this HTML code to display text, images, tables, and other elements. Web scraping works by programmatically accessing this raw HTML to extract useful data.

**2. Using Web Scraping Tools** <br>
To scrape data, we use tools like BeautifulSoup (for parsing HTML) or Selenium (for interacting with dynamic websites). These tools help us navigate the webpage structure, locate the necessary data, and extract it efficiently.

**3. Storing and Structuring the Data** <br>
Once the data is extracted, it needs to be cleaned and stored in a structured format. This could be a CSV file, a database, or even a JSON file for further processing and analysis. With structured data, you can easily perform tasks like trend analysis, machine learning, or automation.

## **What Will We Do in This Tutorial**
![Web Scraping FTTT](/assets/img/web-scraping-fttt.png)
In this tutorial, we will extract country-related data from the following webpage: <br>
🔗[scrapethissite.com/pages/simple/](https://www.scrapethissite.com/pages/simple/) <br>
Using this page, we will:

**1. Scrape Country Data –** Extract country names, capitals, populations, and areas. <br>
**2. Parse the HTML –** Use BeautifulSoup to locate and extract relevant information. <br>
**3. Store Data in CSV –** Save the extracted data in a structured CSV file for further use.<br>

By the end of this tutorial, you'll have a functional web scraper that transforms raw webpage content into structured data. Let’s dive in! 

## **Requirements**
Before we start, make sure you have the following installed:

◉ **Python** – Download from [python.org](https://www.python.org/downloads/) if you haven’t installed it yet. <br>
◉ **BeautifulSoup & Requests** – Required for web scraping. <br>
◉ **Pandas** – Useful for saving data to CSV.<br>

You can install them all at once using the following command:
```bash
pip install beautifulsoup4 requests pandas
```
Once you have these installed, you're ready to start scraping! 

## **Step-by-Step Tutorial**
#### **1️⃣ Fetching the Web Page**
Before extracting data, we first need to fetch the webpage's HTML content. This can be done using the   `requests` library.

```python
# Importing the 'requests' library to send HTTP requests
import requests 

# URL of the website we want to scrape
url = "https://www.scrapethissite.com/pages/simple/"

# Sending a GET request to the provided URL to retrieve the webpage
response = requests.get(url)
```
Next, we need to check if the request was successful by checking the status code
```python
if response.status_code == 200:
    # If the request was successful (status code = 200), print the first 500 characters of the response content
    print(response.text[:500]) 
    # Displaying the first 500 characters of the HTML content for preview
else:
    # If the request failed, print an error message with the status code
    print(f"Failed to retrieve the page, status code: {response.status_code}")
```

#### **2️⃣ Parsing HTML with BeautifulSoup**
Now that we have the raw HTML, we need to parse it using BeautifulSoup to extract specific elements.
```py
from bs4 import BeautifulSoup

# Parses the HTML content of the response using BeautifulSoup with the "html.parser" parser.
soup = BeautifulSoup(response.text, "html.parser")
# Displaying formatted HTML (first 1000 characters)
print(soup.prettify()[:1000]) 
```

#### **3️⃣ Extracting Specific Data**
Once we have successfully parsed the webpage's HTML, it's time to extract the information we need. For this step, we'll use `BeautifulSoup` to search the document and retrieve the specific data about each country.

We can find all the elements that contain the country information using the `find_all()` method. In this case, we're looking for `<div>` elements with the class `country`.
```py
countries = soup.find_all("div", class_="country")
```
After that, we loop through each country element and extract specific details such as the country name, capital, population, and area. We use the `find()` method to get the values from each element, clean them up with `.text.strip()`, and then print them out for verification.
```py
for country in countries:
    name = country.find("h3", class_="country-name").text.strip()
    capital = country.find("span", class_="country-capital").text.strip()
    population = country.find("span", class_="country-population").text.strip()
    area = country.find("span", class_="country-area").text.strip()
    
    print(f"{name} - Capital: {capital}, Population: {population}, Area: {area} sq km")
```
At this point, you can check if the extracted information looks correct by printing it out. This step is useful for ensuring everything is working as expected before we save the data into an array for further use.

#### **4️⃣ Saving Data to CSV**
First, we will extract the data for each country and store it in a list.
```py
# Initialize an empty list to hold the data
data = []

# Loop through each country to extract relevant details
for country in countries:
    name = country.find("h3", class_="country-name").text.strip()
    capital = country.find("span", class_="country-capital").text.strip()
    population = country.find("span", class_="country-population").text.strip()
    area = country.find("span", class_="country-area").text.strip()
    
    # Append the extracted data as a list to the 'data' list
    data.append([name, capital, population, area])
```
Next, we will convert the array into a Pandas DataFrame and save it as a CSV file.
```python
import pandas as pd

# Define the column names for the DataFrame
columns = ["Country", "Capital", "Population", "Area (sq km)"]

# Create a DataFrame from the extracted data
df = pd.DataFrame(data, columns=columns)

# Save the DataFrame to a CSV file
df.to_csv("countries_data.csv", index=False)

# Confirm that the data has been saved
print("Data saved to countries_data.csv")
```
And that's it! 🎉 You’ve successfully built a simple web scraper using Python and BeautifulSoup. Through this tutorial, you’ve learned how to:

✅ Fetch webpage content using `requests` <br>
✅ Parse and navigate HTML with BeautifulSoup <br>
✅ Extract specific data efficiently <br>
✅ Save the extracted data into a structured CSV file <br>

This is just the beginning! Web scraping opens up endless possibilities, from data analysis and automation to competitive research and beyond. If you’re interested in scraping more complex websites, consider exploring Selenium for dynamic pages or handling API requests for structured data.

Remember to always follow ethical guidelines and respect website terms of service when scraping.

Now go ahead, experiment, and start building your own web scraping projects! Happy coding!⭐