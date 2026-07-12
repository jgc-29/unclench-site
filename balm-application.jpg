/* ============================================================
   UNCLENCH — store engine
   Product catalogue, cart state, drawer UI, checkout hook.
   Cart persists in memory for the session (see note in checklist
   about swapping to a real backend + Stripe).
   ============================================================ */

/* ---- PRODUCT CATALOGUE ----
   priceId is where your Stripe Price ID goes once created.
   Leave as null and checkout falls back to demo mode.        */
const CATALOGUE = {
  "night-plate-soft": {
    id:"night-plate-soft", name:"The Night Plate — Soft", price:119, priceId:null,
    blurb:"Custom soft nightguard for light-to-moderate clenchers. Dentist-reviewed.",
    thumb:"SOFT", thumbClass:""
  },
  "night-plate-hard": {
    id:"night-plate-hard", name:"The Night Plate — Hard", price:169, priceId:null,
    blurb:"Clinical-grade hard build for heavy grinders and restored teeth.",
    thumb:"HARD", thumbClass:""
  },
  "night-pair-soft": {
    id:"night-pair-soft", name:"Night Plate Pair — Soft", price:189, priceId:null,
    blurb:"Two soft guards. Upper, lower, or a spare.",
    thumb:"SOFT ×2", thumbClass:""
  },
  "night-pair-hard": {
    id:"night-pair-hard", name:"Night Plate Pair — Hard", price:269, priceId:null,
    blurb:"Two clinical-grade hard guards.",
    thumb:"HARD ×2", thumbClass:""
  },
  "day-plate": {
    id:"day-plate", name:"The Day Plate", price:139, priceId:null,
    blurb:"Ultra-thin, invisible daytime guard. Habit programme included.",
    thumb:"DAY", thumbClass:"coral"
  },
  "reset-kit": {
    id:"reset-kit", name:"The Reset Kit", price:179, priceId:null,
    blurb:"Soft plate + Unwind + jaw balm + release tool + programme.",
    thumb:"RESET KIT", thumbClass:"coral"
  },
  "unwind": {
    id:"unwind", name:"Unwind", price:24, priceId:null, sub:true,
    blurb:"Magnesium glycinate + L-theanine. Nightly.",
    thumb:"UNWIND", thumbClass:"sage"
  },
  "refresh-soft": {
    id:"refresh-soft", name:"Refresh — Soft", price:69, priceId:null, sub:true,
    blurb:"Replacement soft plate every 6–12 months.",
    thumb:"REFRESH", thumbClass:"sage"
  },
  "refresh-hard": {
    id:"refresh-hard", name:"Refresh — Hard", price:99, priceId:null, sub:true,
    blurb:"Replacement hard plate every 6–12 months.",
    thumb:"REFRESH", thumbClass:"sage"
  },
  "jaw-balm": {
    id:"jaw-balm", name:"Jaw Balm", price:16, priceId:null,
    blurb:"Warming magnesium & arnica for masseter and temples.",
    thumb:"BALM", thumbClass:"coral"
  },
  "release-tool": {
    id:"release-tool", name:"The Release Tool", price:22, priceId:null,
    blurb:"Trigger-point tool shaped for the jaw, with video protocols.",
    thumb:"TOOL", thumbClass:"sage"
  },
  "tension-duo": {
    id:"tension-duo", name:"The Tension Duo", price:34, priceId:null,
    blurb:"Jaw balm + release tool, together. Save £4.",
    thumb:"DUO", thumbClass:"coral"
  }
};

/* ---- CART STATE ---- */
let CART = [];

function cartAdd(id, variant){
  const p = CATALOGUE[id];
  if(!p) return;
  const key = id + (variant ? "::"+variant : "");
  const existing = CART.find(l => l.key === key);
  if(existing){ existing.qty += 1; }
  else { CART.push({key, id, variant:variant||null, qty:1}); }
  renderCart();
  openCart();
  pulseCartCount();
}
function cartRemove(key){ CART = CART.filter(l => l.key !== key); renderCart(); }
function cartQty(key, delta){
  const l = CART.find(x => x.key === key);
  if(!l) return;
  l.qty += delta;
  if(l.qty < 1) cartRemove(key); else renderCart();
}
function cartCount(){ return CART.reduce((n,l)=>n+l.qty,0); }
function cartTotal(){ return CART.reduce((s,l)=>s + CATALOGUE[l.id].price * l.qty, 0); }

/* ---- DRAWER UI ---- */
function openCart(){ document.querySelector('.cart-overlay')?.classList.add('open'); document.querySelector('.cart-drawer')?.classList.add('open'); }
function closeCart(){ document.querySelector('.cart-overlay')?.classList.remove('open'); document.querySelector('.cart-drawer')?.classList.remove('open'); }

function pulseCartCount(){
  const el = document.querySelector('.cart-count');
  if(!el) return;
  el.animate([{transform:'scale(1)'},{transform:'scale(1.4)'},{transform:'scale(1)'}],{duration:300});
}

function renderCart(){
  document.querySelectorAll('.cart-count').forEach(el=>{
    const c = cartCount();
    el.textContent = c;
    el.style.display = c > 0 ? 'inline-flex' : 'none';
  });
  const items = document.querySelector('.cart-items');
  const foot = document.querySelector('.cart-foot');
  if(!items) return;

  if(CART.length === 0){
    items.innerHTML = `<div class="cart-empty">
      <div class="ce-word">Nothing held.</div>
      <p style="font-size:14px">Your cart is relaxed. Add something to change that.</p>
    </div>`;
    if(foot) foot.style.display = 'none';
    return;
  }
  if(foot) foot.style.display = 'block';

  items.innerHTML = CART.map(l=>{
    const p = CATALOGUE[l.id];
    return `<div class="ci">
      <div class="ci-thumb ${p.thumbClass}">${p.thumb}</div>
      <div class="ci-body">
        <div class="ci-name">${p.name}</div>
        <div class="ci-variant">${l.variant ? l.variant : (p.sub ? 'Subscription' : 'One-time')}</div>
        <div class="ci-controls">
          <div class="qty">
            <button onclick="cartQty('${l.key}',-1)">–</button>
            <span>${l.qty}</span>
            <button onclick="cartQty('${l.key}',1)">+</button>
          </div>
          <button class="ci-remove" onclick="cartRemove('${l.key}')">Remove</button>
        </div>
      </div>
      <div class="ci-price">£${p.price * l.qty}</div>
    </div>`;
  }).join('');

  const sub = document.querySelector('.cart-subtotal span:last-child');
  if(sub) sub.textContent = '£' + cartTotal();
}

/* ---- CHECKOUT HOOK ----
   In production this posts the cart to your server, which creates
   a Stripe Checkout Session and returns a URL to redirect to.
   Until that backend exists, we route to the local checkout page. */
async function checkout(){
  if(CART.length === 0) return;

  // ---- PRODUCTION PATH (uncomment once your backend is live) ----
  // const res = await fetch('/api/create-checkout-session', {
  //   method:'POST', headers:{'Content-Type':'application/json'},
  //   body: JSON.stringify({ items: CART.map(l=>({ id:l.id, variant:l.variant, qty:l.qty })) })
  // });
  // const { url } = await res.json();
  // window.location = url;   // Stripe-hosted checkout
  // return;

  // ---- DEMO PATH ----
  const payload = encodeURIComponent(JSON.stringify(CART));
  window.location = 'checkout.html?cart=' + payload;
}

/* ---- INJECT SHARED CART DRAWER + FOOTER on every page ---- */
function injectChrome(){
  if(!document.querySelector('.cart-drawer')){
    const drawer = document.createElement('div');
    drawer.innerHTML = `
      <div class="cart-overlay" onclick="closeCart()"></div>
      <aside class="cart-drawer" aria-label="Cart">
        <div class="cart-head">
          <h3>Your cart</h3>
          <button class="cart-close" onclick="closeCart()">×</button>
        </div>
        <div class="cart-items"></div>
        <div class="cart-foot">
          <div class="cart-subtotal"><span>Subtotal</span><span>£0</span></div>
          <p class="cart-note">Shipping &amp; any applicable VAT calculated at checkout.</p>
          <button class="btn btn-coral btn-full" onclick="checkout()">Checkout →</button>
        </div>
      </aside>`;
    document.body.appendChild(drawer);
  }
  renderCart();
}

/* ---- scroll reveal ---- */
function initReveal(){
  const els = document.querySelectorAll('.reveal');
  if(!('IntersectionObserver' in window)){ els.forEach(e=>e.classList.add('in')); return; }
  const io = new IntersectionObserver((entries)=>{
    entries.forEach(e=>{ if(e.isIntersecting){ e.target.classList.add('in'); io.unobserve(e.target); } });
  },{threshold:0.12});
  els.forEach(e=>io.observe(e));
}

document.addEventListener('DOMContentLoaded', ()=>{ injectChrome(); initReveal(); });
