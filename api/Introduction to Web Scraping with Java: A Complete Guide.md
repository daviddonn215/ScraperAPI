
# Introduction to Web Scraping with Java: A Complete Guide

Web scraping allows you to extract structured data from websites, even when no standardized API is available. If you need to gather information from a site without a REST API, **web scraping with Java** can provide an efficient solution. In this guide, we’ll explore the fundamental concepts of web scraping, demonstrate how to build a basic crawler, and provide tips to scale your scraping projects effectively.

---

Stop wasting time on proxies and CAPTCHAs! ScraperAPI's simple API handles millions of web scraping requests, so you can focus on the data. Get structured data from Amazon, Google, Walmart, and more. Start your free trial today! 👉 [ScraperAPI](https://www.scraperapi.com/?fp_ref=coupons)

---

## What is Web Scraping?

Web scraping, also known as web crawling, involves fetching and extracting data from websites. The process typically includes:
- **Downloading** a website’s HTML.
- **Parsing** the HTML to understand its structure.
- **Extracting** the desired data.

Web scraping is used in various scenarios, such as:
- **Competitor analysis**: Monitoring pricing changes or offerings.
- **Market research**: Aggregating news or industry trends.
- **Marketing**: Collecting addresses for campaigns.

---

## Prerequisites for Java Web Scraping

Before starting, ensure you have the following:
1. A basic understanding of **Java programming**.
2. Familiarity with **XPath** for querying HTML documents.
3. **Java Dependencies**:
   - [HtmlUnit](https://htmlunit.sourceforge.io): A headless browser for Java-based scraping.
   - [Jackson](https://github.com/FasterXML/jackson): A library for handling JSON.

### Dependency Configuration
Add the required dependencies to your `pom.xml` file for a Maven project. This ensures HtmlUnit and Jackson are available for your scraping project.

---

## Building a Web Scraper with Java

### Setting Up the Project

In this example, we’ll scrape **Craigslist** to extract classified ads in New York selling an iPhone 13. Since Craigslist doesn’t provide a public API, scraping is the only viable option.

#### Finding the Search URL
When you perform a search on Craigslist, the resulting URL contains query parameters. For example:
```
https://newyork.craigslist.org/search/moa?query=iphone%2013
```
We’ll use this URL format to fetch the required data programmatically.

---

### Fetching the Page

To send an HTTP request to Craigslist, use HtmlUnit’s `WebClient`. Configure the client to disable CSS and JavaScript for faster scraping.

```java
// Define the search term
String searchQuery = "iphone 13";

// Instantiate the client
WebClient client = new WebClient();
client.getOptions().setCssEnabled(false);
client.getOptions().setJavaScriptEnabled(false);

// Set up the URL with the search term and send the request
String searchUrl = "https://newyork.craigslist.org/search/moa?query=" + URLEncoder.encode(searchQuery, "UTF-8");
HtmlPage page = client.getPage(searchUrl);
```

At this point, the `page` variable contains the HTML content of the Craigslist results page.

---

### Extracting Data

Using the browser’s developer tools (`F12`), identify the HTML structure of the desired data. Craigslist lists ads within `<li>` tags under a `<ul>` container with the ID `search-results`. Each `<li>` has the class `result-row`.

#### Using XPath for Extraction
Use HtmlUnit’s `getByXPath` method to extract data based on XPath expressions. For each ad, extract:
- **Title**: Located in a `<div>` with the class `title`.
- **Price**: Found in a `<div>` with the class `price`.
- **Location**: Optional, within a `<div>` with the class `location`.

```java
for (var htmlItem : page.<HtmlElement>getByXPath("//li[contains(@class,'result-row')]")) {
    HtmlAnchor itemAnchor = htmlItem.getFirstByXPath(".//a");
    HtmlElement itemTitle = htmlItem.getFirstByXPath(".//div[@class='title']");
    HtmlElement itemPrice = htmlItem.getFirstByXPath(".//div[@class='price']");
    HtmlElement itemLocation = htmlItem.getFirstByXPath(".//div[@class='location']");

    if (itemAnchor != null && itemTitle != null) {
        System.out.printf("Name: %s, Price: %s, Location: %s, URL: %s%n",
                itemTitle.asNormalizedText(),
                itemPrice.asNormalizedText(),
                (itemLocation == null) ? "N/A" : itemLocation.asNormalizedText(),
                itemAnchor.getHrefAttribute());
    }
}
```

---

### Converting Data to JSON

For structured data storage, convert the scraped information into JSON using Jackson. Create a **POJO** (Plain Old Java Object) to represent each item.

#### Example POJO
```java
record Item(String name, BigDecimal price, String location, String url) {}
```

#### JSON Conversion
Use Jackson’s `ObjectMapper` to convert Java objects into JSON strings.

```java
ObjectMapper objectMapper = new ObjectMapper();

if (itemAnchor != null && itemTitle != null) {
    var item = new Item(
            itemTitle.asNormalizedText(),
            new BigDecimal(itemPrice.asNormalizedText().replace("$", "")),
            (itemLocation == null) ? "N/A" : itemLocation.asNormalizedText(),
            itemAnchor.getHrefAttribute()
    );
    System.out.println(objectMapper.writeValueAsString(item));
}
```

---

## Scaling Your Web Scraper

### Multi-City Scraping
Craigslist categorizes ads by city (e.g., `https://newyork.craigslist.org`). To scrape ads from multiple cities, maintain a list of city subdomains and iterate through them.

```java
List<String> cities = List.of("newyork", "boston", "washington");
for (String city : cities) {
    String searchUrl = "https://" + city + ".craigslist.org/search/moa?query=iphone%2013";
    // Fetch and process data for each city
}
```

### Parallelization
For faster performance, scrape multiple cities simultaneously using Java’s `CompletableFuture` and virtual threads.

```java
List<CompletableFuture<Void>> futures = cities.stream()
        .map(city -> CompletableFuture.runAsync(() -> {
            // Fetch data for the city
        }))
        .toList();

CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
```

---

## Handling Challenges in Web Scraping

### Anti-Scraping Measures
To avoid being blocked, consider:
- **Using Proxies**: Services like [ScraperAPI](https://www.scraperapi.com/?fp_ref=coupons) can handle proxies and CAPTCHA challenges automatically.
- **Throttling Requests**: Mimic human behavior by introducing delays between requests.

### Pagination
To handle paginated results, identify the “next” button’s link and programmatically fetch subsequent pages.

---

## Conclusion

Web scraping with Java is a powerful way to gather data from websites without APIs. Using tools like HtmlUnit and Jackson, you can build efficient crawlers and structure the extracted data into JSON. For larger-scale projects, parallelize your requests and leverage services like ScraperAPI to overcome anti-scraping measures.

Ready to simplify your web scraping journey? Start your free trial with ScraperAPI today! 👉 [ScraperAPI](https://www.scraperapi.com/?fp_ref=coupons)
