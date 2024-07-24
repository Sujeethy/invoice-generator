# Invoice Generator Documentation

## Overview
This HTML and JavaScript program is designed to generate an invoice, display it in the browser, and allow the user to download it as a PDF. It utilizes the `html2canvas` and `jspdf` libraries to achieve this functionality.

## Features
- Display seller, billing, and shipping details.
- List items with descriptions, unit prices, discounts (if applicable), quantities, tax rates, and totals.
- Calculate and display total tax amount and total amount.
- Convert numerical amounts to words.
- Download the invoice as a PDF.
- Handle cases where a discount is not provided for an item.
- Validate input parameters and handle potential errors gracefully.
- Designed for performance and scalability to handle a large volume of orders.
- Platform-agnostic and readily deployable.

## Files
- **index.html**: Main HTML file containing the structure and inline CSS/JavaScript for the invoice.
- **style.css**: (Optional) External stylesheet for additional styling, linked in the HTML.

## Dependencies
- **html2canvas**: Library to capture a screenshot of the HTML element.
  - Source: `https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js`
- **jspdf**: Library to generate PDF files.
  - Source: `https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.4.0/jspdf.umd.min.js`

## Implementation Details

### HTML Structure
- **Header**: Displays the logo and the invoice title.
- **Content**: Divided into left and right sections for seller details and billing/shipping details.
- **Table**: Lists the items with their details.
- **Footer**: Displays the seller's name and an authorized signatory section.
- **Buttons**: A button to trigger the PDF download.

### JavaScript Functions
- **updateInvoice(data)**: Populates the invoice with data provided.
  - **data**: An object containing all necessary invoice details (e.g., seller details, billing details, items).
  - Validates various elements of the invoice.
  - Handles situations where a discount is not provided by checking if the discount value exists or setting it to 0.
  - Calculates net amounts, tax amounts, and totals for each item.
  - Updates the item rows, total tax amount, total amount, and amount in words.

```javascript
function updateInvoice(data) {
    // Validate data
    if (!data || typeof data !== 'object') {
        console.error('Invalid data format');
        return;
    }

    const {
        sellerDetails,
        billingDetails,
        shippingDetails,
        placeOfSupply,
        placeOfDelivery,
        items
    } = data;

    // Validate sellerDetails
    if (!sellerDetails || typeof sellerDetails !== 'object') {
        console.error('Invalid sellerDetails');
        return;
    }
    if (!sellerDetails.address || !sellerDetails.pan || !sellerDetails.gst) {
        console.error('Missing seller details');
        return;
    }

    // Validate billingDetails
    if (!billingDetails || typeof billingDetails !== 'object') {
        console.error('Invalid billingDetails');
        return;
    }
    if (!billingDetails.address || !billingDetails.stateCode) {
        console.error('Missing billing details');
        return;
    }

    // Validate shippingDetails
    if (!shippingDetails || typeof shippingDetails !== 'object') {
        console.error('Invalid shippingDetails');
        return;
    }
    if (!shippingDetails.address || !shippingDetails.stateCode) {
        console.error('Missing shipping details');
        return;
    }

    // Validate placeOfSupply and placeOfDelivery
    if (typeof placeOfSupply !== 'string' || typeof placeOfDelivery !== 'string') {
        console.error('Invalid placeOfSupply or placeOfDelivery');
        return;
    }

    // Validate items
    if (!Array.isArray(items)) {
        console.error('Items should be an array');
        return;
    }

    let itemsHtml = '';
    let totalTaxAmount = 0;
    let totalAmount = 0;

    items.forEach((item, index) => {
        if (typeof item !== 'object' || item === null) {
            console.error(`Invalid item at index ${index}`);
            return;
        }

        const {
            description,
            unitPrice,
            discount,
            qty,
            taxRate
        } = item;

       
        if (typeof description !== 'string' || typeof unitPrice !== 'number' || typeof discount !== 'number' ||
            typeof qty !== 'number' || typeof taxRate !== 'number') {
            console.error(`Invalid properties for item at index ${index}`);
            return;
        }

        if (unitPrice < 0 || discount < 0 || qty < 0 || taxRate < 0) {
            console.error(`Negative values found in item at index ${index}`);
            return;
        }

        const netAmount = unitPrice * qty - discount;
        const taxAmount = (netAmount * taxRate) / 100;
        const total = netAmount + taxAmount;

        totalTaxAmount += taxAmount;
        totalAmount += total;

        itemsHtml += `
            <tr>
                <td>${index + 1}</td>
                <td>${description}</td>
                <td>${unitPrice.toFixed(2)}</td>
                <td>${discount.toFixed(2)}</td>
                <td>${qty}</td>
                <td>${netAmount.toFixed(2)}</td>
                <td>${taxRate}%</td>
                <td>${placeOfSupply === placeOfDelivery ? 'CGST+SGST' : 'IGST'}</td>
                <td>${taxAmount.toFixed(2)}</td>
                <td>${total.toFixed(2)}</td>
            </tr>`;
    });

    
    document.getElementById('sold-by-address').innerText = sellerDetails.address;
    document.getElementById('pan-no').innerText = sellerDetails.pan;
    document.getElementById('gst-no').innerText = sellerDetails.gst;
    document.getElementById('billing-address').innerText = billingDetails.address;
    document.getElementById('billing-state-code').innerText = billingDetails.stateCode;
    document.getElementById('shipping-address').innerText = shippingDetails.address;
    document.getElementById('shipping-state-code').innerText = shippingDetails.stateCode;
    document.getElementById('place-of-supply').innerText = placeOfSupply;
    document.getElementById('place-of-delivery').innerText = placeOfDelivery;
    document.getElementById('item-rows').innerHTML = itemsHtml;
    document.getElementById('total-tax-amount').innerText = totalTaxAmount.toFixed(2);
    document.getElementById('total-amount').innerText = totalAmount.toFixed(2);
    document.getElementById('amount-in-words').innerText = numberToWords(totalAmount);
    document.getElementById('seller-name').innerText = sellerDetails.name;
    document.getElementById('signature-temp').innerHTML = canvasToSVG();
}
```

- **numberToWords(amount)**: Converts numerical amounts to words.
  - This function should be implemented to convert numbers to their word equivalents.

- **downloadPDF()**: Captures the invoice as an image and generates a PDF.
  - Uses `html2canvas` to capture the body of the document as an image.
  - Adds the captured image to a PDF using `jsPDF` and saves it.

```javascript
function downloadPDF() {
  const { jsPDF } = window.jspdf;

  html2canvas(document.body, {
    scale: 2 // Adjust scale for higher resolution
  }).then(canvas => {
    const imgData = canvas.toDataURL('image/png');
    const pdf = new jsPDF('p', 'pt', 'a4');
    const imgProps = pdf.getImageProperties(imgData);
    const pdfWidth = pdf.internal.pageSize.getWidth();
    const pdfHeight = (imgProps.height * pdfWidth) / imgProps.width;

    pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
    pdf.save('invoice.pdf');
  });
}
```

- **Input Validation and Error Handling**: Ensures all required data fields are provided and valid.

```javascript
function validateData(data) {
  if (!data.sellerDetails || !data.billingDetails || !data.shippingDetails || !data.orderDetails || !data.items) {
    throw new Error('Missing required invoice data');
  }
  // Add more specific validations as needed
}

document.addEventListener('DOMContentLoaded', () => {
  try {
    validateData(invoiceData);
    updateInvoice(invoiceData);
  } catch (error) {
    console.error('Error updating invoice:', error);
    alert('There was an error generating the invoice. Please check the console for more details.');
  }

  const button = document.createElement('button');
  button.innerText = 'Download PDF';
  button.onclick = downloadPDF;
  document.body.appendChild(button);
});
```

### Usage
1. **Load the Invoice Data**: The `invoiceData` object at the top of the script contains sample data for the invoice. Modify this object to fit your actual invoice details.
2. **Initialize the Invoice**: The `updateInvoice(invoiceData)` function is called on `DOMContentLoaded` to populate the invoice with the provided data.
3. **Download PDF**: Clicking the "Download PDF" button triggers the `downloadPDF()` function to save the invoice as a PDF file.

### Performance and Scalability
To handle a large volume of orders:
- Use AJAX or other APIs to fetch and load data dynamically

.
- Optimize JavaScript code for performance by minimizing DOM manipulations and avoiding unnecessary calculations.

### Platform-Agnostic Deployment
Ensure the program is platform-agnostic by using standard HTML, CSS, and JavaScript without relying on platform-specific features.

## Example HTML

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Invoice Generator</title>
  <style>
    /* Add your CSS styles here */
  </style>
</head>
<body>
  <div id="invoice">
    <!-- Invoice content goes here -->
  </div>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.4.0/jspdf.umd.min.js"></script>
  <script>
    // JavaScript code goes here
  </script>
</body>
</html>
```

By following this documentation, you can create and customize an invoice generator that fits your specific needs.