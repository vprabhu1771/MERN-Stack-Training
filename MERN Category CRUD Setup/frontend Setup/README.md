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

For example:

```jsx
import {
  Box,
  Button,
  TextField,
  Paper,
  Typography
} from "@mui/material";

export default function CategoryForm() {
  return (
    <Paper sx={{ p: 3 }}>
      <Typography variant="h5" mb={3}>
        Category
      </Typography>

      <Box display="flex" gap={2}>
        <TextField
          label="ID"
          type="number"
        />

        <TextField
          label="Category Name"
          fullWidth
        />

        <Button
          variant="contained"
          size="large"
        >
          Save
        </Button>
      </Box>
    </Paper>
  );
}
```

For the table:

```jsx
import {
  Table,
  TableHead,
  TableBody,
  TableRow,
  TableCell,
  IconButton
} from "@mui/material";

import EditIcon from "@mui/icons-material/Edit";
import DeleteIcon from "@mui/icons-material/Delete";

<Table>
  <TableHead>
    <TableRow>
      <TableCell>ID</TableCell>
      <TableCell>Name</TableCell>
      <TableCell align="right">Actions</TableCell>
    </TableRow>
  </TableHead>

  <TableBody>
    {categories.map((category) => (
      <TableRow key={category.id}>
        <TableCell>{category.id}</TableCell>
        <TableCell>{category.name}</TableCell>

        <TableCell align="right">
          <IconButton
            color="primary"
            onClick={() => editCategory(category)}
          >
            <EditIcon />
          </IconButton>

          <IconButton
            color="error"
            onClick={() => deleteCategory(category.id)}
          >
            <DeleteIcon />
          </IconButton>
        </TableCell>
      </TableRow>
    ))}
  </TableBody>
</Table>
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