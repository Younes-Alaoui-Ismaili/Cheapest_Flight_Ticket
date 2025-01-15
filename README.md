To build a full-stack application for scraping affordable flight ticket data, we'll use the "FlightFinder-Affordable-Flight-Ticket-Scraper-Node.js-master" project as a base. The application will consist of a backend built with Node.js for scraping and a frontend for displaying the data. Below is a step-by-step guide to building this application.
To build a full-stack application for scraping affordable flight ticket data, we'll use the "FlightFinder-Affordable-Flight-Ticket-Scraper-Node.js-master" project as a base. The application will consist of a backend built with Node.js for scraping and a frontend for displaying the data. Below is a step-by-step guide to building this application.

### Step 1: Set Up the Project

1. **Initialize the Project:**
   ```bash
   mkdir flight-finder
   cd flight-finder
   npm init -y
   ```

2. **Install Required Dependencies:**
   ```bash
   npm install express puppeteer cheerio csv-writer dotenv
   ```

### Step 2: Backend Development

1. **Create the Scraper:**
   Create a file named `scraper.js` and implement the scraping logic using Puppeteer and Cheerio.

   ```javascript
   const puppeteer = require('puppeteer');
   const cheerio = require('cheerio');
   const csv = require('csv-writer').createObjectCsvWriter;
   require('dotenv').config();

   const scrapeFlights = async (url) => {
       const browser = await puppeteer.launch({ headless: true });
       const page = await browser.newPage();
       await page.goto(url, { waitUntil: 'networkidle2' });

       const content = await page.content();
       const $ = cheerio.load(content);

       const flights = [];
       $('.flight-item').each((index, element) => {
           const flight = {
               flightNumber: $(element).find('.flight-number').text().trim(),
               departureTime: $(element).find('.departure-time').text().trim(),
               arrivalTime: $(element).find('.arrival-time').text().trim(),
               price: $(element).find('.price').text().trim(),
               airline: $(element).find('.airline').text().trim()
           };
           flights.push(flight);
       });

       await browser.close();
       return flights;
   };

   const saveToCSV = (flights) => {
       const csvWriter = csv({
           path: 'flights.csv',
           header: [
               { id: 'flightNumber', title: 'Flight Number' },
               { id: 'departureTime', title: 'Departure Time' },
               { id: 'arrivalTime', title: 'Arrival Time' },
               { id: 'price', title: 'Price' },
               { id: 'airline', title: 'Airline' }
           ]
       });

       csvWriter.writeRecords(flights)
           .then(() => console.log('CSV file was written successfully'));
   };

   (async () => {
       const url = 'https://example.com/flights'; // Replace with the actual URL
       const flights = await scrapeFlights(url);
       saveToCSV(flights);
   })();
   ```

2. **Error Handling and Retry Mechanism:**
   Implement error handling and a retry mechanism to handle network issues and rate limits.

   ```javascript
   const scrapeFlights = async (url, retries = 3) => {
       for (let i = 0; i < retries; i++) {
           try {
               const browser = await puppeteer.launch({ headless: true });
               const page = await browser.newPage();
               await page.goto(url, { waitUntil: 'networkidle2' });

               const content = await page.content();
               const $ = cheerio.load(content);

               const flights = [];
               $('.flight-item').each((index, element) => {
                   const flight = {
                       flightNumber: $(element).find('.flight-number').text().trim(),
                       departureTime: $(element).find('.departure-time').text().trim(),
                       arrivalTime: $(element).find('.arrival-time').text().trim(),
                       price: $(element).find('.price').text().trim(),
                       airline: $(element).find('.airline').text().trim()
                   };
                   flights.push(flight);
               });

               await browser.close();
               return flights;
           } catch (error) {
               console.error(`Attempt ${i + 1} failed: ${error.message}`);
               if (i === retries - 1) throw error;
           }
       }
   };
   ```

3. **Proxy Support:**
   Add proxy support to avoid IP blocking.

   ```javascript
   const puppeteer = require('puppeteer');
   require('dotenv').config();

   const scrapeFlights = async (url) => {
       const browser = await puppeteer.launch({
           headless: true,
           args: [`--proxy-server=${process.env.PROXY_SERVER}`]
       });
       const page = await browser.newPage();
       await page.authenticate({
           username: process.env.PROXY_USER,
           password: process.env.PROXY_PASS
       });
       await page.goto(url, { waitUntil: 'networkidle2' });

       const content = await page.content();
       const $ = cheerio.load(content);

       const flights = [];
       $('.flight-item').each((index, element) => {
           const flight = {
               flightNumber: $(element).find('.flight-number').text().trim(),
               departureTime: $(element).find('.departure-time').text().trim(),
               arrivalTime: $(element).find('.arrival-time').text().trim(),
               price: $(element).find('.price').text().trim(),
               airline: $(element).find('.airline').text().trim()
           };
           flights.push(flight);
       });

       await browser.close();
       return flights;
   };
   ```

### Step 3: Frontend Development

1. **Create an Express Server:**
   Create a file named `server.js` to serve the scraped data.

   ```javascript
   const express = require('express');
   const fs = require('fs');
   const path = require('path');
   const app = express();
   const port = 3000;

   app.get('/flights', (req, res) => {
       const filePath = path.join(__dirname, 'flights.csv');
       if (fs.existsSync(filePath)) {
           res.sendFile(filePath);
       } else {
           res.status(404).send('File not found');
       }
   });

   app.listen(port, () => {
       console.log(`Server running at http://localhost:${port}`);
   });
   ```

2. **Create a Simple Frontend:**
   Create an `index.html` file to display the scraped data.

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Flight Finder</title>
   </head>
   <body>
       <h1>Flight Finder</h1>
       <button id="fetchFlights">Fetch Flights</button>
       <pre id="flightsData"></pre>

       <script>
           document.getElementById('fetchFlights').addEventListener('click', async () => {
               const response = await fetch('/flights');
               const data = await response.text();
               document.getElementById('flightsData').textContent = data;
           });
       </script>
   </body>
   </html>
   ```

### Step 4: Run the Application

1. **Start the Server:**
   ```bash
   node server.js
   ```

2. **Access the Frontend:**
   Open your browser and navigate to `http://localhost:3000` to see the frontend. Click the "Fetch Flights" button to load and display the scraped flight data.

### Step 5: Deploy the Application

You can deploy this application on platforms like Heroku, Vercel, or any other cloud service that supports Node.js applications. Make sure to set up environment variables for sensitive data like proxy credentials and API keys.

### Conclusion

This full-stack application provides a robust solution for scraping and displaying affordable flight ticket data. The backend handles the scraping, error handling, and data storage, while the frontend provides a simple interface for users to view the data. This setup can be further enhanced with additional features like user authentication, advanced filtering, and more.
