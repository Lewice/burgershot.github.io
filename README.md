<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Shop menu</title>
  <script src="https://code.jquery.com/jquery-3.4.1.js" integrity="sha256-WpOohJOqMqqyKL9FccASB9O0KwACQJpFTUBLTYOVvVU=" crossorigin="anonymous"></script>
  <style>
    body, h2, form, label, p, button, select, input {
      font-size: 8;
      margin-right: 10px; /* Add a margin for spacing */
    }

    label {
		display: block;
		margin-bottom: 5px;
	}

    body, h2, form {
      text-align: center;
    }

    p {
      text-align: center;
      margin: 0; /* Remove default margin for <p> */
    }

    button {
      margin-top: 10px;
    }
	body, h2 {
	font-weight: bold;
	}
	body {
	background-color: grey;
	}
  </style>
  <script>
    function calculateTotals() {
  let total = 0;

  // Calculate total from selected items
  const menuItems = document.querySelectorAll('.menu-item:checked');
  menuItems.forEach(item => {
    const price = parseFloat(item.dataset.price);
    const quantity = parseInt(item.nextElementSibling.value);

    if (!isNaN(price) && !isNaN(quantity) && quantity > 0) {
      // Exclude "Mystery Box" from discounts
      if (item.classList.contains('exclude-discount')) {
        total += price * quantity;
      } else {
        total += price * quantity * (1 - ($("#discount").val() / 100));
      }
    }
  });

  // Change commission rate from 5% to 10%
  const commission = total * 0.50;

  document.getElementById('total').innerText = total.toFixed(2);
  document.getElementById('commission').innerText = commission.toFixed(2);
}

    function SubForm() {
  // Check if the total is not calculated
  const total = $("#total").text().trim();
  if (total === "") {
    alert("Please calculate the total first!");
    return;
  }

  // Check if the employee name is provided
  const employeeName = $("#employeeName").val();
  if (employeeName.trim() === "") {
    alert("Employee Name is required!");
    return;
  }

  // Get selected menu items and quantities
  const orderedItems = [];
  const menuItems = document.querySelectorAll('.menu-item:checked');
  menuItems.forEach(item => {
    const itemName = item.parentNode.textContent.trim();
    const price = parseFloat(item.dataset.price);
    const quantity = parseInt(item.nextElementSibling.value);

    if (!isNaN(price) && !isNaN(quantity) && quantity > 0) {
      orderedItems.push({
        name: itemName,
        price: price,
        quantity: quantity
      });
    }
  });

  // Calculate total and commission
  const totalValue = parseFloat($("#total").text());
  const commission = parseFloat($("#commission").text());
  const discount = parseFloat($("#discount").val());

  // Prepare data for API submission
  const formData = {
    "Employee Name": employeeName,
    "Total": totalValue.toFixed(2),
    "Commission": commission.toFixed(2),
    "Items Ordered": JSON.stringify(orderedItems),
    "Discount Applied": discount
  };

  // Form Submission Logic for Spreadsheet
  $.ajax({
    url: "https://api.apispreadsheets.com/data/M8Fcdq9DkXUm5XUM/",
    type: "post",
    data: formData,
    headers: {
      accessKey: "d7d32bdf84ab4c0e2984d71cb61cd4ba",
      secretKey: "b1a8cbc589e1a2414e0e2421722c1c31",
      "Content-Type": "application/x-www-form-urlencoded",
    },
    success: function () {
      alert("Form Data Submitted to Spreadsheet and Discord :)");
      resetForm();
    },
    error: function () {
      alert("There was an error :(");
    }
  });

  // Prepare data for Discord webhook
  const discordData = {
    username: "Receipts",
    content: `New order submitted by ${employeeName}`,
    embeds: [{
      title: "Order Details",
      fields: [
        { name: "Employee Name", value: employeeName, inline: true },
        { name: "Total", value: `$${totalValue.toFixed(2)}`, inline: true },
        { name: "Commission", value: `$${commission.toFixed(2)}`, inline: true },
        { name: "Discount Applied", value: `${discount}%`, inline: true },
        { name: "Items Ordered", value: orderedItems.map(item => `${item.quantity}x ${item.name}`).join('\n') }
      ],
      color: 0x00ff00 // You can customize the color
    }]
  };

  // Form Submission Logic for Discord webhook
  $.ajax({
    url: "https://discord.com/api/webhooks/1214334288984670228/ooI9nFhBHxj478qoBMhkGH03XEAc0lWMduPMqHmhAEBFi6X35qIzwqLv643ahFTA-Jyt", // Replace with your Discord webhook URL
    type: "post",
    contentType: "application/json",
    data: JSON.stringify(discordData),
    success: function () {
      // Do nothing specific for Discord success
    },
    error: function () {
      console.error("Error sending data to Discord :(");
    }
  });

  // Reset checkboxes and quantity inputs
  $('.menu-item').prop('checked', false);
  $('.quantity').val(1);

  // Reset totals
  document.getElementById('total').innerText = '';
  document.getElementById('commission').innerText = '';
  // Reset discount dropdown to default
  $("#discount").val("0");
}

    function resetForm() {
      // Reset checkboxes and quantity inputs
      $('.menu-item').prop('checked', false);
      $('.quantity').val(1);

      // Reset totals
      document.getElementById('total').innerText = '';
      document.getElementById('commission').innerText = '';
      // Reset discount dropdown to default
      $("#discount").val("0");
    }
  </script>
</head>
<body>

<h2>Burgershot</h2>

  <form id="menuForm">
  <h3>Meals</h3>
    <label>
      <input type="checkbox" class="menu-item" data-price="500"> The Bleeder Bag #1 - $500
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="500"> Double Shot Bag #2 - $500
      <input type="number" class="quantity" value="1" min="1">
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="500"> Heart Stopper Bag #3 - $500
      <input type="number" class="quantity" value="1" min="1">
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="500"> Simply Burger Bag #5 - $500
      <input type="number" class="quantity" value="1" min="1">
    </label>
	    <label>
      <input type="checkbox" class="menu-item" data-price="500"> The Prickly Bag #6 - $500
      <input type="number" class="quantity" value="1" min="1">
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="3000"> Heart Stopper Crate(x5) #4 - $3000
      <input type="number" class="quantity" value="1" min="1">
    </label>
	    <label>
      <input type="checkbox" class="menu-item" data-price="1250"> Sweet Tooth Crate(x5) #7 - $1250
      <input type="number" class="quantity" value="1" min="1">
    </label>
		    <label>
      <input type="checkbox" class="menu-item" data-price="2500"> Expresso Crate (x30) #8 - $2500
      <input type="number" class="quantity" value="1" min="1">
    </label>
	
	
	<h3>Single Items</h3>
 <label>
      <input type="checkbox" class="menu-item" data-price="15000"> The Bleeder Bag #1 - $15,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="35000"> Double Shot Bag #2 - $35,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="30000"> Heart Stopper Bag #3 - $30,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
 <label>
      <input type="checkbox" class="menu-item" data-price="15000"> Simply Burger Bag #5 - $15,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="35000"> The Prickly Bag #6 - $35,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="30000"> Heart Stopper Crate(x5) - $30,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
 <label>
      <input type="checkbox" class="menu-item" data-price="15000"> Sweet Tooth Crate(x5) #7 - $15,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="35000"> Expresso Crate (x30) #8 - $35,000
      <input type="number" class="quantity" value="1" min="1">
    </label>

    
	

 

	
	
	
	
	
	<div style="margin-bottom: 30px;"></div>
	
	<label for="discount">Select Discount:</label>
    <select id="discount" onchange="calculateTotals()">
      <option value="0">No Discount</option>
      <option value="30">30% Discount PD/EMS</option>

    </select>
	
	<div style="margin-bottom: 30px;"></div>
	
		<label for="customerName">Customer Name:</label>
		<input type="text" id="customerName" required>
	<div style="margin-bottom: 30px;"></div>
	
	<div style="margin-bottom: 30px;"></div>
	


    <label for="employeeName">Employee Name:</label>
    <input type="text" id="employeeName" required>
	
	<div style="margin-bottom: 30px;"></div>
	
	

    <p>Total: $<span id="total"></span></p>
    <p>Commission (20%): $<span id="commission"></span></p>
	
	<div style="margin-bottom: 30px;"></div>

    <button type="button" onclick="calculateTotals()">Calculate</button>
    <button type="button" onclick="SubForm()">Submit</button>
    <button type="button" onclick="resetForm()">Reset</button>
  </form>

</body>
</html>
