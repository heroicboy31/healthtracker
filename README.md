<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Colorful Calorie Tracker</title>
<style>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    max-width: 650px;
    margin: 40px auto;
    padding: 25px;
    background: linear-gradient(135deg, #6a11cb 0%, #2575fc 100%);
    border-radius: 12px;
    color: #fff;
    box-shadow: 0 8px 24px rgba(37,117,252,0.4);
  }

  h1, h2 {
    text-align: center;
    margin-bottom: 15px;
    text-shadow: 2px 2px 6px rgba(0,0,0,0.3);
  }

  label {
    display: block;
    margin-top: 18px;
    font-weight: 600;
    text-shadow: 1px 1px 2px rgba(0,0,0,0.25);
  }

  input[type="text"],
  input[type="number"] {
    width: 100%;
    padding: 12px 14px;
    margin-top: 7px;
    border: none;
    border-radius: 8px;
    font-size: 16px;
    box-sizing: border-box;
    box-shadow: inset 0 2px 6px rgba(0,0,0,0.2);
    transition: box-shadow 0.3s ease;
    color: #333;
  }

  input[type="text"]:focus,
  input[type="number"]:focus {
    outline: none;
    box-shadow: 0 0 10px #ffb347;
  }

  button {
    margin-top: 22px;
    width: 100%;
    padding: 14px;
    background: linear-gradient(45deg, #ff6a00, #ee0979);
    border: none;
    border-radius: 12px;
    color: white;
    font-size: 18px;
    font-weight: 700;
    cursor: pointer;
    box-shadow: 0 4px 15px rgba(238,9,121,0.6);
    transition: background 0.3s ease, box-shadow 0.3s ease;
  }

  button:hover {
    background: linear-gradient(45deg, #ee0979, #ff6a00);
    box-shadow: 0 6px 20px rgba(255,106,0,0.8);
  }

  .reset-btn {
    background: linear-gradient(45deg, #ff3c3c, #ff6161);
    box-shadow: 0 4px 15px rgba(255,99,99,0.6);
  }

  .reset-btn:hover {
    background: linear-gradient(45deg, #ff6161, #ff3c3c);
    box-shadow: 0 6px 20px rgba(255,99,99,0.9);
  }

  .summary {
    margin-top: 38px;
    background: rgba(255, 255, 255, 0.15);
    padding: 22px 28px;
    border-radius: 14px;
    box-shadow: inset 0 0 12px rgba(255,255,255,0.25);
    font-size: 18px;
    font-weight: 600;
  }

  .summary h3 {
    margin-top: 0;
    margin-bottom: 12px;
    text-align: center;
    text-shadow: 1px 1px 3px rgba(0,0,0,0.3);
  }

  #overeatWarning {
    display: none;
    color: #ff1e1e;
    font-size: 56px;
    font-weight: 900;
    text-align: center;
    margin-top: 40px;
    animation: flash 1s infinite;
    user-select: none;
    text-shadow:
      0 0 10px #ff1e1e,
      0 0 20px #ff1e1e,
      0 0 30px #ff1e1e,
      0 0 40px #ff6a6a,
      0 0 50px #ff6a6a,
      0 0 60px #ff6a6a;
  }

  @keyframes flash {
    0%, 100% { opacity: 1; }
    50% { opacity: 0; }
  }

  .error {
    color: #ff6161;
    margin-top: 14px;
    text-align: center;
    font-weight: 700;
    text-shadow: 1px 1px 2px rgba(0,0,0,0.2);
  }
</style>
</head>
<body>

<h1>Calorie & Macro Tracker</h1>

<h2>Your Daily Targets</h2>
<label for="targetCalories">Target Calories (kcal):</label>
<input type="number" id="targetCalories" value="0" />

<label for="targetProtein">Target Protein (g):</label>
<input type="number" id="targetProtein" value="0" />

<label for="targetCarbs">Target Carbs (g):</label>
<input type="number" id="targetCarbs" value="0" />

<label for="targetFat">Target Fat (g):</label>
<input type="number" id="targetFat" value="0" />

<hr style="border-color: rgba(255,255,255,0.3); margin-top: 30px; margin-bottom: 30px;" />

<h2>Add Food Item</h2>
<label for="foodName">Food Name & Quantity (e.g. "1 slice pizza"):</label>
<input type="text" id="foodName" placeholder="Type food name & quantity" />
<button onclick="addFood()">Add Food</button>

<h2>Add Liquid Item</h2>
<label for="liquidName">Liquid Name & Quantity (e.g. "2 cups milk"):</label>
<input type="text" id="liquidName" placeholder="Type liquid & quantity" />
<button onclick="addLiquid()">Add Liquid</button>

<button class="reset-btn" onclick="resetAll()">Reset All</button>

<div id="error" class="error"></div>

<div class="summary" id="summary">
  <h3>Today's Intake:</h3>
  <p>Calories: 0 kcal</p>
  <p>Protein: 0 g</p>
  <p>Carbs: 0 g</p>
  <p>Fat: 0 g</p>
</div>

<div id="overeatWarning">OVEREATING!</div>

<script>
  // **IMPORTANT: Replace these with your own Nutritionix API credentials!**
  const APP_ID = "6ca695d9";  // <-- Put your Nutritionix APP ID here
  const API_KEY = 
"707c9e389f565ca7143a435a225ab7cd"; // <-- Put your Nutritionix API KEY here

  let totals = {
    calories: 0,
    protein: 0,
    carbs: 0,
    fat: 0
  };

  function updateSummary() {
    const targetCalories = +document.getElementById('targetCalories').value || 0;
    const targetProtein = +document.getElementById('targetProtein').value || 0;
    const targetCarbs = +document.getElementById('targetCarbs').value || 0;
    const targetFat = +document.getElementById('targetFat').value || 0;

    const summaryDiv = document.getElementById('summary');
    summaryDiv.innerHTML = `
      <h3>Today's Intake:</h3>
      <p>Calories: ${totals.calories.toFixed(1)} kcal / ${targetCalories} kcal</p>
      <p>Protein: ${totals.protein.toFixed(1)} g / ${targetProtein} g</p>
      <p>Carbs: ${totals.carbs.toFixed(1)} g / ${targetCarbs} g</p>
      <p>Fat: ${totals.fat.toFixed(1)} g / ${targetFat} g</p>
    `;

    checkOvereat(targetCalories);
  }

  function checkOvereat(targetCalories) {
    const overeatDiv = document.getElementById('overeatWarning');
    if (totals.calories > targetCalories && targetCalories > 0) {
      overeatDiv.style.display = "block";
    } else {
      overeatDiv.style.display = "none";
    }
  }

  async function addFood() {
    const errorDiv = document.getElementById('error');
    errorDiv.textContent = '';
    const foodInput = document.getElementById('foodName').value.trim();
    if (!foodInput) {
      errorDiv.textContent = "Please enter a food name and quantity.";
      return;
    }

    const url = "https://trackapi.nutritionix.com/v2/natural/nutrients";

    try {
      const response = await fetch(url, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "x-app-id": APP_ID,
          "x-app-key": API_KEY,
          "x-remote-user-id": "0"
        },
        body: JSON.stringify({ query: foodInput })
      });

      if (!response.ok) {
        throw new Error(`API error: ${response.status}`);
      }

      const data = await response.json();

      if (!data.foods || data.foods.length === 0) {
        errorDiv.textContent = "No nutrition data found for that food.";
        return;
      }

      data.foods.forEach(food => {
        totals.calories += food.nf_calories || 0;
        totals.protein += food.nf_protein || 0;
        totals.carbs += food.nf_total_carbohydrate || 0;
        totals.fat += food.nf_total_fat || 0;
      });

      document.getElementById('foodName').value = '';
      updateSummary();
    } catch (err) {
      errorDiv.textContent = "Error fetching nutrition data. Try again.";
      console.error(err);
    }
  }

  async function addLiquid() {
    const liquidInput = document.getElementById('liquidName').value.trim();
    if (!liquidInput) {
      document.getElementById('error').textContent = "Please enter a liquid name and quantity.";
      return;
    }
    // Reuse addFood by copying liquid input
    document.getElementById('foodName').value = liquidInput;
    await addFood();
    document.getElementById('liquidName').value = '';
  }

  function resetAll() {
    totals = { calories: 0, protein: 0, carbs: 0, fat: 0 };
    document.getElementById('foodName').value = '';
    document.getElementById('liquidName').value = '';
    document.getElementById('targetCalories').value = 2000;
    document.getElementById('targetProtein').value = 150;
    document.getElementById('targetCarbs').value = 250;
    document.getElementById('targetFat').value = 70;
    document.getElementById('error').textContent = '';
    updateSummary();
  }

  // Initialize summary on page load
  updateSummary();
</script>

</body>
</html>
