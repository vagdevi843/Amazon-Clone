# Amazon-Clone
Amazon Clone
import React, { useState, useContext, createContext } from "react";
import { BrowserRouter, Routes, Route, Link, useParams } from "react-router-dom";

/* ---------------- DATA ---------------- */
const PRODUCTS = [
  { id: 1, title: "iPhone 15", price: 79999, category: "Mobile", rating: 4.6, image: "https://via.placeholder.com/200", desc: "Latest Apple iPhone" },
  { id: 2, title: "Samsung S24", price: 69999, category: "Mobile", rating: 4.5, image: "https://via.placeholder.com/200", desc: "Samsung flagship phone" },
  { id: 3, title: "HP Laptop", price: 55999, category: "Laptop", rating: 4.4, image: "https://via.placeholder.com/200", desc: "Powerful laptop for work" },
  { id: 4, title: "Sony Headphones", price: 19999, category: "Accessories", rating: 4.3, image: "https://via.placeholder.com/200", desc: "Noise cancelling headphones" }
];

/* ---------------- CART CONTEXT ---------------- */
const CartContext = createContext();
const useCart = () => useContext(CartContext);

function CartProvider({ children }) {
  const [cart, setCart] = useState([]);

  const addToCart = (product) => {
    const found = cart.find(i => i.id === product.id);
    if (found) {
      setCart(cart.map(i => i.id === product.id ? { ...i, qty: i.qty + 1 } : i));
    } else {
      setCart([...cart, { ...product, qty: 1 }]);
    }
  };

  const remove = (id) => setCart(cart.filter(i => i.id !== id));

  const updateQty = (id, qty) =>
    setCart(cart.map(i => i.id === id ? { ...i, qty } : i));

  return (
    <CartContext.Provider value={{ cart, addToCart, remove, updateQty }}>
      {children}
    </CartContext.Provider>
  );
}

/* ---------------- NAVBAR ---------------- */
function Navbar() {
  const { cart } = useCart();
  return (
    <nav style={styles.nav}>
      <h2>🛍️ Amazon Lite</h2>
      <div>
        <Link to="/">Home</Link>
        <Link to="/cart">Cart ({cart.length})</Link>
      </div>
    </nav>
  );
}

/* ---------------- HOME ---------------- */
function Home() {
  const [search, setSearch] = useState("");
  const [category, setCategory] = useState("All");

  const filtered = PRODUCTS.filter(p =>
    (category === "All" || p.category === category) &&
    p.title.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <div style={styles.container}>
      <input
        placeholder="Search product..."
        onChange={(e) => setSearch(e.target.value)}
        style={styles.input}
      />

      <select onChange={(e) => setCategory(e.target.value)} style={styles.input}>
        <option>All</option>
        <option>Mobile</option>
        <option>Laptop</option>
        <option>Accessories</option>
      </select>

      <div style={styles.grid}>
        {filtered.map(p => (
          <div key={p.id} style={styles.card}>
            <img src={p.image} alt="" />
            <h4>{p.title}</h4>
            <p>₹{p.price}</p>
            <Link to={`/product/${p.id}`}>View</Link>
          </div>
        ))}
      </div>
    </div>
  );
}

/* ---------------- PRODUCT DETAIL ---------------- */
function ProductDetail() {
  const { id } = useParams();
  const { addToCart } = useCart();
  const product = PRODUCTS.find(p => p.id === Number(id));

  return (
    <div style={styles.container}>
      <img src={product.image} alt="" />
      <h2>{product.title}</h2>
      <p>{product.desc}</p>
      <p>Rating ⭐ {product.rating}</p>
      <h3>₹{product.price}</h3>
      <button onClick={() => addToCart(product)}>Add to Cart</button>
    </div>
  );
}

/* ---------------- CART ---------------- */
function Cart() {
  const { cart, remove, updateQty } = useCart();
  const total = cart.reduce((s, i) => s + i.price * i.qty, 0);

  return (
    <div style={styles.container}>
      <h2>Your Cart</h2>

      {cart.length === 0 && <p>Cart is empty</p>}

      {cart.map(i => (
        <div key={i.id} style={styles.cart}>
          <h4>{i.title}</h4>
          <input
            type="number"
            min="1"
            value={i.qty}
            onChange={(e) => updateQty(i.id, Number(e.target.value))}
          />
          <span> ₹{i.price * i.qty}</span>
          <button onClick={() => remove(i.id)}>Remove</button>
        </div>
      ))}

      <h3>Total: ₹{total}</h3>
    </div>
  );
}

/* ---------------- APP ---------------- */
export default function App() {
  return (
    <CartProvider>
      <BrowserRouter>
        <Navbar />
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/product/:id" element={<ProductDetail />} />
          <Route path="/cart" element={<Cart />} />
        </Routes>
      </BrowserRouter>
    </CartProvider>
  );
}

/* ---------------- STYLES ---------------- */
const styles = {
  nav: {
    background: "#131921",
    color: "#fff",
    padding: "15px",
    display: "flex",
    justifyContent: "space-between"
  },
  container: { padding: "20px" },
  input: { padding: "8px", marginRight: "10px" },
  grid: {
    display: "grid",
    gridTemplateColumns: "repeat(auto-fit,minmax(200px,1fr))",
    gap: "15px",
    marginTop: "15px"
  },
  card: {
    background: "#fff",
    padding: "10px",
    textAlign: "center",
    borderRadius: "8px"
  },
  cart: {
    display: "flex",
    gap: "10px",
    marginBottom: "10px"
  }
};
