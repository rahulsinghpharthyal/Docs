# 🥗 AI Food Scan Backend – README

## 📌 Overview

This backend provides an **AI-powered food scanning system** that allows users to upload a food image and receive:

* Identified food items
* Nutritional information
* Total calorie calculation
* Cached responses for identical images (cost optimized)

The system is designed to be:

* ⚡ Fast
* 💸 Cost-efficient
* 📈 Scalable
* 🧠 AI-assisted (Gemini Vision + Nutrition)

---

## 🧱 Tech Stack

* **Node.js / Express**
* **MongoDB + Mongoose**
* **Multer (Memory Storage for images)**
* **Google Gemini AI (Vision + Text)**
* **JWT Authentication**
* **Clean Architecture (Controller / Service separation)**

---

## 🔐 Authentication

All APIs require authentication.

The backend expects:

```http
Authorization: Bearer <JWT_TOKEN>
```

After authentication, the backend has access to:

```js
req.user.userId
```

---

## 📸 Scan Food API

### Endpoint

```http
POST /api/scan-food
```

### Headers

```http
Authorization: Bearer <JWT_TOKEN>
Content-Type: multipart/form-data
```

### Body (form-data)

| Key   | Type | Required | Description               |
| ----- | ---- | -------- | ------------------------- |
| image | File | ✅ Yes    | Food image (jpg/png/webp) |

⚠️ **Do NOT send base64 from frontend**
The backend handles conversion internally.

---

## 🔄 What Happens Internally (Step-by-Step)

### 1️⃣ Image Upload (Frontend → Backend)

* Frontend sends the **original image file**
* Multer stores the image in **memory (RAM)**
* No disk storage → faster and safer

---

### 2️⃣ Image Hashing (Cost Optimization)

```js
SHA-256(imageBase64)
```

* A unique hash is generated for each image
* Used to detect **duplicate scans**

---

### 3️⃣ Cache Check (MongoDB)

Backend checks:

```js
ScannedMeal.findOne({ imageHash })
```

#### ✅ If image was scanned before:

* No AI call
* Data returned instantly from DB
* **ZERO AI cost**

#### ❌ If image is new:

* Continue to AI processing

---

### 4️⃣ AI Vision (Gemini)

Gemini Vision:

* Identifies food items in the image
* Returns:

```json
{
  "items": ["Rice", "Chicken Curry"],
  "summary": "A plate of rice with chicken curry"
}
```

---

### 5️⃣ Food Database Lookup (More Cost Optimization)

For each detected food:

```js
FoodItem.findOne({ name })
```

#### If food exists:

* Reuse stored nutrition data
* No AI nutrition call

#### If food does NOT exist:

* Call Gemini Nutrition API
* Save result to database
* Mark as:

```js
createdBy: "scan",
isVerified: false
```

---

### 6️⃣ Calorie Calculation

Total calories are calculated by summing all detected food items.

---

### 7️⃣ Cache the Result

The final scan result is saved:

```js
ScannedMeal.create({
  imageHash,
  items,
  summary,
  totalCalories
})
```

This ensures:

* Future identical images are served from cache
* No repeated AI usage

---

### 8️⃣ Response Sent to Frontend

```json
{
  "success": true,
  "source": "ai | cache",
  "summary": "A plate of rice with chicken curry",
  "totalCalories": 620,
  "items": [
    {
      "name": "Rice",
      "calories": 200,
      "protein": 4,
      "carbs": 45,
      "fat": 1
    }
  ]
}
```

---

## 🧠 Cost Optimization Strategy (Client Explanation)

### ✅ 1. Image Hash Caching

* Same image → same hash
* Same hash → no AI call
* Massive reduction in Vision API usage

### ✅ 2. Food-Level Caching

* Nutrition for each food is stored once
* Reused across all users and scans

### ✅ 3. Memory Upload (No Disk I/O)

* Faster processing
* Lower server cost
* Automatic cleanup

### ✅ 4. AI Called Only When Necessary

* Vision AI → only for new images
* Nutrition AI → only for new food items

📉 **Result:**
AI costs scale **sub-linearly**, not per request.

---

## 🗂 Architecture Overview

```
Controller
 ├── Request validation
 ├── Image conversion
 ├── AI calls
 └── Response handling

Service Layer
 ├── FoodItem DB queries
 └── ScannedMeal DB queries

Utilities
 └── Gemini AI integration
```

This separation ensures:

* Clean code
* Easy testing
* Long-term maintainability

---

## 🧑‍💻 Notes for Frontend Developers

* Send image as **multipart/form-data**
* Do not convert image to base64
* Handle both `source: "ai"` and `source: "cache"`
* API response is consistent in both cases

---

## 🚀 Future Enhancements

* Auto-add scanned food to User Diet Diary
* Admin verification of AI-added food items
* Per-user scan history
* Cloud image storage (optional)

---

## ✅ Summary

This system:

* Uses AI intelligently
* Avoids unnecessary costs
* Is scalable for thousands of users
* Is easy to integrate with frontend
* Is client-friendly and explainable

---
