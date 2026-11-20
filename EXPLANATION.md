# Assignment Explanation: React Routing & Firebase Authentication

This document explains the assignment in simple words: what was asked, how the app is built, and how each part works. It is written for someone new to software development.

---

## 1. What the assignment asks for (in plain English)

Build a small website with **5 pages**:

| Page      | Address (route) | Who can see it               |
|-----------|-----------------|------------------------------|
| Home      | `/`             | Everyone                     |
| Login     | `/login`        | Everyone                     |
| Register  | `/register`     | Everyone                     |
| Dashboard | `/dashboard`    | Only people who are logged in |
| Profile   | `/profile`      | Only people who are logged in |

The website must:

1. **Move between pages** using React Router, with a menu (Navbar) at the top.
2. **Let people sign up, log in, and log out** with an email and password, using Firebase.
3. **Protect** Dashboard and Profile: if you are not logged in and try to open them, you are sent to the Login page.
4. **Show the right menu**: "Login / Register" when logged out, "Logout" when logged in.
5. **Keep you logged in** when you refresh the page.

No database or backend server is needed. Firebase does the account work for us.

### Checklist: requirement → where it is done

| Requirement                                      | Done in file                        | Status |
|--------------------------------------------------|-------------------------------------|--------|
| 5 pages                                          | `src/pages/*.jsx`                   | ✅ |
| React Router used for navigation                 | `src/main.jsx`, `src/App.jsx`       | ✅ |
| Simple Navbar with links                         | `src/components/Navbar.jsx`         | ✅ |
| Register with Email & Password                   | `src/pages/Register.jsx`            | ✅ |
| Login with Firebase                              | `src/pages/Login.jsx`               | ✅ |
| Logout                                           | `Navbar.jsx` and `Profile.jsx`      | ✅ |
| Dashboard & Profile only for logged-in users     | `src/components/ProtectedRoute.jsx` | ✅ |
| Redirect to Login when not logged in             | `ProtectedRoute.jsx` (`<Navigate>`) | ✅ |
| Show Login or Logout based on status             | `Navbar.jsx` (conditional rendering)| ✅ |
| Stay logged in after refresh                     | `src/context/AuthContext.jsx`       | ✅ |
| Dashboard shows "Welcome to Dashboard" + email   | `src/pages/Dashboard.jsx`           | ✅ |
| Profile shows email + Logout button              | `src/pages/Profile.jsx`             | ✅ |

---

## 2. Basic words you need to know

- **React**: a JavaScript library for building web pages out of small pieces called *components*.
- **Component**: a JavaScript function that returns what should appear on screen. Example: `Navbar`, `Login`.
- **Props**: information passed into a component, like arguments to a function. Example: `ProtectedRoute` receives the page it protects as `children`.
- **State (`useState`)**: a value a component remembers, and when it changes the screen updates. Example: what you typed in the email box.
- **Effect (`useEffect`)**: code that runs after the component appears on screen. We use it to start listening to Firebase.
- **Context (`useContext`)**: a way to share one value (here: the logged-in user) with every component, without passing it through each one by hand.
- **Routing**: deciding which page to show based on the URL in the address bar.
- **Authentication**: checking who a user is (sign up, log in, log out).
- **Firebase**: a Google service. We use its *Authentication* part to store accounts and check passwords, so we don't need our own server.

---

## 3. How the app starts (the big picture)

```
index.html
   └── main.jsx
         └── <BrowserRouter>          ← turns routing on
               └── <AuthProvider>     ← knows who is logged in
                     └── <App>
                           ├── <Navbar>            (on every page)
                           └── <Routes>
                                 ├── /           → Home
                                 ├── /login      → Login
                                 ├── /register   → Register
                                 ├── /dashboard  → ProtectedRoute → Dashboard
                                 └── /profile    → ProtectedRoute → Profile
```

1. The browser loads `index.html`, which has an empty `<div id="root">`.
2. `main.jsx` puts our React app inside that div.
3. `BrowserRouter` wraps everything so routing works everywhere.
4. `AuthProvider` wraps everything so every page knows who is logged in.
5. `App` shows the Navbar and picks the page that matches the URL.

---

## 4. How Routing works

### `BrowserRouter` (in `main.jsx`)
Turns on routing. It watches the URL in the address bar.

### `Routes` and `Route` (in `App.jsx`)
A list of "if the URL is X, show page Y":

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/login" element={<Login />} />
  <Route path="/register" element={<Register />} />
  <Route path="/dashboard" element={<ProtectedRoute><Dashboard /></ProtectedRoute>} />
  <Route path="/profile" element={<ProtectedRoute><Profile /></ProtectedRoute>} />
  <Route path="*" element={<h1>404 - Page Not Found</h1>} />
</Routes>
```

`path="*"` catches any address that does not exist.

### `Link` (in `Navbar.jsx`)
`<Link to="/login">Login</Link>` is like an `<a>` tag, but it changes the page **without reloading** the whole website. That makes it fast.

### `useNavigate` (in `Login.jsx`, `Register.jsx`, `Profile.jsx`, `Navbar.jsx`)
Moves to another page **from code**, for example right after a successful login:

```jsx
const navigate = useNavigate();
navigate("/dashboard");
```

### `Navigate` (in `ProtectedRoute.jsx`, `Login.jsx`, `Register.jsx`)
A component that **redirects** as soon as it is shown:

```jsx
return <Navigate to="/login" replace />;
```

`replace` means the page you were redirected *from* is not kept in the browser's back history.

**`useNavigate` vs `Navigate`:** use `useNavigate()` inside a function (like after clicking a button). Use `<Navigate />` when you want to redirect while drawing the page.

---

## 5. How Firebase Authentication works

### Step 1: Connect to Firebase (`src/firebase.js`)

```js
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";

const firebaseConfig = { apiKey: "...", authDomain: "...", projectId: "...", ... };

const app = initializeApp(firebaseConfig); // start Firebase with our project's settings
export const auth = getAuth(app);          // get the Authentication service
```

- `initializeApp` tells Firebase which project to use.
- `getAuth` gives us the `auth` object that every login/logout function needs.
- The web `apiKey` is not a password. It only identifies the project and is meant to be in front-end code. Security is handled by Firebase itself.

### Step 2: Register (`src/pages/Register.jsx`)

```js
await createUserWithEmailAndPassword(auth, email, password);
navigate("/dashboard");
```

Firebase creates the account. It also logs the new user in automatically, so we go straight to the Dashboard. Firebase requires passwords of **at least 6 characters**.

### Step 3: Login (`src/pages/Login.jsx`)

```js
await signInWithEmailAndPassword(auth, email, password);
navigate("/dashboard");
```

Firebase checks the email and password. If they are wrong, it throws an error that we show on screen.

### Step 4: Logout (`Navbar.jsx` and `Profile.jsx`)

```js
await signOut(auth);
navigate("/login");
```

### Step 5: Knowing who is logged in (`src/context/AuthContext.jsx`)

```js
useEffect(() => {
  const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
    setUser(currentUser);   // a user object, or null if logged out
    setLoading(false);
  });
  return unsubscribe;
}, []);
```

`onAuthStateChanged` is a **listener**. Firebase calls it:
- once when the page first loads, and
- every time someone logs in or logs out.

We save the result in state (`user`) and share it through Context. Any component can then do:

```js
const { user } = useAuth();
```

### Error handling (`src/utils/errorMessages.js`)
Firebase returns codes like `auth/invalid-credential`. We turn them into simple messages such as *"Wrong email or password."* Inside Login/Register we use `try { ... } catch (err) { ... }` to catch these errors.

---

## 6. How Protected Routes work

`src/components/ProtectedRoute.jsx`:

```jsx
function ProtectedRoute({ children }) {
  const { user } = useAuth();
  if (!user) {
    return <Navigate to="/login" replace />;   // not logged in → go to Login
  }
  return children;                              // logged in → show the page
}
```

And in `App.jsx`:

```jsx
<Route path="/dashboard" element={<ProtectedRoute><Dashboard /></ProtectedRoute>} />
```

Think of `ProtectedRoute` as a **security guard at a door**:
- The page inside it (`<Dashboard />`) is passed in as a prop called `children`.
- The guard checks: is there a `user`?
  - **Yes** → let them in (show `children`).
  - **No** → send them to `/login`.

---

## 7. How "stay logged in after refresh" works

1. When you log in, Firebase automatically saves your login in the browser's storage.
2. When you refresh, the React app starts from zero, so at first it doesn't know who you are.
3. `AuthProvider` sets `loading = true` and shows "Loading...".
4. Firebase reads the saved login and calls `onAuthStateChanged` with your user.
5. We set `user` and `loading = false`, and now the app is shown.

**Why `loading` matters:** without it, the first moment after refresh would have `user = null`. `ProtectedRoute` would then wrongly send you to `/login` even though you are logged in. Waiting for Firebase's first answer prevents this.

---

## 8. Showing Login or Logout (Conditional Rendering)

In `Navbar.jsx`:

```jsx
{user ? (
  <>
    <Link to="/dashboard">Dashboard</Link>
    <Link to="/profile">Profile</Link>
    <button onClick={handleLogout}>Logout</button>
  </>
) : (
  <>
    <Link to="/login">Login</Link>
    <Link to="/register">Register</Link>
  </>
)}
```

`condition ? A : B` means "if condition is true show A, otherwise show B". This is **conditional rendering**. Because `user` comes from Context, the Navbar updates by itself the moment you log in or out.

Extra: if a logged-in user opens `/login` or `/register`, they are sent to `/dashboard`, since logging in twice doesn't make sense.

---

## 9. What happens step by step (user journeys)

**A. New user registers**
1. Opens `/register`, types email + password, clicks **Register**.
2. `createUserWithEmailAndPassword` creates the account in Firebase.
3. `onAuthStateChanged` fires → `user` is set.
4. The app goes to `/dashboard` → "Welcome to Dashboard" + email.
5. Navbar now shows Dashboard, Profile, Logout.

**B. Logged-out user types `/dashboard`**
1. `ProtectedRoute` sees `user` is `null`.
2. `<Navigate to="/login" />` redirects to the Login page.

**C. Logged-in user refreshes**
1. "Loading..." appears for a moment.
2. Firebase restores the login → user stays on the same page.

**D. User logs out (from Profile or Navbar)**
1. `signOut(auth)` runs → `user` becomes `null`.
2. The app goes to `/login`; Navbar shows Login and Register again.
3. Opening `/profile` now redirects to `/login`.

---

## 10. React concepts used (and where)

| Concept               | Where it is used                                                   |
|-----------------------|--------------------------------------------------------------------|
| `useState`            | Email/password/error inputs in Login & Register; `user` in AuthContext |
| `useEffect`           | Start the `onAuthStateChanged` listener in AuthContext             |
| `useContext`          | `useAuth()` hook in AuthContext, used by Navbar, pages, ProtectedRoute |
| Conditional rendering | Navbar (Login vs Logout), Home message, error messages, loading screen |
| Components & Props    | Every page is a component; `ProtectedRoute` receives `children` as a prop |

---

## 11. How to run and test it

```bash
npm install
npm run dev
```

Open http://localhost:5173.

**Firebase setup (only once):** Firebase Console → project **basic-auth-d8a32** → **Build → Authentication → Get started** → **Sign-in method** → **Email/Password** → **Enable** → **Save**.

**Manual test list:**
1. Home page shows "Welcome to Home Page"; Navbar shows Home, Login, Register.
2. Type `/dashboard` in the address bar → you land on Login.
3. Register with a new email and a 6+ character password → you land on Dashboard with your email.
4. Refresh → you are still logged in.
5. Click Profile → your email and a Logout button.
6. Click Logout → you land on Login; Navbar shows Login and Register.
7. Try logging in with a wrong password → "Wrong email or password."
8. Log in with the right password → Dashboard.

---

## 12. Possible viva / oral questions with short answers

- **What is React Router?** A library that shows different components depending on the URL, without reloading the page.
- **Difference between `<a>` and `<Link>`?** `<a>` reloads the whole website; `<Link>` only swaps the page component, which is faster and keeps app state.
- **What does `onAuthStateChanged` do?** It listens for login/logout and tells us the current user. It also restores the user after a refresh.
- **How do you protect a route?** Wrap the page in `ProtectedRoute`. It checks `user`; if there is none it returns `<Navigate to="/login" />`.
- **Why the `loading` state?** So the app waits for Firebase to restore the login before deciding to redirect. Otherwise logged-in users would be kicked to Login on refresh.
- **Why use Context?** So every component can read the logged-in user without passing it through props on every level.
- **Where are passwords stored?** In Firebase, not in our code. Firebase stores them securely; we never see them.
- **Is the Firebase API key a secret?** No. A web API key only identifies the project. Access is controlled by Firebase's settings and rules.
