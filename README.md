# Shopping App - Redux State Management

A feature-rich shopping application built with React and Redux, demonstrating advanced state management patterns. This project was developed as part of an internship task to showcase proficiency in handling complex application state with Redux Toolkit.

## 🎯 Project Overview

This shopping application integrates with the Fake Store API to display products, manage shopping cart functionality, and demonstrate real-world Redux state management. The app showcases how Redux efficiently handles global state across multiple components without prop drilling.

## ✨ Features

### **Product Management**
- Browse products fetched from Fake Store API
- Product listing with images, prices, and descriptions
- Category-wise filtering
- Product search functionality
- Product detail view

### **Shopping Cart**
- Add/remove products from cart
- Update product quantities
- Real-time price calculation
- Cart item count badge
- Persistent cart state
- Clear cart functionality

### **Redux State Management**
- Centralized state management
- Cart state handled by Redux
- Product state management
- User preferences state
- Async API calls with Redux Thunk

### **UI/UX**
- Responsive design with Tailwind CSS
- Loading states during API calls
- Error handling and user feedback
- Smooth animations and transitions
- Mobile-friendly interface

## 🛠️ Tech Stack

- **React.js** - Frontend UI library
- **Redux Toolkit** - State management (simplified Redux)
- **React-Redux** - React bindings for Redux
- **Redux Thunk** - Async middleware for API calls
- **Fake Store API** - Product data source
- **Tailwind CSS** - Utility-first styling
- **Axios** - HTTP client for API requests
- **React Router DOM** - Navigation and routing
- **React Icons** - Icon library

## 🔄 How Redux Works in This App

### **Redux Architecture**

```
┌─────────────┐
│   UI/View   │ (React Components)
└──────┬──────┘
       │ Dispatch Action
       ▼
┌─────────────┐
│   Actions   │ (User interactions)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Reducers   │ (State update logic)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    Store    │ (Single source of truth)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   UI/View   │ (Re-renders with new state)
└─────────────┘
```

### **Redux Store Configuration**

```javascript
// src/redux/store.js
import { configureStore } from '@reduxjs/toolkit';
import cartReducer from './slices/cartSlice';
import productsReducer from './slices/productsSlice';

export const store = configureStore({
  reducer: {
    cart: cartReducer,
    products: productsReducer
  }
});
```

### **Cart Slice (State Management)**

```javascript
// src/redux/slices/cartSlice.js
import { createSlice } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    totalQuantity: 0,
    totalAmount: 0
  },
  reducers: {
    addToCart: (state, action) => {
      const existingItem = state.items.find(item => item.id === action.payload.id);
      if (existingItem) {
        existingItem.quantity++;
      } else {
        state.items.push({ ...action.payload, quantity: 1 });
      }
      state.totalQuantity++;
      state.totalAmount += action.payload.price;
    },
    removeFromCart: (state, action) => {
      const itemToRemove = state.items.find(item => item.id === action.payload);
      state.totalQuantity -= itemToRemove.quantity;
      state.totalAmount -= itemToRemove.price * itemToRemove.quantity;
      state.items = state.items.filter(item => item.id !== action.payload);
    },
    updateQuantity: (state, action) => {
      const { id, quantity } = action.payload;
      const item = state.items.find(item => item.id === id);
      const quantityDiff = quantity - item.quantity;
      item.quantity = quantity;
      state.totalQuantity += quantityDiff;
      state.totalAmount += item.price * quantityDiff;
    },
    clearCart: (state) => {
      state.items = [];
      state.totalQuantity = 0;
      state.totalAmount = 0;
    }
  }
});

export const { addToCart, removeFromCart, updateQuantity, clearCart } = cartSlice.actions;
export default cartSlice.reducer;
```

### **Products Slice (API Integration)**

```javascript
// src/redux/slices/productsSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

// Async thunk for fetching products
export const fetchProducts = createAsyncThunk(
  'products/fetchProducts',
  async () => {
    const response = await axios.get('https://fakestoreapi.com/products');
    return response.data;
  }
);

const productsSlice = createSlice({
  name: 'products',
  initialState: {
    items: [],
    loading: false,
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});

export default productsSlice.reducer;
```

### **Component Usage**

```javascript
// Using Redux in React components
import { useSelector, useDispatch } from 'react-redux';
import { addToCart } from '../redux/slices/cartSlice';

function ProductCard({ product }) {
  const dispatch = useDispatch();
  
  // Read from Redux store
  const cartItems = useSelector((state) => state.cart.items);
  
  // Dispatch action to update store
  const handleAddToCart = () => {
    dispatch(addToCart(product));
  };
  
  return (
    <div>
      <h3>{product.title}</h3>
      <button onClick={handleAddToCart}>Add to Cart</button>
    </div>
  );
}
```

## 📁 Project Structure

```
shopping-app/
├── public/
├── src/
│   ├── Components/
│   │   ├── Navbar.jsx          # Navigation with cart badge
│   │   ├── ProductCard.jsx     # Individual product display
│   │   ├── CartItem.jsx        # Cart item component
│   │   └── Spinner.jsx         # Loading spinner
│   ├── Pages/
│   │   ├── Home.jsx            # Product listing page
│   │   ├── Cart.jsx            # Shopping cart page
│   │   └── ProductDetail.jsx   # Single product view
│   ├── redux/
│   │   ├── store.js            # Redux store configuration
│   │   └── slices/
│   │       ├── cartSlice.js    # Cart state management
│   │       └── productsSlice.js # Products state management
│   ├── App.jsx                 # Main app component
│   ├── index.jsx               # Entry point with Provider
│   └── index.css               # Global styles
├── package.json
├── tailwind.config.js
└── README.md
```

## 🚀 Getting Started

### Prerequisites
- Node.js (v14 or higher)
- npm or yarn

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/nikhild1111/shopping-app.git
cd shopping-app
```

2. **Install dependencies**
```bash
npm install
```

3. **Start the development server**
```bash
npm start
```

The app will open at `http://localhost:3000`

## 📦 Key Dependencies

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-redux": "^8.1.0",
    "@reduxjs/toolkit": "^1.9.5",
    "react-router-dom": "^6.14.0",
    "axios": "^1.4.0",
    "react-icons": "^4.10.0"
  },
  "devDependencies": {
    "tailwindcss": "^3.3.0"
  }
}
```

## 🎓 Redux Concepts Demonstrated

### **1. Store**
- Single source of truth for application state
- Created using `configureStore` from Redux Toolkit
- Combines multiple reducers

### **2. Slices**
- Modern Redux pattern using Redux Toolkit
- Combines actions and reducers in one file
- Automatically generates action creators

### **3. Actions**
- Plain JavaScript objects describing what happened
- Generated automatically by createSlice
- Examples: `addToCart`, `removeFromCart`, `updateQuantity`

### **4. Reducers**
- Pure functions that specify how state changes
- Take current state and action, return new state
- Handle all cart operations (add, remove, update)

### **5. Dispatch**
- Method to send actions to the store
- Triggers state updates
- Used via `useDispatch()` hook

### **6. Selectors**
- Functions to extract specific data from store
- Used via `useSelector()` hook
- Example: `useSelector(state => state.cart.items)`

### **7. Async Operations**
- Handled using Redux Thunk middleware
- `createAsyncThunk` for API calls
- Manages loading, success, and error states

## 🔍 State Management Flow

1. **User Action** → User clicks "Add to Cart" button
2. **Dispatch Action** → `dispatch(addToCart(product))`
3. **Reducer Executes** → Cart reducer updates state immutably
4. **Store Updates** → Redux store holds new state
5. **Components Re-render** → All subscribed components get new data
6. **UI Updates** → Cart badge shows updated count

## 🎯 Why Redux?

### **Problems Redux Solves:**
- **Prop Drilling** → No need to pass props through multiple levels
- **State Sharing** → Easy sharing of state between components
- **Predictable State** → Single source of truth
- **Debugging** → Redux DevTools for time-travel debugging
- **Middleware** → Handle async operations cleanly

### **Redux vs Context API:**
- Redux better for complex state logic
- Built-in middleware support
- Better performance for large apps
- Excellent debugging tools
- More boilerplate but more powerful

## 🚢 API Integration

**Fake Store API Endpoints Used:**
- `GET /products` - Fetch all products
- `GET /products/:id` - Fetch single product
- `GET /products/categories` - Fetch categories
- `GET /products/category/:category` - Filter by category

## 🎨 Features Breakdown

### **Cart Functionality**
- ✅ Add items to cart
- ✅ Remove items from cart
- ✅ Update item quantities
- ✅ Calculate total price
- ✅ Display item count
- ✅ Clear entire cart
- ✅ Persist cart in localStorage (optional)

### **Product Features**
- ✅ Display all products
- ✅ Category filtering
- ✅ Search products
- ✅ Product details view
- ✅ Loading states
- ✅ Error handling

## 🧪 Testing Redux

```javascript
// Example test for cart reducer
import cartReducer, { addToCart } from './cartSlice';

test('should add item to cart', () => {
  const initialState = { items: [], totalQuantity: 0 };
  const product = { id: 1, title: 'Product', price: 10 };
  
  const newState = cartReducer(initialState, addToCart(product));
  
  expect(newState.items.length).toBe(1);
  expect(newState.totalQuantity).toBe(1);
});
```

## 📈 Performance Optimizations

- Memoized selectors to prevent unnecessary re-renders
- Redux Toolkit's Immer for immutable updates
- Code splitting with React.lazy()
- Debounced search functionality

## 🤝 Contributing

This project was built as an internship assignment. Suggestions and improvements are welcome!

## 📝 Learnings from This Project

1. **Redux Toolkit** simplifies Redux setup significantly
2. **Immutability** is automatic with Immer in Redux Toolkit
3. **Async operations** are cleaner with createAsyncThunk
4. **DevTools** make debugging state changes easy
5. **Middleware** like Thunk handle side effects elegantly

## 📧 Contact

For questions about Redux implementation or this project, feel free to reach out via GitHub.

---

**Built as an internship project to master Redux state management 🚀**
