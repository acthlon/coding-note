# Enterprise Frontend Engineering Blueprint & Fast-Track Master Curriculum
**Author:** Principal Software Engineer & Systems Architect  
**Target Standard:** Top 1% Global Practitioner (Industrial Rating: 8.5/10 General, 9.8/10 Junior/Intern Benchmark)  
**Stack:** React 19, React Router, Tailwind CSS v4, Vite, Axios, Django REST Framework Backend  
**Scope:** Universal Blueprint for Marvelam & Any High-Performance Web Application

---

## 1. Foundational Axioms & The Python-to-React Bridge

### Core Engineering Axioms
1. **The React Law:** The user interface is a pure mathematical projection of data state:
   $$f(\text{state}) = \text{UI}$$
   You never manipulate DOM elements directly. You mutate state, and the React reconciliation engine computes the minimal DOM diff in $O(N)$ time.
2. **Immutability Invariant:** Never mutate state in-place (`state.property = x` is strictly prohibited). Always produce a new memory reference:
   $$\text{setState}(\text{prev} \Rightarrow (\{ \dots\text{prev}, \text{property}: x \}))$$
3. **Single Source of Truth (SSOT):** State belongs in the highest common ancestor or a global store. If two components rely on the same value, it must never be duplicated.

### The Python / Django Mental Translation
| Django / Python Concept | React 19 Production Equivalent | Architectural Note |
| :--- | :--- | :--- |
| Model Instances / Dicts | `useState` / `useReducer` | Reactive state; re-evaluates the view upon reference change. |
| `{% for item in items %}` | `items.map(item => <Card key={item.id} />)` | Requires unique, stable $O(1)$ lookup key (`item.id`). Never use array index. |
| `{% if condition %}` | `{condition ? <ActiveView /> : <Fallback />}` | Declarative short-circuiting and ternary operators. |
| Django Middleware | Axios Request / Response Interceptors | Intercepts HTTP traffic globally for token injection and error normalization. |
| Model Properties (`@property`) | `useMemo(() => computeSubtotal(cart), [cart])` | Amortized $O(1)$ cached derivations; recalculates only when dependencies mutate. |
| Form Validation (`forms.Form`) | Client-Side Zod Schemas | Strict type inference and schema validation before network dispatch. |

---

## 2. Complete Application Topology & Route Matrix

The entire application runs on **14 core page components**, **1 layout container**, and **2 global overlays**, cleanly delivering **21 distinct user routes**.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                   AppLayout Shell                                      │
│                Header (Nav, Cart Trigger, Auth Badge) ── Footer (Links, Policies)      │
│                Global Overlays: QuickViewModal ── ToastNotificationContainer           │
├───────────────────────────┬───────────────────────────┬────────────────────────────────┤
│ 1. Transactional Core     │ 2. Account & Identity     │ 3. Institutional & Regulatory  │
├───────────────────────────┼───────────────────────────┼────────────────────────────────┤
│ /                         │ /login                    │ /about                         │
│ /catalog                  │ /register                 │ /contact                       │
│ /ready-made               │ /forgot-password          │ /terms                         │
│ /fabrics                  │ /reset-password/:token    │ /privacy-policy                │
│ /new-arrivals             │ /account                  │ /refund-policy                 │
│ /sales                    │ /orders                   │ /shipping-policy               │
│ /product/:slug            │ /orders/:orderId          │ /* (404 Error Boundary)        │
│ /cart                     │ /track                    │                                │
│ /checkout                 │                           │                                │
│ /order-success/:orderId   │                           │                                │
└───────────────────────────┴───────────────────────────┴────────────────────────────────┘
```

---

## 3. The 6 Mastery Pillars

### Pillar I: React 19 Core Mechanics & Structural Hooks
1. **`useState`**: Primitive and object state. Pass updater function (`prev => ...`) when next state depends on prior state.
2. **`useEffect`**: Side effects, API calls, and cleanup functions. Master dependency arrays; clean up timers and abort controllers.
3. **`useCallback` & `useMemo`**: Preserve function identity and cache complex computations in $O(1)$ amortized memory.
4. **`useRef`**: Store mutable values that persist across renders without triggering a re-render.
5. **Custom Hooks**: Extract stateful business logic away from JSX (`useCart`, `useAuth`, `useSEO`, `useDebounce`).
6. **Error Boundaries**: Catch JavaScript crashes and render graceful fallback states.

### Pillar II: Data Structures & Algorithms in Frontend Systems
1. **Hash Map Cart Store ($O(1)$ Optimization)**:
   - Key-Value Hash Map `Record<string, CartItem>` replaces $O(N)$ linear scans with $O(1)$ lookups, updates, and removals.
   - Deriving subtotal uses $O(U)$ aggregation across unique items:
     $$\text{Total} = \sum_{k \in \text{keys}} (\text{cart}[k].\text{qty} \times \text{cart}[k].\text{unitPrice})$$
2. **Debouncing ($O(1)$ Network Optimization)**:
   - Delays search execution by 300ms to eliminate $O(N)$ unnecessary HTTP calls while typing.

### Pillar III: Routing Architecture (React Router)
1. **Layout Routing**: Utilize `<Outlet />` inside `AppLayout` so `<Header />` and `<Footer />` render once without remounting.
2. **Dynamic Segments**: Extract URL params (`/product/:slug`, `/orders/:orderId`) via `useParams()`.
3. **URL Search Parameters (`useSearchParams`)**: Store catalog filter states (`?category=lace&page=2`) in the URL for bookmarking.

### Pillar IV: Enterprise Networking, Auth & Security
1. **Centralized Axios Gateway**: Base instance configured with timeout limits and error normalization.
2. **Silent Token Refresh Loop**:
   - Request Interceptor: Injects memory-held Bearer token.
   - Response Interceptor: Catches `401 Unauthorized`, pauses queue, hits `/api/v1/auth/token/refresh/` using HttpOnly cookie, updates memory token, retries original request.
3. **Security Invariants**:
   - Access tokens strictly in memory; refresh tokens in `HttpOnly`, `SameSite=Lax`, `Secure` cookies. Zero raw JWTs in `localStorage`.

### Pillar V: Industrial SEO & Discovery (Vite SPA)
1. **Semantic HTML:** `<header>`, `<main>`, `<article>`, `<section>`, `<nav>`, `<footer>`.
2. **Dynamic Meta Tags (`useSEO` hook):** Dynamically updates document title and OpenGraph tags per page.
3. **Structured Data (JSON-LD):** Dynamically mounts `<script type="application/ld+json">` with Schema.org Product specifications.

### Pillar VI: Tailwind CSS v4 & Layout Systems
Master the 4 structural models:
1. **Flexbox:** `flex flex-col`, `flex items-center justify-between`, `gap-4`.
2. **CSS Grid:** `grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6`.
3. **Stacking Contexts:** `relative`, `absolute`, `z-50`.
4. **Interactive States:** `hover:`, `focus:`, `active:`, `disabled:opacity-50`.

---

## 4. The Master AI Prompt Engineering Templates

### Component Generation Template
```markdown
Act as a Principal Frontend Engineer. Write a production-ready React 19 component using Tailwind CSS v4.

COMPONENT REQUIREMENTS:
- Component Name: [e.g., OrderSummary]
- Purpose: [e.g., Displays subtotal, shipping, coupon discount, and Paystack trigger button]
- Props Interface:
  * items: Record<string, CartItem>
  * onCheckout: () => Promise<void>
  * isLoading: boolean

ARCHITECTURAL RULES:
1. Zero unnecessary boilerplate. No verbose wrappers, no dead comments.
2. Defensive Validation: Handle empty items state and undefined values gracefully.
3. Accessibility: Use semantic HTML, button states (disabled, aria-busy), and screen-reader considerations.
4. Tailwind Styling: Modern, clean, responsive (mobile-first), professional aesthetics with neutral palette.
5. Provide the exact Time and Space complexity for any derived calculations.
```

---

## 5. Granular, Neuroscience-Backed 40-Day Master Schedule

> **Working Standard:** Every day represents **2 to 3.5 hours of Deep Work**.  
> **Tracer-Bullet Rule:** Step 1: Scaffold (30%) ➔ Step 2: State (40%) ➔ Step 3: Network & Errors (30%).

```
┌────────────────────────────────────────────────────────────────────────┐
│ Sprint 1 (Days 1–8):   Layout Shell, Reactive State Core & Cart Engine │
│ Sprint 2 (Days 9–18):  Polymorphic Catalogs, Dynamic Detail & Filters  │
│ Sprint 3 (Days 19–28): API Gateway, Auth Machine, Security & Accounts  │
│ Sprint 4 (Days 29–35): Checkout Engine, Paystack Gateway & Orders      │
│ Sprint 5 (Days 36–40): Sales Engine, Legal, SEO & Production Deploy    │
└────────────────────────────────────────────────────────────────────────┘
```

### SPRINT 1: Layout Shell, Reactive State Core & Cart Engine (Days 1–8)

#### Day 1: Global Application Shell & Outlet Routing
- **Time:** 2.5 Hours
- **Mental Model:** Layout Routing vs Component Remounting.
- **Tasks:**
  1. Create `src/components/layout/AppLayout.jsx` with `<Header />`, `<Outlet />`, and `<Footer />`.
  2. Refactor `src/App.jsx` to nest all current routes inside `<Route element={<AppLayout />}>`.
  3. Remove local `<Header />` and `<Footer />` calls from `HomePage.jsx` and `DressCatalogPage.jsx`.
- **Verification:** Navigate between `/` and `/ready-made`. Header and Footer remain permanently mounted without flashing or re-rendering.

#### Day 2: Toast Notification Engine (Global Async Overlay)
- **Time:** 2.5 Hours
- **Mental Model:** React Portals and Context-driven Notification Queues.
- **Tasks:**
  1. Create `src/context/ToastContext.jsx` with actions: `showToast(message, type = 'success' | 'error')`.
  2. Build `ToastContainer.jsx` with animated dismiss timers (3000ms auto-dismiss).
  3. Mount `ToastContainer` inside `AppLayout.jsx`.
- **Verification:** Trigger `showToast('Item added to cart', 'success')` from a test button. Notification slides in, stays 3 seconds, and unmounts cleanly from DOM memory.

#### Day 3: Cart Store Core ($O(1)$ Hash Map Architecture)
- **Time:** 3.0 Hours
- **Mental Model:** Hash Table lookup ($O(1)$) vs Array Iteration ($O(N)$).
- **Tasks:**
  1. Create `src/context/CartContext.jsx` using `useReducer`.
  2. Implement state shape: `{ items: Record<string, CartItem>, totalQuantity: number }`.
  3. Implement reducer actions: `ADD_ITEM`, `REMOVE_ITEM`, `UPDATE_QUANTITY`, `CLEAR_CART`.
- **Verification:** Dispatch `ADD_ITEM` for identical item IDs. Ensure item count increments in $O(1)$ time without duplicate dictionary keys.

#### Day 4: Cart State Persistence & Schema Validation
- **Time:** 2.5 Hours
- **Mental Model:** Browser Storage Serialization with Defensive Invariants.
- **Tasks:**
  1. Add an effect to `CartContext` to synchronize state with `localStorage.getItem('marvelam_cart')`.
  2. Implement defensive parsing with fallback: if local data is corrupted, clear storage and initialize `{ items: {} }`.
  3. Expose custom hook: `export const useCart = () => useContext(CartContext)`.
- **Verification:** Add an item to cart, refresh the browser page. Cart items reload from storage with zero data loss.

#### Day 5: Cart Item Line-Item Component & Mutation Guardrails
- **Time:** 2.5 Hours
- **Mental Model:** Event Propagation and Component State Invariants.
- **Tasks:**
  1. Build `src/components/cart/CartItem.jsx` rendering title, thumbnail, price, unit type (yards vs dress size).
  2. Implement quantity increment/decrement controls with guardrail: decrementing below 1 triggers item removal confirmation.
  3. Wire buttons directly to `updateQuantity(id, qty)` and `removeItem(id)` from `useCart()`.
- **Verification:** Increment quantity to 5; verify line subtotal updates instantly. Decrement to 0; item is pruned from the Hash Map.

#### Day 6: Cart Page & Derived Pricing Computations
- **Time:** 3.0 Hours
- **Mental Model:** Memoized Derivations (`useMemo`) vs Redundant State Synchronization.
- **Tasks:**
  1. Refactor `src/pages/cart-page/CartPage.jsx` to consume `useCart()`.
  2. Create `OrderSummary.jsx` computing Subtotal, Estimated Tax (7.5%), and Delivery Fee using `useMemo`.
  3. Implement Empty State illustration with a prominent "Explore Collection" button linking to `/catalog`.
- **Verification:** An empty cart shows the empty illustration; adding items instantly updates subtotal in $O(U)$ amortized time.

#### Day 7: Global Header Badge & Quick Cart Flyout Sync
- **Time:** 2.5 Hours
- **Mental Model:** Derived Global Selectors across Unrelated Components.
- **Tasks:**
  1. Open `src/components/layout/Header.jsx`.
  2. Wire the shopping cart icon to display a dynamic badge with `totalQuantity` from `useCart()`.
  3. Add a pulse animation (`animate-scale`) to the badge whenever an item is added.
- **Verification:** Clicking "Add to Cart" anywhere updates the Header counter immediately without page reload.

#### Day 8: HomePage Architecture Audit & Countdown Timer Refactor
- **Time:** 2.5 Hours
- **Mental Model:** Memory-Safe Interval Timers (`setInterval` cleanup).
- **Tasks:**
  1. Audit `src/pages/homepage/HomePage.jsx` and remove unused mock imports.
  2. Inspect `CountDownBlock.jsx`: verify `clearInterval` is called inside `useEffect` cleanup to prevent memory leaks.
  3. Run `npm run build` to verify zero build errors in Sprint 1.
- **Verification:** Console displays zero warnings. Navigating between Home and Cart runs with 60fps performance.

---

### SPRINT 2: Polymorphic Catalogs, Dynamic Detail & Filters (Days 9–18)

#### Day 9: Polymorphic Catalog Engine Scaffold
- **Time:** 2.5 Hours
- **Mental Model:** Parameterized View Engines (DRY Principle).
- **Tasks:**
  1. Create `src/pages/catalog/CatalogPage.jsx` accepting `title`, `endpoint`, and `bannerImage` as props.
  2. Structure layout: `PageHero` ➔ Main Content (`Sidebar` + `ToolBarAndGrid`).
  3. Configure route parameters in `App.jsx` to map `/ready-made`, `/fabrics`, and `/new-arrivals` to `CatalogPage`.
- **Verification:** Visiting `/fabrics` and `/ready-made` renders the correct titles using one single component codebase.

#### Day 10: Django Product Model Integration & Skeleton Loader
- **Time:** 3.0 Hours
- **Mental Model:** Asynchronous State Machine: `idle` | `loading` | `success` | `error`.
- **Tasks:**
  1. Add Axios fetch logic inside `CatalogPage.jsx` hitting the parameterized `endpoint`.
  2. Build `ProductCardSkeleton.jsx` using Tailwind pulse animation (`animate-pulse`).
  3. Render 8 skeleton cards while `isLoading` is true.
- **Verification:** Throttle network to "Slow 3G" in DevTools. Grid displays elegant animated skeletons before product cards appear.

#### Day 11: URL Search Parameters Synchronization (`useSearchParams`)
- **Time:** 3.0 Hours
- **Mental Model:** URL-as-Single-Source-of-Truth for Shareable Filters.
- **Tasks:**
  1. Integrate `useSearchParams` in `CatalogPage.jsx`.
  2. Synchronize URL query parameters (`?category=lace&sort=price_desc&page=1`) with component state.
  3. Ensure pressing the browser "Back" button restores previous filter selections.
- **Verification:** Copy the URL with filters, open in a new private browser tab. The catalog loads with identical filters pre-applied.

#### Day 12: Faceted Sidebar Filter Wiring & Price Debounce
- **Time:** 3.0 Hours
- **Mental Model:** Event Throttling & Debouncing ($O(1)$ Network Optimization).
- **Tasks:**
  1. Refactor `src/components/catalog/SideBar.jsx` to receive `filters` and `onFilterChange`.
  2. Implement a 300ms debounce timer on the price range slider before updating URL search params.
  3. Connect category checkboxes and rating filter pills.
- **Verification:** Drag price slider rapidly. Only one final API query triggers after sliding stops.

#### Day 13: Dynamic Product Detail Route (`/product/:slug`)
- **Time:** 2.5 Hours
- **Mental Model:** Dynamic URL Parameters (`useParams`).
- **Tasks:**
  1. Create `src/pages/product-detail/ProductDetailPage.jsx`.
  2. Register route `/product/:slug` inside `App.jsx`.
  3. Extract `slug` via `useParams()` and execute `GET /api/v1/products/${slug}/`.
- **Verification:** Navigating to `/product/african-luxury-lace` queries the specific product record from the backend.

#### Day 14: Product Image Gallery & Active Thumbnail State
- **Time:** 2.5 Hours
- **Mental Model:** Local Component State for UI Selection.
- **Tasks:**
  1. Build image gallery container: 1 large active display image, horizontal row of clickable thumbnail previews.
  2. State: `const [activeImage, setActiveImage] = useState(product.images[0])`.
  3. Add hover zoom effect using Tailwind `transition-transform duration-300 hover:scale-105`.
- **Verification:** Clicking any thumbnail updates the main preview instantly with smooth CSS transitions.

#### Day 15: Polymorphic Variant Picker (Fabric Yardage vs Dress Sizing)
- **Time:** 3.0 Hours
- **Mental Model:** Polymorphic Domain Logic based on Item Type.
- **Tasks:**
  1. In `ProductDetailPage`, inspect `product.type`.
  2. If `type === 'fabric'`: Render yardage counter (`[ - ] 5 Yards [ + ]`) with minimum purchase constraint (e.g., minimum 2 yards).
  3. If `type === 'dress'`: Render size buttons (`UK 8`, `UK 10`, `UK 12`, `UK 14`).
- **Verification:** Fabrics show yardage modifiers; dresses show size selection pills. Active selections bind to product payload.

#### Day 16: QuickView Modal Architecture (React Portals)
- **Time:** 3.0 Hours
- **Mental Model:** DOM Stacking Contexts and Event Bubble Isolation.
- **Tasks:**
  1. Create `src/components/common/QuickViewModal.jsx` mounted via `createPortal` to `document.body`.
  2. Wire eye-icon on `ItemCard.jsx` to pass `productId` to modal.
  3. Add keyboard escape listener (`Escape` key closes modal) and outside-click backdrop handler.
- **Verification:** Clicking the eye-icon opens modal without changing URL or losing scroll position; pressing `Escape` dismisses it.

#### Day 17: Production Add-to-Cart Action Button
- **Time:** 2.0 Hours
- **Mental Model:** Micro-State Feedback Machines (`idle` ➔ `loading` ➔ `success`).
- **Tasks:**
  1. Implement `AddToCartButton.jsx` on `ProductDetailPage` and `QuickViewModal`.
  2. Connect to `useCart().addItem(product, quantity)`.
  3. Trigger global toast: `"Added 5 yards of Ankara Wax to your bag"`.
- **Verification:** Button shows spinner for 300ms, transitions to green checkmark, updates header cart badge, and resets to idle.

#### Day 18: Sprint 2 Integration Review & Empty/Error Boundaries
- **Time:** 2.5 Hours
- **Mental Model:** Defensive Programming and Fault Tolerance.
- **Tasks:**
  1. Test invalid product slug (`/product/non-existent-item`). Render clean 404 UI with "Return to Catalog" button.
  2. Audit mobile responsiveness of the filter sidebar (convert to off-canvas drawer on mobile screens).
  3. Commit code: `git commit -m "feat(catalog): complete polymorphic catalog, dynamic product details and quick view"`.
- **Verification:** Catalog and Product views pass mobile responsiveness checks down to 320px viewport width.

---

### SPRINT 3: API Gateway, Auth Machine, Security & Accounts (Days 19–28)

#### Day 19: Centralized Axios Client & Error Normalizer
- **Time:** 2.0 Hours
- **Mental Model:** Inversion of Control in Network Gateways.
- **Tasks:**
  1. Create `src/utils/apiClient.js` with `axios.create({ baseURL, timeout: 10000, withCredentials: true })`.
  2. Build error normalizer function transforming Django REST Framework error dictionaries into plain English strings.
- **Verification:** Deliberately trigger a 400 Bad Request; verify normalized error string prints in console.

#### Day 20: Authentication Context & Auth State Machine
- **Time:** 3.0 Hours
- **Mental Model:** Identity State Machine: `UNAUTHENTICATED` | `AUTHENTICATING` | `AUTHENTICATED`.
- **Tasks:**
  1. Create `src/context/AuthContext.jsx`.
  2. State: `{ user: null, accessToken: null, isAuthenticated: false, isLoading: true }`.
  3. Expose methods: `login(credentials)`, `logout()`, `register(userData)`.
- **Verification:** Mounting application runs initialization check against `/api/v1/auth/me/` to verify session.

#### Day 21: Axios Response Interceptor (Silent 401 Refresh Loop)
- **Time:** 3.5 Hours
- **Mental Model:** Asynchronous Queueing and Transparent Retry Loops.
- **Tasks:**
  1. In `apiClient.js`, attach response interceptor catching `error.response.status === 401`.
  2. If 401: pause queue, execute `POST /api/v1/auth/token/refresh/`, update memory token, retry failed request.
  3. If refresh fails: dispatch `LOGOUT` and redirect to `/login`.
- **Verification:** Set mock access token expiry to 5 seconds. Send API call; observe automatic silent token refresh without UI disruption.

#### Day 22: Sign In Page (`LoginPage.jsx`)
- **Time:** 2.5 Hours
- **Mental Model:** Controlled Form State and Redirect Preservation.
- **Tasks:**
  1. Create `src/pages/auth/LoginPage.jsx`.
  2. Form fields: Email, Password, "Remember Me" checkbox.
  3. Implement post-login redirect using `useLocation().state?.from || '/account'`.
- **Verification:** Attempting to view `/account` redirects to `/login`; signing in returns user directly back to `/account`.

#### Day 23: Sign Up Page (`SignUpPage.jsx`) & Password Entropy
- **Time:** 2.5 Hours
- **Mental Model:** Real-Time Client Validation and Defensive Form Feedback.
- **Tasks:**
  1. Create `src/pages/auth/SignUpPage.jsx`.
  2. Form fields: Full Name, Email, Password, Confirm Password.
  3. Build real-time password strength meter (Length >= 8, Contains Number, Contains Special Character).
- **Verification:** Submit button disabled until password satisfies security criteria; submitting creates user in Django backend.

#### Day 24: Password Recovery Flow (`ForgotPassword` & `ResetPassword`)
- **Time:** 2.5 Hours
- **Mental Model:** One-Time Token Lifecycle Management.
- **Tasks:**
  1. Create `src/pages/auth/ForgotPasswordPage.jsx` requesting reset email.
  2. Create `src/pages/auth/ResetPasswordPage.jsx` reading token from `/reset-password/:token`.
  3. Connect to Django password reset endpoints.
- **Verification:** Submitting valid email displays confirmation banner; submitting mismatched passwords shows instant validation error.

#### Day 25: Protected Route Barrier Component
- **Time:** 2.0 Hours
- **Mental Model:** Declarative Access Control Wrappers.
- **Tasks:**
  1. Create `src/components/auth/ProtectedRoute.jsx`.
  2. If `isLoading`: render full-screen loading spinner.
  3. If `!isAuthenticated`: return `<Navigate to="/login" state={{ from: location }} replace />`.
  4. If `isAuthenticated`: render `<Outlet />`.
- **Verification:** Direct URL entry to `/account` when logged out immediately navigates to `/login`.

#### Day 26: Account Dashboard & Saved Address Book
- **Time:** 3.0 Hours
- **Mental Model:** Master-Detail Layouts & Sub-Navigation.
- **Tasks:**
  1. Create `src/pages/account/AccountPage.jsx` with sidebar tabs: Profile, Orders, Addresses.
  2. Build Address Book card: default shipping address, secondary address, "Add New Address" modal form.
  3. Connect to `/api/v1/user/addresses/`.
- **Verification:** Adding a new address updates state and marks default shipping destination.

#### Day 27: Django DRF Full Authentication Linkage
- **Time:** 3.0 Hours
- **Mental Model:** Cross-Origin Resource Sharing (CORS) and Credential Transmission.
- **Tasks:**
  1. Verify Django CORS configuration: `CORS_ALLOW_CREDENTIALS = True`.
  2. Test registration, email validation error handling, login, and cookie generation end-to-end.
- **Verification:** Inspect browser DevTools Application tab: access token is **not** in localStorage; refresh cookie is marked `HttpOnly` and `SameSite=Lax`.

#### Day 28: Sprint 3 Security Audit
- **Time:** 2.0 Hours
- **Mental Model:** Threat Modeling: XSS, CSRF, Token Exfiltration.
- **Tasks:**
  1. Inject malicious script `<script>alert('xss')</script>` into user name and review forms; verify React automatically escapes strings.
  2. Confirm no sensitive credentials remain in console logs.
  3. Commit code: `git commit -m "feat(auth): complete secure authentication, silent refresh and protected routes"`.
- **Verification:** Security audit passes; zero credentials exposed in browser history or storage.

---

### SPRINT 4: Checkout Engine, Paystack Gateway & Orders (Days 29–35)

#### Day 29: Checkout Page Architecture & Address Form
- **Time:** 3.0 Hours
- **Mental Model:** Stepped Transactional Workflows.
- **Tasks:**
  1. Create `src/pages/checkout/CheckoutPage.jsx` protected by `ProtectedRoute`.
  2. Build shipping information form: Full Name, Phone, Delivery Address, State/City.
  3. Auto-populate fields if user has a default address saved in `AccountContext`.
- **Verification:** Navigating to `/checkout` while authenticated auto-fills saved shipping details.

#### Day 30: Courier Selection & Dynamic Shipping Rates
- **Time:** 2.5 Hours
- **Mental Model:** Dynamic Cost Calculation based on Geographic Inputs.
- **Tasks:**
  1. Add Delivery Courier radio options: Standard Delivery (Lagos: ₦2,500, Nationwide: ₦5,000), Express Pickup (₦0).
  2. Wire shipping fee calculation to update final order total in real-time.
- **Verification:** Switching between Standard and Express recalculates order total dynamically.

#### Day 31: Paystack Payment Gateway Integration
- **Time:** 3.5 Hours
- **Mental Model:** Third-Party Gateway Bridges & Payment Idempotency.
- **Tasks:**
  1. Install or load Paystack inline script / SDK.
  2. Create checkout submission handler:
     - Step 1: POST order draft to Django `/api/v1/orders/create/` (returns `order_id` and reference).
     - Step 2: Trigger Paystack modal with public key, customer email, and total amount.
     - Step 3: Handle `onSuccess` callback.
- **Verification:** Trigger payment modal in test mode. Completing test payment executes success callback.

#### Day 32: Order Confirmation & Receipt Page (`OrderSuccessPage.jsx`)
- **Time:** 2.5 Hours
- **Mental Model:** Post-Transaction Verification and State Clearing.
- **Tasks:**
  1. Create `src/pages/checkout/OrderSuccessPage.jsx` at `/order-success/:orderId`.
  2. On mount: call `useCart().clearCart()` to empty cart state.
  3. Display printable receipt: Order ID, items purchased, amount paid, delivery estimate, "Track Order" button.
- **Verification:** Cart is automatically empty upon arriving at this page; order receipt renders accurate purchase data.

#### Day 33: Customer Purchase History Table (`OrdersListPage.jsx`)
- **Time:** 2.5 Hours
- **Mental Model:** Tabular Data Display and Status Classification.
- **Tasks:**
  1. Create `src/pages/orders/OrdersListPage.jsx`.
  2. Fetch user order history from `GET /api/v1/orders/`.
  3. Render order list table with status badges (`Pending`: yellow, `Paid`: blue, `Dispatched`: purple, `Delivered`: green).
- **Verification:** Every completed order appears in the list with active link to view detailed receipt.

#### Day 34: Granular Order Detail Invoice (`OrderDetailPage.jsx`)
- **Time:** 2.5 Hours
- **Mental Model:** Relational Deep-Dives via Parametric Routing.
- **Tasks:**
  1. Create `src/pages/orders/OrderDetailPage.jsx` at `/orders/:orderId`.
  2. Render itemized product list, delivery address, shipping courier, payment transaction reference.
  3. Add "Download Receipt" print trigger (`window.print()`).
- **Verification:** Clicking an order from the orders table navigates to this view and renders the full invoice.

#### Day 35: Public Order Tracking Lookup (`TrackPage.jsx`)
- **Time:** 2.5 Hours
- **Mental Model:** Public Lookup Interfaces without Authentication Requirement.
- **Tasks:**
  1. Refactor `src/pages/track-page/TrackPage.jsx`.
  2. Build tracking input form accepting `order_id` or `tracking_code`.
  3. Render step progress bar: `Order Placed` ➔ `Processing` ➔ `Dispatched` ➔ `Delivered`.
- **Verification:** Entering a valid tracking code displays current fulfillment progress bar.

---

### SPRINT 5: Sales Engine, Legal, SEO & Production Deployment (Days 36–40)

#### Day 36: Dedicated Flash Sales Page (`SalesPage.jsx`)
- **Time:** 3.0 Hours
- **Mental Model:** Urgency-Driven Conversion Design.
- **Tasks:**
  1. Create `src/pages/sales/SalesPage.jsx` at `/sales`.
  2. Build Flash Sale countdown header banner (e.g., "Mega Year-End Clearance Ends in 14h 22m 10s").
  3. Product cards include discount pills (`-35% OFF`), original strikethrough price, and stock progress meters (`8 left`).
- **Verification:** Urgency tickers count down every second; items render promotional pricing correctly.

#### Day 37: Polymorphic Legal & Regulatory Engine (`LegalPolicyPage.jsx`)
- **Time:** 2.5 Hours
- **Mental Model:** Parameterized Static Content Delivery.
- **Tasks:**
  1. Create `src/pages/legal/LegalPolicyPage.jsx`.
  2. Register routes: `/terms`, `/privacy-policy`, `/refund-policy`, `/shipping-policy`.
  3. Render official clauses matching Paystack merchant verification standards.
- **Verification:** All 4 policy routes display accurate legal information; links in footer navigate correctly.

#### Day 38: Brand Story (`AboutUsPage`) & Customer Support (`ContactUsPage`)
- **Time:** 2.5 Hours
- **Mental Model:** Trust Signals and Real-Time Customer Communication Channels.
- **Tasks:**
  1. Create `src/pages/about/AboutUsPage.jsx` showcasing brand artisan craftsmanship.
  2. Create `src/pages/contact/ContactUsPage.jsx` with contact form, business address, and direct WhatsApp chat bridge (`https://wa.me/...`).
- **Verification:** Clicking WhatsApp bridge opens chat directly with business phone number.

#### Day 39: Dynamic SEO (`useSEO.js`) & JSON-LD Structured Data
- **Time:** 3.0 Hours
- **Mental Model:** Search Engine Indexability and OpenGraph Sharing Protocols.
- **Tasks:**
  1. Create `src/utils/useSEO.js` hook.
  2. Attach `useSEO` across `HomePage`, `CatalogPage`, and `ProductDetailPage`.
  3. In `ProductDetailPage`, inject Schema.org JSON-LD structured product metadata.
  4. Build `src/pages/not-found/NotFoundPage.jsx` for 404 boundaries.
- **Verification:** Inspect `<head>` in Chrome DevTools: `<title>`, meta descriptions, and `<script type="application/ld+json">` update per product.

#### Day 40: Production Bundle Splitting, Audit & Vercel Deployment
- **Time:** 3.5 Hours
- **Mental Model:** Zero-Downtime Edge Deployment and Performance Profiling.
- **Tasks:**
  1. Implement route-level code splitting in `App.jsx` using `React.lazy` and `<Suspense>`.
  2. Run `npm run build` and inspect Vite output; verify no single chunk exceeds 200KB.
  3. Run 20-point production readiness audit checklist.
  4. Push codebase to GitHub and link to Vercel/Cloudflare Pages for continuous deployment.
- **Verification:** Production URL is live on the internet, secure under HTTPS, with 90+ Lighthouse score and zero console errors.

---

## 6. The 20-Point Production Readiness Audit (The 8.5/10 Standard)

- [ ] 1. **No Duplicate Pages:** Catalog pages share the single parameterized `CatalogPage` component.
- [ ] 2. **Single Layout Shell:** `<Header />` and `<Footer />` mount once via `AppLayout` and `<Outlet />`.
- [ ] 3. **$O(1)$ Cart Lookups:** Cart operations use key-value Hash Maps, not array linear scans.
- [ ] 4. **Derived State Memoization:** Subtotals and taxes use `useMemo`; no redundant state sync bugs.
- [ ] 5. **Clean Route Boundaries:** 404 catch-all (`*`) exists and handles invalid routes gracefully.
- [ ] 6. **Zero Raw JWT in LocalStorage:** Access tokens live in memory; refresh tokens in HttpOnly cookies.
- [ ] 7. **Silent Token Refresh:** Axios interceptor catches 401s and refreshes sessions automatically.
- [ ] 8. **Request Sanitization:** All forms validate inputs; no raw unescaped HTML rendered.
- [ ] 9. **Route Protection:** `/checkout`, `/account`, and `/orders` redirect unauthenticated users to `/login`.
- [ ] 10. **Error Normalization:** Network timeouts and 500 errors display user-friendly toast alerts.
- [ ] 11. **Loading Skeletons:** Product catalog displays skeleton cards during API fetches (no blank screens).
- [ ] 12. **Empty State Handling:** Cart, Catalog, and Orders handle zero-data scenarios gracefully.
- [ ] 13. **Debounced Search:** Product searches debounce network calls by 300ms.
- [ ] 14. **Optimistic UI Updates:** Cart item deletion updates instantly with background sync.
- [ ] 15. **Responsive Integrity:** All 21 routes render without horizontal scrollbars on 320px mobile screens.
- [ ] 16. **Dynamic Page Titles:** Browser tab updates dynamically per page (e.g., `Ankara Wax | Marvelam`).
- [ ] 17. **JSON-LD Schema:** Product pages contain valid Schema.org structured data.
- [ ] 18. **Regulatory Compliance:** Public `/privacy-policy` and `/refund-policy` routes are operational.
- [ ] 19. **Bundle Splitting:** Routes are loaded via `React.lazy` to keep initial bundle < 150KB.
- [ ] 20. **Zero Console Warnings:** Clean developer console during normal user navigation.

---

## 7. Operational Workflow, Context Management & AI Collaboration Protocol

### 7.1 Sprint-Boundary Chat Isolation Strategy
To prevent context window degradation and token hallucination ($O(K)$ conversational noise):
- **1 Dedicated Chat per Sprint:** Maintain one chat thread for each 8–10 day sprint (Sprint 1 Chat, Sprint 2 Chat, etc.).
- **Context Priming on New Chats:** When starting a fresh sprint chat, simply link or provide the path to this master blueprint. The AI immediately inherits all architectural invariants, routes, and naming conventions without re-explaining the stack.
- **Context Compaction:** If an active sprint chat becomes sluggish or nears memory limits, trigger the `/compact` command to prune transient chit-chat while retaining active state decisions.

### 7.2 Design System Continuity & Visual Token Ingestion
To ensure every page and component generated by AI conforms to Marvelam's visual language:
1. **Design System Inheritance:** The AI inspects existing reference components (`Header.jsx`, `ItemCard.jsx`, `index.css`) to extract established typography, neutral color scales, and border radii.
2. **Visual Screenshot Uploads:** When building complex bespoke layouts (like `SalesPage` or `ProductDetailPage`), paste screenshot mockups directly into chat alongside the functional prompt.
3. **The Design Contract Directive:** Enforce this directive in every component request:
   > *"Match the existing Marvelam luxury fashion design system: neutral-900 typography, subtle borders (`border-neutral-200`), clean whitespace (`gap-6`, `py-12`), and high-contrast call-to-action buttons."*

### 7.3 The 4-Step Daily Execution Ritual
Execute this closed-loop cycle every day:
1. **Open Blueprint:** Check today's day number, mental model, and verification criterion.
2. **Command AI:** Dispatch the component/hook prompt using Section 4's master template.
3. **Inspect, Type & Test:** Assemble the code using Tracer-Bullet Slicing (Scaffold ➔ State ➔ Network). Verify zero console errors.
4. **Atomic Git Commit:** Close the daily dopamine loop with an explicit commit:
   ```bash
   git commit -m "feat(<scope>): <deliverable description>"
   ```
   Check off the day's completion box in the blueprint.
