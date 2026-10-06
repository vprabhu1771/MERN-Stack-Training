Yes. You can build a **Category CRUD system** using:

* **MongoDB** — database
* **Express.js + Node.js** — REST API
* **React + Vite** — web frontend
* **SweetAlert2** — success/delete confirmations
* **.NET MAUI** — mobile/desktop client consuming the same API

### Architecture

```text
                 ┌──────────────────────┐
                 │      MongoDB         │
                 │ categories           │
                 │ _id, name            │
                 └──────────▲───────────┘
                            │
                     REST API / JSON
                            │
                 ┌──────────┴───────────┐
                 │ Express + Node.js    │
                 │ /api/categories      │
                 └───────▲───────▲──────┘
                         │       │
             ┌───────────┘       └────────────┐
             │                                │
   ┌─────────┴─────────┐             ┌────────┴─────────┐
   │ React + Vite      │             │ .NET MAUI        │
   │ SweetAlert2       │             │ HttpClient       │
   │ Category CRUD     │             │ Category CRUD     │
   └───────────────────┘             └──────────────────┘
```

## 1. MongoDB Category

I recommend keeping MongoDB's `_id` and your own numeric `id` separate:

```json
{
  "_id": "68f123...",  
  "name": "Electronics"
}
```

API endpoints:

```text
GET    /api/categories
GET    /api/categories/:id
POST   /api/categories
PUT    /api/categories/:id
DELETE /api/categories/:id
```

## 2. Express API

Project structure:

```text
category-api/
│
├── models/
│   └── Category.js
│
├── routes/
│   └── categoryRoutes.js
│
├── controllers/
│   └── categoryController.js
│
├── server.js
├── package.json
└── .env
```

### Install

```bash
mkdir api
cd api

npm init -y

npm install express mongoose cors dotenv
npm install -D nodemon
```

### `models/Category.js`

```javascript
const mongoose = require("mongoose");

const categorySchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
      trim: true
    }
  },
  {
    timestamps: true
  }
);

module.exports = mongoose.model("Category", categorySchema);
```

### `server.js`

```javascript
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
require("dotenv").config();

const categoryRoutes = require("./routes/categoryRoutes");

const app = express();

app.use(cors());
app.use(express.json());

mongoose
  .connect(process.env.MONGO_URI)
  .then(() => {
    console.log("MongoDB Connected");
  })
  .catch((error) => {
    console.error("MongoDB Error:", error);
  });

app.use("/api/categories", categoryRoutes);

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### `.env`

```env
MONGO_URI=mongodb://127.0.0.1:27017/categorydb
PORT=5000
```

---

You have put the **`server.js` code inside `routes/categoryRoutes.js`**. These should be two separate files.

Use this structure:

```text
category-api/
├── models/
│   └── Category.js
├── routes/
│   └── categoryRoutes.js
├── server.js
├── package.json
└── .env
```

### 1. `server.js`

Move your code to `server.js`:

```javascript
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
require("dotenv").config();

const categoryRoutes = require("./routes/categoryRoutes");

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// MongoDB
mongoose
  .connect(process.env.MONGO_URI)
  .then(() => {
    console.log("MongoDB Connected");
  })
  .catch((error) => {
    console.error("MongoDB Error:", error);
  });

// Routes
app.use("/api/categories", categoryRoutes);

// Server
const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

### 2. `routes/categoryRoutes.js`

This file should contain the **category API routes**:

```javascript
const express = require("express");

const router = express.Router();

// Test
router.get("/", (req, res) => {
  res.json({
    message: "Category API working"
  });
});

module.exports = router;
```

Now start:

```bash
node server.js
```

Open:

```text
http://localhost:5000/api/categories
```

You should get:

```json
{
  "message": "Category API working"
}
```

### 3. Then add MongoDB CRUD

Once the route is working, your `categoryRoutes.js` can become:

```javascript
const express = require("express");
const mongoose = require("mongoose");
const Category = require("../models/Category");

const router = express.Router();

// GET all categories
router.get("/", async (req, res) => {
  try {
    const categories = await Category.find().sort({ createdAt: -1 });

    res.status(200).json(categories);
  } catch (error) {
    res.status(500).json({
      message: "Failed to fetch categories",
      error: error.message
    });
  }
});

// GET single category
router.get("/:id", async (req, res) => {
  try {
    const { id } = req.params;

    if (!mongoose.Types.ObjectId.isValid(id)) {
      return res.status(400).json({
        message: "Invalid category ID"
      });
    }

    const category = await Category.findById(id);

    if (!category) {
      return res.status(404).json({
        message: "Category not found"
      });
    }

    res.status(200).json(category);
  } catch (error) {
    res.status(500).json({
      message: "Failed to fetch category",
      error: error.message
    });
  }
});

// CREATE category
router.post("/", async (req, res) => {
  try {
    const { name } = req.body;

    if (!name || !name.trim()) {
      return res.status(400).json({
        message: "Category name is required"
      });
    }

    const existing = await Category.findOne({
      name: name.trim()
    });

    if (existing) {
      return res.status(409).json({
        message: "Category already exists"
      });
    }

    const category = await Category.create({
      name: name.trim()
    });

    res.status(201).json({
      message: "Category created successfully",
      category
    });
  } catch (error) {
    res.status(500).json({
      message: "Failed to create category",
      error: error.message
    });
  }
});

// UPDATE category
router.put("/:id", async (req, res) => {
  try {
    const { id } = req.params;
    const { name } = req.body;

    if (!mongoose.Types.ObjectId.isValid(id)) {
      return res.status(400).json({
        message: "Invalid category ID"
      });
    }

    if (!name || !name.trim()) {
      return res.status(400).json({
        message: "Category name is required"
      });
    }

    const existing = await Category.findOne({
      name: name.trim(),
      _id: { $ne: id }
    });

    if (existing) {
      return res.status(409).json({
        message: "Category name already exists"
      });
    }

    const category = await Category.findByIdAndUpdate(
      id,
      {
        name: name.trim()
      },
      {
        new: true,
        runValidators: true
      }
    );

    if (!category) {
      return res.status(404).json({
        message: "Category not found"
      });
    }

    res.status(200).json({
      message: "Category updated successfully",
      category
    });
  } catch (error) {
    res.status(500).json({
      message: "Failed to update category",
      error: error.message
    });
  }
});

// DELETE category
router.delete("/:id", async (req, res) => {
  try {
    const { id } = req.params;

    if (!mongoose.Types.ObjectId.isValid(id)) {
      return res.status(400).json({
        message: "Invalid category ID"
      });
    }

    const category = await Category.findByIdAndDelete(id);

    if (!category) {
      return res.status(404).json({
        message: "Category not found"
      });
    }

    res.status(200).json({
      message: "Category deleted successfully"
    });
  } catch (error) {
    res.status(500).json({
      message: "Failed to delete category",
      error: error.message
    });
  }
});

module.exports = router;
```

And `models/Category.js`:

```javascript
const mongoose = require("mongoose");

const categorySchema = new mongoose.Schema(
  {    
    name: {
      type: String,
      required: true,
      trim: true
    }
  },
  {
    timestamps: true
  }
);

module.exports = mongoose.model("Category", categorySchema);
```

So the important correction is:

```text
server.js
    ↓
app.use("/api/categories", categoryRoutes)
    ↓
routes/categoryRoutes.js
    ↓
models/Category.js
    ↓
MongoDB
```

Replace `192.168.1.100` with the IP address of your Node.js computer when testing on a physical phone.

---

## Recommended final structure

```text
CategorySystem/
│
├── backend/
│   ├── models/
│   │   └── Category.js
│   ├── routes/
│   │   └── categoryRoutes.js
│   ├── controllers/
│   │   └── categoryController.js
│   ├── server.js
│   └── .env
│
├── web/
│   └── React Vite
│       ├── components/
│       ├── pages/
│       └── services/
│
└── mobile/
    └── CategoryApp/
        ├── Models/
        ├── Services/
        └── Pages/
```

This gives you **one MongoDB + one REST API**, with both **React/Vite** and **.NET MAUI** clients using it.

If this is for your POS system, I would extend the `categories` table/model next with **`id, name, status, createdAt, updatedAt`**, then build the same CRUD pattern for **Products, Customers, Suppliers, Units and Brands**.
