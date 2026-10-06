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
                 │ _id, id, name        │
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
  "id": 1,
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
mkdir category-api
cd category-api

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
const Category = require("../models/Category");

const router = express.Router();

// GET all categories
router.get("/", async (req, res) => {
  try {
    const categories = await Category.find().sort({ id: 1 });

    res.json(categories);
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
    const category = await Category.findOne({
      id: Number(req.params.id)
    });

    if (!category) {
      return res.status(404).json({
        message: "Category not found"
      });
    }

    res.json(category);
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
    const { id, name } = req.body;

    if (!id || !name) {
      return res.status(400).json({
        message: "ID and name are required"
      });
    }

    const existing = await Category.findOne({
      id: Number(id)
    });

    if (existing) {
      return res.status(400).json({
        message: "Category ID already exists"
      });
    }

    const category = await Category.create({
      id: Number(id),
      name: name.trim()
    });

    res.status(201).json(category);

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
    const { name } = req.body;

    const category = await Category.findOneAndUpdate(
      {
        id: Number(req.params.id)
      },
      {
        name: name.trim()
      },
      {
        new: true
      }
    );

    if (!category) {
      return res.status(404).json({
        message: "Category not found"
      });
    }

    res.json(category);

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
    const category = await Category.findOneAndDelete({
      id: Number(req.params.id)
    });

    if (!category) {
      return res.status(404).json({
        message: "Category not found"
      });
    }

    res.json({
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
    id: {
      type: Number,
      required: true,
      unique: true
    },

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

This is the proper structure before connecting your **Vite + MUI + SweetAlert2** frontend.


### Category page

```jsx
import { useEffect, useState } from "react";
import Swal from "sweetalert2";

import {
  getCategories,
  createCategory,
  updateCategory,
  deleteCategory
} from "../services/categoryService";

function Categories() {
  const [categories, setCategories] = useState([]);
  const [id, setId] = useState("");
  const [name, setName] = useState("");
  const [editing, setEditing] = useState(false);

  const loadCategories = async () => {
    try {
      const response = await getCategories();
      setCategories(response.data);
    } catch (error) {
      Swal.fire("Error", "Unable to load categories", "error");
    }
  };

  useEffect(() => {
    loadCategories();
  }, []);

  const saveCategory = async (e) => {
    e.preventDefault();

    if (!id || !name.trim()) {
      Swal.fire("Warning", "ID and Name are required", "warning");
      return;
    }

    try {
      if (editing) {
        await updateCategory(id, {
          id: Number(id),
          name: name.trim()
        });

        Swal.fire("Updated!", "Category updated successfully", "success");
      } else {
        await createCategory({
          id: Number(id),
          name: name.trim()
        });

        Swal.fire("Created!", "Category created successfully", "success");
      }

      resetForm();
      loadCategories();

    } catch (error) {
      Swal.fire(
        "Error",
        error.response?.data?.message || "Operation failed",
        "error"
      );
    }
  };

  const editCategory = (category) => {
    setId(category.id);
    setName(category.name);
    setEditing(true);
  };

  const removeCategory = async (categoryId) => {
    const result = await Swal.fire({
      title: "Are you sure?",
      text: "This category will be deleted.",
      icon: "warning",
      showCancelButton: true,
      confirmButtonText: "Yes, delete it!",
      cancelButtonText: "Cancel"
    });

    if (!result.isConfirmed) {
      return;
    }

    try {
      await deleteCategory(categoryId);

      Swal.fire(
        "Deleted!",
        "Category deleted successfully.",
        "success"
      );

      loadCategories();

    } catch (error) {
      Swal.fire("Error", "Unable to delete category", "error");
    }
  };

  const resetForm = () => {
    setId("");
    setName("");
    setEditing(false);
  };

  return (
    <div className="container mt-4">

      <h2>Category Management</h2>

      <form onSubmit={saveCategory}>

        <div>
          <label>ID</label>

          <input
            type="number"
            value={id}
            onChange={(e) => setId(e.target.value)}
            disabled={editing}
          />
        </div>

        <div>
          <label>Name</label>

          <input
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
            placeholder="Category name"
          />
        </div>

        <button type="submit">
          {editing ? "Update" : "Save"}
        </button>

        {editing && (
          <button
            type="button"
            onClick={resetForm}
          >
            Cancel
          </button>
        )}

      </form>

      <hr />

      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Actions</th>
          </tr>
        </thead>

        <tbody>
          {categories.map((category) => (
            <tr key={category.id}>

              <td>{category.id}</td>

              <td>{category.name}</td>

              <td>
                <button
                  onClick={() => editCategory(category)}
                >
                  Edit
                </button>

                <button
                  onClick={() => removeCategory(category.id)}
                >
                  Delete
                </button>
              </td>

            </tr>
          ))}
        </tbody>
      </table>

    </div>
  );
}

export default Categories;
```

---




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
