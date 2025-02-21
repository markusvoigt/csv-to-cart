# CSV Cart Manager

This project allows users to upload a CSV file to fill their cart and download the current cart as a CSV file. It is designed to work with Shopify's cart system.

[Demo video](https://screenshot.click/21-58-oohjx-86ujw.mp4)

## Features

- **Upload CSV**: Users can upload a CSV file containing item IDs and quantities to populate their cart.
- **Download Cart**: Users can download the current cart contents as a CSV file.

## Usage

1. **Upload your CSV to fill the cart**:
   - Click on the file input to select a CSV file.
   - The CSV file should contain two columns: item ID and quantity, separated by commas.
   - Note: Shopify does not support carts with more than 500 line items.

2. **Download your current cart as a CSV file**:
   - Click the "Download Cart as CSV" button to download the current cart contents.

## Code Snippet

Place the following code snippet in your HTML where you want the upload and download functionality to appear. It is recommended to place it before the `</cart-items>` closing tag.

```html
<div>
    <p>Upload your CSV to fill the cart:</p>
    <input id="csv" type="file" accept=".csv">
</div>
<div>
    <p>Download your current cart as CSV file:</p>
    <button id="download-cart">Download Cart as CSV</button>
</div>

<script>
    // Get the file input element and the download button from the DOM
    const fileInput = document.getElementById('csv');
    const downloadButton = document.getElementById('download-cart');

    // Function to read the uploaded CSV file
    const readFile = () => {
        const reader = new FileReader(); // Create a new FileReader instance

        // Define what happens when the file is loaded
        reader.onload = () => {
            const rows = reader.result.split("\n"); // Split the file content into rows based on new lines

            // Check if the number of rows exceeds 500, alert the user if it does
            if (rows.length > 500) {
                alert("Shopify does currently not support carts with > 500 line items! Please split your order or contact us.");
                return; // Exit the function if the condition is met
            }

            const items = []; // Initialize an array to hold the items from the CSV

            // Iterate over each row in the CSV
            rows.forEach((row) => {
                const item = {}; // Create an object to hold the item data
                const columns = row.split(","); // Split the row into columns based on commas

                // Check if the row has exactly 2 columns (id and quantity)
                if (columns.length !== 2) return; // Skip this row if it doesn't have 2 columns

                // Assign the values from the columns to the item object
                item.id = columns[0]; // First column is the item ID
                item.quantity = columns[1]; // Second column is the item quantity
                items.push(item); // Add the item object to the items array
            });

            // Create the form data object to send to Shopify
            let formData = {
                'items': items // Assign the items array to the 'items' property
            };

            // Send a POST request to add items to the Shopify cart
            fetch(window.Shopify.routes.root + 'cart/add.js', {
                method: 'POST', // Specify the request method
                headers: {
                    'Content-Type': 'application/json' // Set the content type to JSON
                },
                body: JSON.stringify(formData) // Convert the formData object to a JSON string
            })
            .then(response => {
                window.location = window.location; // Refresh the page after adding items
                // Clear the file input after processing
                fileInput.value = ''; // Reset the file input to allow for a new file selection
                return response.json(); // Parse the JSON response
            })
            .catch((error) => {
                console.error('Error:', error); // Log any errors to the console
            });
        };

        // Read the selected file as a binary string
        reader.readAsBinaryString(fileInput.files[0]);
    };

    // Function to download the current cart as a CSV file
    const downloadCart = () => {
        // Fetch the current cart data from Shopify
        fetch(window.Shopify.routes.root + 'cart.js')
            .then(response => response.json()) // Parse the JSON response
            .then(cart => {
                const csvRows = []; // Initialize an array to hold CSV rows

                // Iterate over each item in the cart
                cart.items.forEach(item => {
                    const row = [
                        item.id, // Get the item ID
                        item.quantity // Get the item quantity
                    ];
                    csvRows.push(row.join(',')); // Join the row values with a comma and add to csvRows
                });

                const csvData = csvRows.join('\n'); // Join all rows with a new line to create the CSV data
                const blob = new Blob([csvData], { type: 'text/csv' }); // Create a Blob object for the CSV data
                const url = URL.createObjectURL(blob); // Create a URL for the Blob

                const a = document.createElement('a'); // Create a temporary anchor element
                a.href = url; // Set the href to the Blob URL
                a.download = 'cart.csv'; // Set the download attribute with the desired file name
                document.body.appendChild(a); // Append the anchor to the document body
                a.click(); // Programmatically click the anchor to trigger the download
                document.body.removeChild(a); // Remove the anchor from the document
                URL.revokeObjectURL(url); // Release the Blob URL
            })
            .catch((error) => {
                console.error('Error fetching cart:', error); // Log any errors to the console
            });
    };

    // Add event listeners for file input change and download button click
    fileInput.addEventListener('change', readFile);
    downloadButton.addEventListener('click', downloadCart);
</script>