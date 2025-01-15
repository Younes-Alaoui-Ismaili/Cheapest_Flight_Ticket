# Cheapest_Flight_Ticket
Modify the server.js file to serve static files (like images).

javascript
Copy
const express = require('express');
const fs = require('fs');
const path = require('path');
const app = express();
const port = 3000;

// Serve static files
app.use(express.static(path.join(__dirname, 'public')));

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
Organizing Static Files:
Create a folder named public at the root of your project and place the images folder inside it.

Copy
flight-finder/
├── public/
│   └── images/
│       └── flight.jpg
├── scraper.js
├── server.js
├── index.html
└── package.json
Step 3: Test the Application
Start the Server:

bash
Copy
node server.js
Access the Frontend:
Open your browser and navigate to http://localhost:3000. You should see the image, the "Fetch Flights" button, and a space to display flight data.

Step 4: Additional Improvements
You can enhance the frontend by adding extra features, such as:

Filtering Flights: Add form fields to filter flights by date, price, etc.

Displaying Data in a Table Format: Use a library like DataTables to display data in an interactive table.

Responsive Design: Use media queries to make the user interface responsive.
