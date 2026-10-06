## MUI (Material UI)
# 3. React + Vite

Create the frontend:

```bash
npm create vite@latest category-web -- --template react

cd category-web

npm install
npm install axios sweetalert2
npm run dev
```

Suggested structure:

```text
category-web/
│
├── src/
│   ├── components/
│   │   └── CategoryForm.jsx
│   │
│   ├── pages/
│   │   └── Categories.jsx
│   │
│   ├── services/
│   │   └── categoryService.js
│   │
│   ├── App.jsx
│   └── main.jsx
```

Yes — you mean **MUI (Material UI)** for the React + Vite frontend.

For your **MERN Categories CRUD**, I recommend:

```text
MongoDB
   ↓
Express + Node.js API
   ↓
React + Vite
   ↓
MUI (Material UI)
   ↓
SweetAlert2
```

Install MUI:

```bash
npm install @mui/material @emotion/react @emotion/styled
npm install @mui/icons-material
npm install axios sweetalert2
```

Then the Categories screen can use:

* `TextField` → ID / Name
* `Button` → Save / Update / Cancel
* `Table` → category list
* `IconButton` → Edit / Delete
* `Paper` / `Card` → clean layout
* `Dialog` → add/edit category
* `Alert` / SweetAlert2 → confirmation and success messages
* `CircularProgress` → loading

## Category Service

`src\services\categoryService.jsx`
```jsx
import axios from "axios";

const API_URL = "http://localhost:5000/api/categories";

export const getCategories = () => {
  return axios.get(API_URL);
};

export const createCategory = (data) => {
  return axios.post(API_URL, data);
};

export const updateCategory = (id, data) => {
  return axios.put(`${API_URL}/${id}`, data);
};

export const deleteCategory = (id) => {
  return axios.delete(`${API_URL}/${id}`);
};
```

`src\App.jsx`
```jsx
import { useEffect, useState } from "react";
import axios from "axios";
import Swal from "sweetalert2";

import {
  AppBar,
  Toolbar,
  Typography,
  Container,
  Paper,
  Box,
  TextField,
  Button,
  Table,
  TableHead,
  TableBody,
  TableRow,
  TableCell,
  IconButton,
  Dialog,
  DialogTitle,
  DialogContent,
  DialogActions,
  CircularProgress,
  InputAdornment,
} from "@mui/material";

import AddIcon from "@mui/icons-material/Add";
import EditIcon from "@mui/icons-material/Edit";
import DeleteIcon from "@mui/icons-material/Delete";
import SearchIcon from "@mui/icons-material/Search";

const API_URL = "http://localhost:5000/api/categories";

function App() {
  const [categories, setCategories] = useState([]);

  const [open, setOpen] = useState(false);
  const [editing, setEditing] = useState(false);

  const [selectedId, setSelectedId] = useState("");
  const [name, setName] = useState("");

  const [search, setSearch] = useState("");
  const [loading, setLoading] = useState(false);

  // Load categories
  const loadCategories = async () => {
    try {
      setLoading(true);

      const response = await axios.get(API_URL);

      setCategories(response.data);
    } catch (error) {
      console.error(error);

      Swal.fire({
        icon: "error",
        title: "Error",
        text: "Unable to load categories",
      });
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    loadCategories();
  }, []);

  // Open Add dialog
  const handleAdd = () => {
    setEditing(false);
    setSelectedId("");
    setName("");
    setOpen(true);
  };

  // Open Edit dialog
  const handleEdit = (category) => {
    setEditing(true);
    setSelectedId(category._id);
    setName(category.name);
    setOpen(true);
  };

  // Close dialog
  const handleClose = () => {
    setOpen(false);
    setSelectedId("");
    setName("");
    setEditing(false);
  };

  // Save / Update
  const handleSubmit = async (e) => {
    e?.preventDefault();

    if (!name.trim()) {
      Swal.fire({
        icon: "warning",
        title: "Required",
        text: "Category Name is required",
      });

      return;
    }

    try {
      if (editing) {
        await axios.put(`${API_URL}/${selectedId}`, {
          name: name.trim(),
        });
      } else {
        await axios.post(API_URL, {
          name: name.trim(),
        });
      }

      // Close dialog FIRST
      handleClose();

      // Reload categories
      await loadCategories();

      // Show success popup AFTER dialog is closed
      Swal.fire({
        icon: "success",
        title: editing ? "Updated" : "Created",
        text: editing
          ? "Category updated successfully"
          : "Category created successfully",
        timer: 1500,
        showConfirmButton: false,
      });

    } catch (error) {
      console.error(error);

      Swal.fire({
        icon: "error",
        title: "Error",
        text:
          error.response?.data?.message ||
          "Unable to save category",
      });
    }
  };

  // Delete
  const handleDelete = async (category) => {
    const result = await Swal.fire({
      title: "Delete Category?",
      text: `Are you sure you want to delete "${category.name}"?`,
      icon: "warning",
      showCancelButton: true,
      confirmButtonColor: "#d33",
      cancelButtonColor: "#666",
      confirmButtonText: "Yes, Delete",
      cancelButtonText: "Cancel",
    });

    if (!result.isConfirmed) {
      return;
    }

    try {
      await axios.delete(`${API_URL}/${category._id}`);

      await Swal.fire({
        icon: "success",
        title: "Deleted",
        text: "Category deleted successfully",
        timer: 1500,
        showConfirmButton: false,
      });

      loadCategories();
    } catch (error) {
      console.error(error);

      Swal.fire({
        icon: "error",
        title: "Error",
        text:
          error.response?.data?.message ||
          "Unable to delete category",
      });
    }
  };

  // Search
  const filteredCategories = categories.filter((category) =>
    category.name
      .toLowerCase()
      .includes(search.toLowerCase())
  );

  return (
    <>
      {/* Header */}
      <AppBar position="static">
        <Toolbar>
          <Typography
            variant="h6"
            component="div"
            sx={{ flexGrow: 1 }}
          >
            Category Management
          </Typography>

          <Button
            variant="contained"
            color="secondary"
            startIcon={<AddIcon />}
            onClick={handleAdd}
          >
            Add Category
          </Button>
        </Toolbar>
      </AppBar>

      {/* Main */}
      <Container maxWidth="lg" sx={{ mt: 4, mb: 4 }}>
        <Paper elevation={3} sx={{ p: 3 }}>

          {/* Title / Search */}
          <Box
            sx={{
              display: "flex",
              justifyContent: "space-between",
              alignItems: "center",
              mb: 3,
              gap: 2,
            }}
          >
            <Typography variant="h5">
              Categories
            </Typography>

            <TextField
              size="small"
              placeholder="Search category..."
              value={search}
              onChange={(e) => setSearch(e.target.value)}
              InputProps={{
                startAdornment: (
                  <InputAdornment position="start">
                    <SearchIcon />
                  </InputAdornment>
                ),
              }}
            />
          </Box>

          {/* Table */}
          {loading ? (
            <Box
              sx={{
                display: "flex",
                justifyContent: "center",
                p: 5,
              }}
            >
              <CircularProgress />
            </Box>
          ) : (
            <Table>
              <TableHead>
                <TableRow>
                  <TableCell>
                    <strong>MongoDB ID</strong>
                  </TableCell>

                  <TableCell>
                    <strong>Category Name</strong>
                  </TableCell>

                  <TableCell align="right">
                    <strong>Actions</strong>
                  </TableCell>
                </TableRow>
              </TableHead>

              <TableBody>
                {filteredCategories.length === 0 ? (
                  <TableRow>
                    <TableCell
                      colSpan={3}
                      align="center"
                    >
                      No categories found
                    </TableCell>
                  </TableRow>
                ) : (
                  filteredCategories.map((category) => (
                    <TableRow
                      key={category._id}
                      hover
                    >
                      <TableCell>
                        {category._id}
                      </TableCell>

                      <TableCell>
                        {category.name}
                      </TableCell>

                      <TableCell align="right">
                        <IconButton
                          color="primary"
                          onClick={() =>
                            handleEdit(category)
                          }
                        >
                          <EditIcon />
                        </IconButton>

                        <IconButton
                          color="error"
                          onClick={() =>
                            handleDelete(category)
                          }
                        >
                          <DeleteIcon />
                        </IconButton>
                      </TableCell>
                    </TableRow>
                  ))
                )}
              </TableBody>
            </Table>
          )}
        </Paper>
      </Container>

      {/* Add / Edit Dialog */}
      <Dialog
        open={open}
        onClose={handleClose}
        fullWidth
        maxWidth="sm"
      >
        <DialogTitle>
          {editing
            ? "Edit Category"
            : "Add Category"}
        </DialogTitle>

        <DialogContent>
          <TextField
            margin="normal"
            label="Category Name"
            fullWidth
            autoFocus
            value={name}
            onChange={(e) => setName(e.target.value)}
            onKeyDown={(e) => {
              if (e.key === "Enter") {
                handleSubmit();
              }
            }}
          />
        </DialogContent>

        <DialogActions sx={{ p: 2 }}>
          <Button
            onClick={handleClose}
            color="inherit"
          >
            Cancel
          </Button>

          <Button
            variant="contained"
            onClick={handleSubmit}
          >
            {editing ? "Update" : "Save"}
          </Button>
        </DialogActions>
      </Dialog>
    </>
  );
}

export default App;
```
### Recommended UI

For your project, I'd make the screen like this:

```text
┌──────────────────────────────────────────────────────────────┐
│  Category Management                              + Add       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ ID       │ Category Name                    │ Actions  │  │
│  ├──────────┼──────────────────────────────────┼──────────┤  │
│  │ 1        │ Grocery                          │ ✏️  🗑️   │  │
│  │ 2        │ Electronics                      │ ✏️  🗑️   │  │
│  │ 3        │ Stationery                       │ ✏️  🗑️   │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

I can build the **complete MUI Categories CRUD** next, including **MERN API + MUI DataGrid + Add/Edit Dialog + SweetAlert2 delete confirmation + search + pagination**, ready to copy into your Vite project.