import { useMemo, useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from "@/components/ui/dialog";
import { motion } from "framer-motion";

// ----- UTILITIES -----
const currency = (n) => `₹${n.toLocaleString("en-IN")}`;
const uid = () => Math.random().toString(36).slice(2, 10).toUpperCase();

// ----- SEED DATA (replace with your real photos & details) -----
const seedProducts = [
  { id: "s1", name: "Designer Suit – Maroon", price: 2499, image: "/suit1.jpg", sizes: ["S","M","L","XL"], category: "Designer", fabric: "Georgette" },
  { id: "s2", name: "Designer Suit – Emerald", price: 3199, image: "/suit2.jpg", sizes: ["S","M","L","XL"], category: "Designer", fabric: "Silk Blend" },
  { id: "s3", name: "Party Wear Suit – Gold", price: 4299, image: "/suit3.jpg", sizes: ["M","L","XL"], category: "Party Wear", fabric: "Chiffon" },
  { id: "s4", name: "Daily Wear – Floral Blue", price: 1599, image: "/suit4.jpg", sizes: ["S","M","L"], category: "Daily", fabric: "Cotton" },
];

export default function LaxmiCollection() {
  const [page, setPage] = useState("home");
  const [products] = useState(seedProducts);
  const [query, setQuery] = useState("");
  const [filter, setFilter] = useState("All");
  const [cart, setCart] = useState(() => JSON.parse(localStorage.getItem("lc_cart")||"[]"));
  const [customer, setCustomer] = useState({ name: "", phone: "", email: "", address: "", payment: "COD" });
  const [tracking, setTracking] = useState("");

  useEffect(()=>{ localStorage.setItem("lc_cart", JSON.stringify(cart)); }, [cart]);

  const categories = useMemo(()=>["All", ...new Set(products.map(p=>p.category))], [products]);

  const shown = useMemo(()=> products.filter(p =>
    (filter === "All" || p.category === filter) &&
    (p.name.toLowerCase().includes(query.toLowerCase()) || p.fabric.toLowerCase().includes(query.toLowerCase()))
  ), [products, query, filter]);

  const addToCart = (product, size="M") => {
    setCart(prev => {
      const idx = prev.findIndex(i => i.id === product.id && i.size === size);
      if (idx > -1) {
        const copy = [...prev];
        copy[idx].qty += 1;
        return copy;
      }
      return [...prev, { id: product.id, name: product.name, price: product.price, size, qty: 1 }];
    });
  };

  const updateQty = (i, delta) => {
    setCart(prev => prev.map((it, idx) => idx===i ? { ...it, qty: Math.max(1, it.qty+delta)} : it));
  };
  const removeItem = (i) => setCart(prev => prev.filter((_, idx)=> idx!==i));

  const subtotal = cart.reduce((s,i)=> s + i.price * i.qty, 0);
  const shipping = subtotal > 4999 || subtotal===0 ? 0 : 99;
  const total = subtotal + shipping;

  const logoSrc = "/logo.png"; // Replace with your uploaded logo path

  // ----- CHECKOUT -----
  const placeOrder = () => {
    if (!customer.name || !customer.phone || !customer.address) {
      alert("Please fill name, phone and address.");
      return;
    }
    if (cart.length===0) { alert("Your cart is empty."); return; }

    const orderId = `LC-${uid()}`;
    const lines = cart.map(it => `${it.name} (${it.size}) x ${it.qty} = ${currency(it.price*it.qty)}`).join("
");
    const text = `New order ${orderId}%0A%0ACustomer: ${customer.name}%0APhone: ${customer.phone}%0AEmail: ${customer.email||"-"}%0AAddress: ${customer.address.replace(/
/g," ")}%0A%0AItems:%0A${encodeURIComponent(lines)}%0A%0ASubtotal: ${currency(subtotal)}%0AShipping: ${currency(shipping)}%0ATotal: ${currency(total)}%0A%0APayment: ${customer.payment}`;

    // Save a lightweight tracking record in localStorage
    const record = { orderId, items: cart, total, status: customer.payment==="Online"?"Awaiting Payment":"Confirmed", placedAt: new Date().toISOString(), name: customer.name };
    const all = JSON.parse(localStorage.getItem("lc_orders")||"[]");
    localStorage.setItem("lc_orders", JSON.stringify([record, ...all]));

    // WhatsApp handoff (store owner)
    const wa = `https://wa.me/917310670194?text=${text}`;

    if (customer.payment === "Online" && PAYMENT_LINK) {
      // Optional: redirect to your Stripe/Razorpay payment link BEFORE WhatsApp
      window.open(PAYMENT_LINK, "_blank");
    }

    window.open(wa, "_blank");
    setCart([]);
    setPage("track");
    setTracking(orderId);
  };

  // >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
  // Set this to your real payment link (Stripe/Razorpay Payment Link)
  const PAYMENT_LINK = ""; // e.g. "https://rzp.io/l/laxmi-collection" or Stripe payment page
  // <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

  return (
    <div className="bg-purple-900 min-h-screen text-yellow-400">
      {/* TOP BAR */}
      <div className="max-w-6xl mx-auto px-4 py-6 flex items-center justify-between">
        <div className="flex items-center gap-3">
          <img src={logoSrc} alt="Laxmi Collection" className="w-14 h-14 rounded-full ring-2 ring-yellow-400" />
          <div>
            <h1 className="text-2xl md:text-3xl font-extrabold tracking-wide">LAXMI COLLECTION</h1>
            <p className="text-xs uppercase tracking-widest">Best Quality Suits</p>
          </div>
        </div>
        <nav className="flex gap-2 md:gap-3">
          <Button variant="secondary" onClick={()=>setPage("home")}>Home</Button>
          <Button variant="secondary" onClick={()=>setPage("shop")}>Collection</Button>
          <Button variant="secondary" onClick={()=>setPage("cart")}>Cart ({cart.reduce((s,i)=>s+i.qty,0)})</Button>
          <Button variant="secondary" onClick={()=>setPage("track")}>Track Order</Button>
        </nav>
      </div>

      {/* HERO */}
      {page==="home" && (
        <section className="max-w-6xl mx-auto px-4 pb-12">
          <motion.div initial={{opacity:0, y:20}} animate={{opacity:1, y:0}} className="bg-yellow-100 text-purple-900 rounded-2xl p-8 md:p-12 shadow-xl grid md:grid-cols-2 gap-8">
            <div>
              <h2 className="text-4xl font-extrabold mb-3">Grace. Quality. Comfort.</h2>
              <p className="text-lg mb-6">Explore our curated collection of designer, party and daily-wear suits. Crafted with premium fabrics and tailored fits.</p>
              <div className="flex gap-3">
                <Button onClick={()=>setPage("shop")}>Shop Now</Button>
                <Button variant="outline" onClick={()=>window.open("https://wa.me/917310670194","_blank")}>Chat on WhatsApp</Button>
              </div>
            </div>
            <img src="/hero-saree.png" alt="Laxmi Collection" className="w-full rounded-2xl object-cover" />
          </motion.div>
        </section>
      )}

      {/* SHOP */}
      {page==="shop" && (
        <section className="max-w-6xl mx-auto px-4 pb-16">
          <div className="flex flex-col md:flex-row gap-3 md:items-center md:justify-between mb-6">
            <div className="flex gap-2 w-full md:w-2/3">
              <Input placeholder="Search by name, fabric..." value={query} onChange={e=>setQuery(e.target.value)} />
              <Select value={filter} onValueChange={setFilter}>
                <SelectTrigger className="w-44"><SelectValue placeholder="Category" /></SelectTrigger>
                <SelectContent>
                  {categories.map(c => (<SelectItem key={c} value={c}>{c}</SelectItem>))}
                </SelectContent>
              </Select>
            </div>
            <div className="text-sm opacity-90">Call: 7310670194 • Email: laxmicollection24@yahoo.com</div>
          </div>

          <div className="grid sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
            {shown.map(p => (
              <Card key={p.id} className="bg-yellow-100 text-purple-900 rounded-2xl overflow-hidden">
                <CardContent className="p-0">
                  <img src={p.image} alt={p.name} className="w-full h-64 object-cover" />
                  <div className="p-4">
                    <h3 className="text-lg font-semibold leading-tight">{p.name}</h3>
                    <p className="text-sm opacity-80">Fabric: {p.fabric}</p>
                    <div className="flex items-center justify-between mt-2">
                      <span className="text-xl font-bold">{currency(p.price)}</span>
                      <Dialog>
                        <DialogTrigger asChild>
                          <Button size="sm">Select Size</Button>
                        </DialogTrigger>
                        <DialogContent className="bg-white text-purple-900">
                          <DialogHeader>
                            <DialogTitle>{p.name}</DialogTitle>
                          </DialogHeader>
                          <div className="flex flex-wrap gap-2">
                            {p.sizes.map(s => (
                              <Button key={s} variant="secondary" onClick={()=>addToCart(p, s)}>Add {s}</Button>
                            ))}
                          </div>
                        </DialogContent>
                      </Dialog>
                    </div>
                  </div>
                </CardContent>
              </Card>
            ))}
          </div>
        </section>
      )}

      {/* CART + CHECKOUT */}
      {page==="cart" && (
        <section className="max-w-6xl mx-auto px-4 pb-16 grid md:grid-cols-3 gap-8">
          <div className="md:col-span-2">
            <h2 className="text-2xl font-bold mb-4">Your Cart</h2>
            {cart.length===0 ? (
              <p className="opacity-90">Your cart is empty. Go to <button className="underline" onClick={()=>setPage("shop")}>Collection</button>.</p>
            ) : (
              <div className="space-y-3">
                {cart.map((it, i) => (
                  <div key={i} className="bg-yellow-100 text-purple-900 rounded-xl p-4 flex items-center justify-between">
                    <div>
                      <div className="font-semibold">{it.name} • Size {it.size}</div>
                      <div className="text-sm opacity-80">{currency(it.price)} each</div>
                    </div>
                    <div className="flex items-center gap-2">
                      <Button variant="secondary" onClick={()=>updateQty(i, -1)}>-</Button>
                      <span className="font-semibold">{it.qty}</span>
                      <Button variant="secondary" onClick={()=>updateQty(i, +1)}>+</Button>
                      <Button variant="outline" onClick={()=>removeItem(i)}>Remove</Button>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </div>

          {/* SUMMARY / CHECKOUT FORM */}
          <div>
            <h3 className="text-xl font-semibold mb-3">Order Summary</h3>
            <div className="bg-yellow-100 text-purple-900 rounded-2xl p-4 space-y-2 mb-4">
              <div className="flex justify-between"><span>Subtotal</span><span>{currency(subtotal)}</span></div>
              <div className="flex justify-between"><span>Shipping</span><span>{shipping===0?"FREE":currency(shipping)}</span></div>
              <div className="flex justify-between font-bold text-lg"><span>Total</span><span>{currency(total)}</span></div>
              <p className="text-xs opacity-70">Free shipping over ₹5,000</p>
            </div>

            <div className="bg-yellow-100 text-purple-900 rounded-2xl p-4 space-y-3">
              <Input placeholder="Full Name" value={customer.name} onChange={e=>setCustomer({...customer, name:e.target.value})} />
              <Input placeholder="Phone (WhatsApp)" value={customer.phone} onChange={e=>setCustomer({...customer, phone:e.target.value})} />
              <Input placeholder="Email (optional)" value={customer.email} onChange={e=>setCustomer({...customer, email:e.target.value})} />
              <Textarea rows={4} placeholder="Full Address with Pincode" value={customer.address} onChange={e=>setCustomer({...customer, address:e.target.value})} />
              <div>
                <label className="text-sm">Payment Method</label>
                <div className="flex gap-2 mt-2">
                  <Button variant={customer.payment==="COD"?"default":"secondary"} onClick={()=>setCustomer({...customer, payment:"COD"})}>Cash on Delivery</Button>
                  <Button variant={customer.payment==="Online"?"default":"secondary"} onClick={()=>setCustomer({...customer, payment:"Online"})}>Online (UPI/Card)</Button>
                </div>
                {customer.payment==="Online" && (
                  <p className="text-xs mt-2">After clicking "Place Order", you'll be redirected to WhatsApp. If a payment link is configured, it will open first.</p>
                )}
              </div>
              <Button className="w-full" onClick={placeOrder} disabled={cart.length===0}>Place Order</Button>
            </div>
          </div>
        </section>
      )}

      {/* TRACK ORDER */}
      {page==="track" && (
        <section className="max-w-3xl mx-auto px-4 pb-16">
          <h2 className="text-2xl font-bold mb-4">Track Your Order</h2>
          <div className="bg-yellow-100 text-purple-900 rounded-2xl p-4">
            <p className="text-sm mb-3">Enter your Order ID (e.g., LC-ABCDEFGH) from WhatsApp confirmation.</p>
            <div className="flex gap-2 mb-4">
              <Input placeholder="Order ID" value={tracking} onChange={e=>setTracking(e.target.value.toUpperCase())} />
              <Button onClick={()=>setTracking(tracking)}>Check</Button>
            </div>
            <OrderStatus orderId={tracking} />
          </div>
        </section>
      )}

      {/* FOOTER */}
      <footer className="py-10 border-t border-purple-800">
        <div className="max-w-6xl mx-auto px-4 grid md:grid-cols-3 gap-6">
          <div>
            <div className="font-bold text-lg">Laxmi Collection</div>
            <div className="text-sm opacity-90">Best Quality Suits</div>
          </div>
          <div className="text-sm">
            <div>Email: laxmicollection24@yahoo.com</div>
            <div>Phone/WhatsApp: 7310670194</div>
          </div>
          <div className="text-sm">
            <div>Returns within 7 days of delivery (unused, with tags).</div>
            <div>All prices in INR. © {new Date().getFullYear()}</div>
          </div>
        </div>
      </footer>
    </div>
  );
}

function OrderStatus({ orderId }){
  if (!orderId) return <div className="text-sm">Enter an Order ID above.</div>;
  const orders = JSON.parse(localStorage.getItem("lc_orders")||"[]");
  const order = orders.find(o=>o.orderId===orderId);
  if (!order) return <div className="text-sm">No order found for <b>{orderId}</b>. If you just ordered, wait a moment and try again.</div>;
  return (
    <div>
      <div className="font-semibold mb-2">Order {order.orderId}</div>
      <div className="text-sm mb-3 opacity-90">Placed on {new Date(order.placedAt).toLocaleString()}</div>
      <div className="bg-white rounded-xl p-4 text-purple-900">
        <div className="mb-2"><b>Status:</b> {order.status}</div>
        <div className="mb-2"><b>Total:</b> {currency(order.total)}</div>
        <div className="mb-2"><b>Items:</b>
          <ul className="list-disc ml-6">
            {order.items.map((i, idx)=>(<li key={idx}>{i.name} • {i.size} × {i.qty}</li>))}
          </ul>
        </div>
        <p className="text-xs opacity-80">For updates, message us on WhatsApp with your Order ID.</p>
      </div>
    </div>
  );
}
