# ☕ PREMIUM COFFEE SHOP E-COMMERCE APP — MASTER PROMPT (v2 with Brand Guidelines)

> Copy everything below the line and paste it into Claude (Opus 4.7 recommended) to generate a complete, production-quality coffee shop application with full brand identity.

---

## 🎯 PROMPT START — COPY FROM HERE

Build me a complete, production-ready coffee shop e-commerce web application called **"BrewHaven"** as a single-file React artifact. This should be a visually stunning, fully-functional premium app — not a demo. Think Starbucks meets Blue Bottle Coffee with modern e-commerce polish. Follow the brand guidelines below strictly — every pixel should feel on-brand.

---

## 📘 BRAND GUIDELINES (Strictly Follow)

### 1. BRAND IDENTITY

**Brand Name:** BrewHaven
**Tagline:** *"Crafted with Love, Brewed to Perfection"*
**Short Tagline (for buttons/badges):** *"Sip the Moment"*

**Brand Mission:**
To serve the world's finest handcrafted coffee while creating warm, welcoming moments — one cup at a time.

**Brand Vision:**
To be the most-loved neighborhood coffee destination where quality meets comfort.

**Brand Personality:**
- Warm & Welcoming (not cold or corporate)
- Premium but Approachable (not snobby)
- Artisan & Authentic (not mass-produced)
- Cozy & Comfortable (like a friend's living room)
- Passionate about Craft (every bean, every brew)

**Brand Values:**
1. Quality First — premium beans, perfect brews
2. Human Connection — every customer matters
3. Sustainability — ethically sourced, eco-friendly packaging
4. Craftsmanship — attention to every detail
5. Community — a gathering place, not just a shop

---

### 2. LOGO CONCEPT

**Logo Design:**
- Use a coffee-cup icon combined with a rising steam/leaf motif
- Place the word "BrewHaven" next to or below the icon
- Use Lucide's `Coffee` icon as the base mark
- Circular stamp-style badge version for favicons and small spaces

**Logo Usage Rules:**
- Minimum padding around logo: equal to the height of the "B" letter
- Never stretch, skew, or rotate the logo
- On dark backgrounds: use cream/caramel color
- On light backgrounds: use deep coffee brown
- Never place logo on busy photo backgrounds without an overlay

**Logo Placement:**
- Top-left of navbar (always visible, sticky)
- Footer centered
- Login/confirmation screens: centered, large

---

### 3. COLOR PALETTE

**Primary Colors (Coffee Tones):**
- `#4A2C17` — Deep Espresso (primary text, logo, headings)
- `#6F4E37` — Coffee Brown (primary buttons, accents)
- `#8B5E3C` — Medium Roast (hover states, secondary buttons)

**Secondary Colors (Caramel & Cream):**
- `#D4A574` — Caramel Gold (highlights, CTAs, ratings)
- `#C68B4F` — Warm Caramel (active states, badges)
- `#F5E6D3` — Warm Cream (card backgrounds, sections)
- `#FAF7F2` — Soft Cream (main page background)

**Accent Colors (Use Sparingly):**
- `#2D6A4F` — Forest Green (success, "Available", organic tags)
- `#C1440E` — Burnt Orange (sale tags, urgent notices)
- `#8B1A1A` — Deep Red (errors, out-of-stock)

**Neutral Colors:**
- `#2C1810` — Near Black (body text)
- `#6B5B4F` — Warm Gray (secondary text, captions)
- `#D6CFC4` — Light Gray (borders, dividers)
- `#FFFFFF` — Pure White (minimal use, only for contrast cards)

**Color Usage Rules (60-30-10 Rule):**
- 60% Cream/White (backgrounds) — dominant
- 30% Coffee Brown (text, nav, footer) — supportive
- 10% Caramel Gold (CTAs, highlights) — accent
- Never use pure black (`#000`) — always use Deep Espresso
- Never use cold grays or blues — stay in warm family
- Gradients allowed: Coffee Brown → Caramel for hero sections

---

### 4. TYPOGRAPHY SYSTEM

**Font Families:**
- **Headings (H1-H3):** Serif — use Tailwind's `font-serif` (creates elegant, artisan feel)
- **Body + UI:** Sans-serif — use Tailwind's `font-sans`
- **Accent / Pricing:** Bold sans-serif for numbers

**Type Scale:**
- Hero H1: `text-6xl md:text-7xl`, `font-serif`, `font-bold`, tracking-tight
- Page H1: `text-4xl md:text-5xl`, `font-serif`, `font-bold`
- Section H2: `text-3xl md:text-4xl`, `font-serif`, `font-semibold`
- Card H3: `text-xl md:text-2xl`, `font-serif`, `font-semibold`
- Body: `text-base`, `font-sans`, `leading-relaxed`
- Caption: `text-sm`, `font-sans`, warm gray
- Price: `text-2xl`, `font-bold`, `font-sans`
- Button: `text-base`, `font-semibold`, `tracking-wide`

**Typography Rules:**
- Line height: relaxed for body (`leading-relaxed`), tight for headings (`leading-tight`)
- Letter spacing: tight on big headings, normal on body, wide on buttons/labels
- Never use more than 3 font weights per screen
- All-caps only for small labels, badges, or buttons (never body copy)

---

### 5. VOICE & TONE

**Brand Voice Attributes:**
- Warm (not corporate)
- Poetic (not dry)
- Confident (not arrogant)
- Friendly (not overly casual)

**Writing Examples:**

✅ **DO write:**
- "Your morning ritual, reimagined."
- "Hand-picked beans from the highlands of Ethiopia."
- "Steaming hot, just the way you like it."
- "Your cart is feeling lonely — let's fix that."
- "One more step to caffeinated bliss."

❌ **DON'T write:**
- "Buy now! Limited time!" (too pushy)
- "Click here to purchase this item" (too robotic)
- "LOL, your cart is empty bro" (too casual)
- "Premium luxury ultra-exclusive coffee" (overselling)

**Microcopy Tone Guide:**
- **Buttons:** Action-oriented, warm → "Add to My Cup", "Brew My Order", "Take Me There"
- **Empty states:** Gentle, inviting → "No favorites yet — explore our menu and fall in love"
- **Errors:** Apologetic, helpful → "Oops, that code doesn't brew right. Try another?"
- **Success:** Joyful, celebratory → "Your order is on its way! ☕"
- **Loading:** Charming → "Brewing your order..." instead of "Loading..."

---

### 6. IMAGERY & ICONOGRAPHY STYLE

**Product Visuals:**
- Use relevant emojis as placeholders: ☕ 🍵 🥐 🍰 🫘 🥛
- Or generate simple SVG illustrations in brand colors
- Never use generic stock-photo placeholders

**Icon Style:**
- Use `lucide-react` icons throughout for consistency
- Icon stroke width: 1.5–2px (not too thick, not too thin)
- Icon color matches surrounding text color
- Sizes: 16px (inline), 20px (buttons), 24px (nav), 32px+ (feature sections)

**Visual Motifs:**
- Coffee bean clusters as decorative elements
- Rising steam lines above hot drinks (subtle animation)
- Leaf/plant accents for organic/vegan tags
- Circular badges for promotions
- Soft blob/organic shapes rather than sharp geometric ones

**Photography Direction (if images added later):**
- Warm, natural lighting (golden hour feel)
- Overhead flat-lay of coffee + pastry
- Hands holding cups (human warmth)
- Never cold, studio-white, sterile shots

---

### 7. UI PATTERNS & COMPONENTS

**Buttons:**
- **Primary:** Coffee Brown bg, cream text, rounded-full, px-8 py-3, hover darkens to Deep Espresso with subtle lift (shadow-lg)
- **Secondary:** Transparent bg, Coffee Brown border + text, hover fills with Coffee Brown
- **Tertiary/Ghost:** No border, Coffee Brown text, underline on hover
- **Destructive:** Deep Red bg, white text (only for delete confirmations)
- All buttons: `transition-all duration-300`, subtle shadow on hover

**Cards:**
- Background: White or Warm Cream
- Border: 1px solid Light Gray OR no border + soft shadow
- Border-radius: `rounded-2xl` (generous, cozy)
- Padding: `p-6` minimum
- Hover: `hover:shadow-xl hover:-translate-y-1 transition`

**Inputs:**
- Background: Soft Cream or White
- Border: 2px solid Light Gray, focus:border-Caramel
- Rounded-lg, px-4 py-3
- Label above, placeholder inside, helper text below
- Error state: Deep Red border + error message in red

**Badges:**
- Rounded-full, px-3 py-1, text-xs, font-semibold
- "New" → Caramel Gold bg
- "Popular" → Coffee Brown bg
- "Vegan" → Forest Green bg
- "Sale" → Burnt Orange bg

**Navigation:**
- Sticky top, cream/white bg with subtle bottom shadow on scroll
- Height: 72px desktop, 64px mobile
- Logo left, menu center, icons right (search, favorites, cart, user)
- Cart icon: coffee-brown bag with caramel-colored count badge

**Shadows:**
- Small: `shadow-sm` (subtle card lift)
- Medium: `shadow-md` (hover states)
- Large: `shadow-xl` (modals, floating elements)
- Always warm-tinted, never cold blue shadows

**Border Radius Scale:**
- Small (badges, chips): `rounded-full` or `rounded-md`
- Medium (buttons, inputs): `rounded-lg` to `rounded-xl`
- Large (cards, modals): `rounded-2xl` to `rounded-3xl`

**Spacing Scale (use Tailwind's default):**
- Between sections: `py-16 md:py-24`
- Between cards: `gap-6 md:gap-8`
- Inside cards: `p-6`
- Between text blocks: `space-y-4`

---

### 8. ANIMATION & INTERACTION PRINCIPLES

**Motion Language:**
- Smooth, warm, organic (no harsh bounces or robotic linear moves)
- Duration: 200–400ms for micro, 500–800ms for page transitions
- Easing: `ease-in-out` or `ease-out` (never linear)

**Signature Animations:**
- Steam rising from hot drinks (loop SVG animation)
- Cart icon bounces when item added
- Heart fills with caramel color on favorite toggle
- Confetti burst on order placement (coffee-brown + caramel palette)
- Skeleton loaders in warm cream tones (not gray)
- Buttons: subtle scale-105 on hover

**Page Transitions:**
- Fade + slight slide up (translate-y-2)
- Never hard cuts

---

### 9. ACCESSIBILITY (WCAG AA)

- Color contrast minimum 4.5:1 for body text, 3:1 for large text
- All interactive elements have focus ring (caramel color)
- ARIA labels on icon-only buttons
- Keyboard navigation fully supported
- Alt text on all visual elements
- Never rely on color alone to convey meaning

---

### 10. DO'S AND DON'TS SUMMARY

✅ **DO:**
- Keep it warm, premium, artisan
- Use serif for headings to feel handcrafted
- Use cream backgrounds instead of white
- Add small delightful details (steam, beans, warm messages)
- Write copy like a friendly barista, not a salesperson

❌ **DON'T:**
- Use cold grays, blues, or harsh whites
- Use tech-startup gradients (purple/blue)
- Use sans-serif for big headings (loses craft feel)
- Write aggressive marketing copy
- Over-animate — keep it subtle
- Use sharp corners or hard geometric shapes

---

## 🛠 TECH STACK

- React with functional components and hooks (useState, useReducer, useContext)
- Tailwind CSS (core utility classes only)
- `lucide-react` for all icons
- Everything in a single `.jsx` file
- Mock data only — no backend, all state in memory
- NO localStorage/sessionStorage — use React state only

---

## 📄 PAGES TO BUILD (Navigate via internal state)

### 1. HOME PAGE
- Hero banner with tagline "Crafted with Love, Brewed to Perfection" + CTA "Order Now"
- Time-based greeting ("Good Morning! Start your day with...")
- Promo banner — "20% off first order, code: BREW20"
- Category grid (7 categories with icons): Espresso, Hot Coffee, Cold Brew, Frappuccinos, Tea, Pastries, Beans
- Bestsellers carousel — 6 top items with visuals, ratings, prices, Add-to-Cart
- New Arrivals / Seasonal Specials section
- Our Story block with warm copy
- Customer reviews — 3–4 testimonials with avatars and star ratings
- Newsletter signup
- Footer — links, social icons, store hours, contact

### 2. MENU / ITEMS PAGE
- Left sidebar filters: Category checkboxes, price range slider, Temperature (Hot/Cold/Iced), Dietary tags (Vegan, Gluten-free, Dairy-free), Minimum rating
- Top bar: search input + sort dropdown (Popular, Price ↑, Price ↓, Rating, New)
- Product grid cards: visual, name, short description, price, star rating + review count, heart/favorite toggle, Add-to-Cart button, hover "Quick View"
- Smooth empty-state when no results match filters

### 3. PRODUCT DETAIL (Modal)
- Large product visual
- Name, full description, price, rating
- Customization options: Size (S/M/L with price deltas), Milk (Whole/Skim/Almond/Oat/Soy), Sweetness (None/Light/Medium/Extra), Temperature (Hot/Iced), Extra espresso shots (+$0.75), Add-ons (Whipped cream, Cinnamon, Chocolate drizzle, Caramel)
- Quantity selector
- Total price updates live
- Add to Cart button
- Related products section

### 4. CART (Slide-over drawer)
- List items with visual, name, customizations, quantity controls, line total
- Remove item
- Subtotal, tax (8%), delivery fee, promo discount, grand total
- Promo code input with validation
- Proceed to Checkout button
- Empty cart state with friendly illustration and message

### 5. CHECKOUT (Multi-step stepper)
- Step 1: Delivery info — name, phone, full address, delivery notes
- Step 2: Delivery method — Pickup vs Delivery, time-slot selection
- Step 3: Payment (see page 6)
- Step 4: Review & Confirm
- Sticky order summary sidebar throughout
- Form validation with inline errors
- Guest checkout option

### 6. PAYMENT PAGE
- Payment methods: Credit/Debit Card (auto-format + card-type detection, cardholder name, expiry MM/YY, CVV), UPI (GPay/PhonePe/Paytm), Net Banking, Cash on Delivery, Wallet/Loyalty Points
- Apply loyalty points toggle
- Secure SSL badge
- Final total breakdown
- Pay Now button with loading spinner → success

### 7. ORDER CONFIRMATION
- Animated success checkmark + confetti
- Order number (random)
- Estimated delivery/pickup time
- Full order summary
- Track Order and Continue Shopping buttons

### 8. ORDER TRACKING
- Visual horizontal progress tracker: Order Placed → Preparing → Ready → Out for Delivery → Delivered
- Animated progress (simulate with `setInterval`)
- Order details + Contact Support button

### 9. USER PROFILE / ACCOUNT
- Personal info section
- Order history with Reorder buttons
- Saved addresses
- Loyalty points progress bar ("3 more orders to a free drink!")
- Favorites / Wishlist tab

### 10. PERSISTENT NAVIGATION
- Sticky top nav: Logo, Home, Menu, About, Contact, Search, Favorites (heart with count), Cart (bag icon with badge), User
- Cart icon bounces when item added
- Mobile hamburger menu

---

## 📦 SAMPLE DATA (build at least 24 products)

**Espressos:** Classic Espresso, Double Espresso, Macchiato, Cortado
**Hot Coffee:** Americano, Cafe Latte, Cappuccino, Mocha, Flat White
**Cold Brew:** Nitro Cold Brew, Vanilla Cold Brew, Coffee Tonic
**Frappuccinos:** Caramel Frappe, Mocha Frappe, Java Chip, Vanilla Bean
**Seasonal:** Pumpkin Spice Latte, Peppermint Mocha
**Tea:** Chai Latte, Matcha Latte, Earl Grey
**Pastries:** Butter Croissant, Blueberry Muffin, Chocolate Cookie, Cheesecake Slice
**Beans:** Ethiopian Yirgacheffe, Colombian Supremo, Brazilian Santos

Each product must have: brand-voice name, poetic description ("Notes of dark chocolate and caramel, with a whisper of citrus"), price ($3.50–$24.99), emoji/SVG visual, rating (3.8–5.0), review count, tags (Popular/New/Vegan/Hot/Iced), dietary flags.

---

## ⚙️ FUNCTIONALITY REQUIREMENTS (MUST ALL WORK)

- Add to cart, update quantity, remove — fully working
- Filters and search filter the grid in real-time
- Favorites toggle with heart fill animation (caramel color)
- Promo code `BREW20` → 20% off, `COFFEE10` → $10 off (else "Oops, that code doesn't brew right")
- Checkout form validation with on-brand error messages
- Payment form validation (card number formatting, expiry, CVV)
- Toast notifications ("Added to your cup!", "Order brewing!")
- Loading states with branded copy ("Brewing..." not "Loading...")
- Responsive on mobile + desktop

---

## 🎁 DELIVERABLE

A single React artifact fully functional end-to-end:
**Browse → Customize → Cart → Checkout → Payment → Confirmation → Tracking**

Every screen, every button, every microcopy line must reflect the BrewHaven brand — warm, premium, artisan, human. Surprise me with creative details that honor the brand. Make it feel like a real coffee brand I'd fall in love with.

## 🎯 PROMPT END

---

## 💡 HOW TO USE THIS PROMPT

1. **Copy everything** between "PROMPT START" and "PROMPT END"
2. **Paste into Claude** (best results with Opus 4.7)
3. **Wait** for the artifact to render — it will be long
4. **Iterate:** if something is missing, ask "add [feature]" or "make the [section] more on-brand"

## 🔧 CUSTOMIZATION TIPS

- Change **brand name** "BrewHaven" to your own
- Change **currency** `$` → `₹` for Indian Rupees
- Add **local coffee names** (Filter Coffee, Madras Kaapi, Sulaimani Chai)
- Swap **payment methods** (Razorpay, Paytm, UPI prominent for India)
- Ask Claude for **dark mode** variant after the first build
- Ask for **admin dashboard** as a separate artifact afterwards

## 🎨 BRAND CUSTOMIZATION TIPS

Want to tweak the brand for your own shop? Change these sections:

- **Brand Name & Tagline** → Replace "BrewHaven" and "Crafted with Love..."
- **Color Palette** → Swap hex codes (keep the 60/30/10 ratio)
- **Voice & Tone** → Adjust writing examples to your local language/culture
- **Logo Concept** → Describe your actual logo if you have one
- **Sample Data** → Use your actual menu items

## 🚀 FOLLOW-UP PROMPTS (After first build)

- "Add a dark mode toggle with warm night colors — espresso black bg with caramel accents"
- "Add an admin dashboard to manage products and view orders"
- "Make it a Progressive Web App with installable manifest"
- "Add a barista live-chat widget"
- "Connect to a mock backend using the Anthropic API for smart coffee recommendations"
- "Add a loyalty tier system — Bronze Bean, Silver Steam, Golden Roast"
