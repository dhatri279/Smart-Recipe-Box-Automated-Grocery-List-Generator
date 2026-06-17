# Smart Recipe Box & Grocery Aggregator API

This is a clean, fast backend API built with **FastAPI** and **MongoDB** designed to manage recipes and dynamically generate combined grocery shopping lists. 

Instead of just doing basic database storage, this project handles real-world scaling issues: it automatically converts different ingredient units (like tablespoons into cups) so your list stays neat, calculates basic nutritional estimations on the fly, flags common food allergens, and has a built-in safety net that keeps the server running beautifully even if your database suddenly goes offline.

---

## Key Features

* **Full Recipe CRUD:** Easily create, view, update, and delete recipes using clean, nested data structures.
* **Smart Shopping Lists:** Pass in multiple recipe IDs, and the app instantly merges them together—adding up duplicate ingredients so you don't buy the same thing twice.
* **Smart Unit Conversion:** Built-in logic standardizes sub-units automatically (e.g., if one recipe asks for `4 tbsp` of milk and another asks for `0.5 cups`, it recognizes they match, does the math, and outputs `0.75 cups`).
* **Calorie & Allergen Tracker:** Scans ingredient names during aggregation to calculate a rough calorie total and automatically flag dietary alerts like `gluten`, `dairy`, or `poultry`.
* **Saved History Ledger:** Every time a shopping list is generated, it is permanently logged in a dedicated `shopping_lists` database collection with its own tracking ID and timestamp so you can look it up later.
* **Database Crash Protection:** Features custom global middleware that acts as a safety guardrail. If MongoDB goes down, the web server doesn't freeze or crash—it intercepts the error instantly and returns a friendly, clear message.

---

## How the Project is Structured

* **Framework:** FastAPI (Python)
* **Database:** MongoDB Community Edition (via PyMongo)
* **Data Validation:** Pydantic (keeps inputs clean and strictly typed)
* **Server:** Uvicorn

```text
recipe_box_api/
│
├── config.py          # Database connection strings & fast network timeout settings
├── models.py          # Pydantic schemas protecting the API from bad input data
├── engine.py          # Core logic for aggregation, unit conversions, and health checks
├── main.py            # API routes, endpoints, and the global error-handler middleware
└── README.md          # Project documentation
```

---

## Setup & Installation

### 1. Prerequisites
Make sure you have these installed on your machine:
* **Python (v3.10 or higher)**
* **MongoDB Community Server** (running locally on port `27017`)
* **Postman** (to test the endpoints)

### 2. Install Dependencies
Open your terminal inside the project directory and run:
```bash
pip install fastapi uvicorn pymongo
```

### 3. Start MongoDB
* **Windows:** Press `Win + R`, type `services.msc`, find **MongoDB Server**, right-click it, and choose **Start**.
* **macOS:** Run `brew services start mongodb-community` in your terminal.

### 4. Run the API Server
Start the development server by running this command where your `main.py` is located:
```bash
uvicorn main:app --reload --port 8000
```
Your API will now be live at: **`http://127.0.0.1:8000`**

---

## API Testing Playbook (Postman)

### 1. Save a New Recipe
* **Method:** `POST`
* **URL:** `http://127.0.0.1:8000/recipes`
* **Body (raw -> JSON):**
```json
{
  "title": "Breakfast Omelet",
  "servings": 1,
  "ingredients": [
    {"name": "eggs", "quantity": 3.0, "unit": "pieces", "category": "Dairy"},
    {"name": "milk", "quantity": 4.0, "unit": "tbsp", "category": "Dairy"},
    {"name": "onions", "quantity": 0.5, "unit": "cups", "category": "Produce"}
  ]
}
```

### 2. Get All Recipes
* **Method:** `GET`
* **URL:** `http://127.0.0.1:8000/recipes`
* **Description:** Lists every recipe currently saved in your database.

### 3. Update a Recipe
* **Method:** `PUT`
* **URL:** `http://127.0.0.1:8000/recipes/PASTE_YOUR_RECIPE_ID_HERE`
* **Description:** Swaps out or modifies the details of an existing recipe using its MongoDB ID.

### 4. Delete a Recipe
* **Method:** `DELETE`
* **URL:** `http://127.0.0.1:8000/recipes/PASTE_YOUR_RECIPE_ID_HERE`
* **Description:** Permanently removes a recipe from the collection.

### 5. Generate a Shopping List
* **Method:** `POST`
* **URL:** `http://127.0.0.1:8000/recipes/shopping-list`
* **Body (Array of Recipe IDs):**
```json
[
  "6a3033d75c2cfe48eeff3a88",
  "6a3033d75c2cfe48eeff3a88"
]
```
* **What you get back:**
```json
{
  "shopping_list_id": "6a3045f89c2bfe48eeffb987",
  "saved_at": "2026-06-17 15:35:10 UTC",
  "nutritional_summary": {
    "total_calories_estimate": "460.0 kcal",
    "dietary_alerts": ["poultry", "dairy"]
  },
  "categories": {
    "Dairy": [
      {"name": "eggs", "quantity": 6.0, "unit": "pieces", "category": "Dairy"},
      {"name": "milk", "quantity": 0.5, "unit": "cups", "category": "Dairy"}
    ],
    "Produce": [
      {"name": "onions", "quantity": 1.0, "unit": "cups", "category": "Produce"}
    ]
  }
}
```

---

## Testing the App's Resilience (The "Wow Factor" Demo)

To show off how stable this app is during an evaluation:
1. Make sure your FastAPI server and MongoDB are both up and running normally.
2. Open your **Windows Services panel** (`services.msc`) and completely **Stop** the **MongoDB Server** service.
3. Go to Postman and try to send a `GET /recipes` request.
4. **The Result:** Instead of freezing, spinning forever, or crashing with an ugly console trace, the custom middleware catches the database drop instantly. Your API stays perfectly online and returns a clean, professional error response status code (`503`):

```json
{
  "status": "Offline",
  "error": "Database Service Temporarily Unavailable",
  "details": "The backend middleware safely intercepted a database connection loss. FastAPI remains online."
}
```
