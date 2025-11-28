# 2400080089-skill-end-sem-exam-
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Protected Dashboard - Demo</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 0; padding: 0; background:#f7f8fa; color:#222; }
    header { background:#0b69ff; color:white; padding:12px 18px; display:flex; justify-content:space-between; align-items:center; }
    nav a { color:white; margin-right:12px; text-decoration:none; font-weight:600; }
    .container { max-width:760px; margin:30px auto; padding:20px; background:white; border-radius:8px; box-shadow:0 6px 18px rgba(0,0,0,0.06); }
    .hidden { display:none; }
    input[type="text"], input[type="password"] { width:100%; padding:8px; margin-top:6px; box-sizing:border-box; }
    label { font-weight:600; }
    button { margin-top:12px; padding:8px 14px; border:0; background:#0b69ff; color:white; border-radius:6px; cursor:pointer; }
    .muted { color:#666; font-size:0.95rem; }
    .small { font-size:0.9rem; color:#444; margin-top:8px; }
    footer { text-align:center; color:#888; padding:20px 0; font-size:0.9rem; }
  </style>
</head>
<body>
  <header>
    <div><strong>KL University Portal</strong></div>
    <nav>
      <a href="#home" id="nav-home">Home</a>
      <a href="#dashboard" id="nav-dashboard">Dashboard</a>
      <a href="#login" id="nav-login">Login</a>
    </nav>
  </header>

  <main class="container">
    <!-- Public Home -->
    <section id="page-home" class="">
      <h2>Welcome (Public)</h2>
      <p class="muted">This is the public home page. Please login to access the student dashboard.</p>
      <p><a href="#login">Click here to Login</a></p>
    </section>

    <!-- Login Page -->
    <section id="page-login" class="hidden">
      <h2>Login</h2>
      <form id="loginForm">
        <div>
          <label for="username">Username</label>
          <input id="username" type="text" autocomplete="username" />
        </div>
        <div style="margin-top:10px;">
          <label for="password">Password</label>
          <input id="password" type="password" autocomplete="current-password" />
        </div>
        <button type="submit">Login</button>
        <p class="small">For demo use any username and password. (Client-side fake auth.)</p>
      </form>
    </section>

    <!-- Dashboard (Protected) -->
    <section id="page-dashboard" class="hidden">
      <h2>Dashboard</h2>
      <p id="welcomeText" class="small"></p>
      <p>This page is protected — only authenticated users can see it.</p>
      <button id="logoutBtn">Logout</button>
    </section>

    <!-- Fallback / 404 -->
    <section id="page-404" class="hidden">
      <h2>Page not found</h2>
      <p class="muted">The page you requested doesn't exist. <a href="#home">Go home</a>.</p>
    </section>
  </main>

  <footer>
    Protected Dashboard Demo — client-side only (not secure for production)
  </footer>

  <script>
    /**********************
     * Simple client-side "auth" implementation
     **********************/
    const AUTH_KEY = "kl_demo_user"; // localStorage key

    function isAuthenticated() {
      return !!localStorage.getItem(AUTH_KEY);
    }

    function loginUser(username) {
      // store minimal info — do not use for real security
      const user = { username, loggedAt: Date.now() };
      localStorage.setItem(AUTH_KEY, JSON.stringify(user));
    }

    function logoutUser() {
      localStorage.removeItem(AUTH_KEY);
    }

    function getUser() {
      const raw = localStorage.getItem(AUTH_KEY);
      return raw ? JSON.parse(raw) : null;
    }

    /**********************
     * Simple hash-based router
     **********************/
    const routes = {
      "": showHome,
      "#": showHome,
      "#home": showHome,
      "#login": showLogin,
      "#dashboard": showDashboard
    };

    // We use this to store where the user wanted to go before forced to login
    let intendedHash = null;

    // page elements
    const pageHome = document.getElementById("page-home");
    const pageLogin = document.getElementById("page-login");
    const pageDashboard = document.getElementById("page-dashboard");
    const page404 = document.getElementById("page-404");
    const navLogin = document.getElementById("nav-login");
    const navDashboard = document.getElementById("nav-dashboard");
    const welcomeText = document.getElementById("welcomeText");

    function hideAll() {
      pageHome.classList.add("hidden");
      pageLogin.classList.add("hidden");
      pageDashboard.classList.add("hidden");
      page404.classList.add("hidden");
    }

    function showHome() {
      hideAll();
      pageHome.classList.remove("hidden");
      updateNav(); 
    }

    function showLogin() {
      // If already logged in, go to dashboard
      if (isAuthenticated()) {
        location.hash = "#dashboard";
        return;
      }
      hideAll();
      pageLogin.classList.remove("hidden");
      updateNav();
      // clear form
      document.getElementById("username").value = "";
      document.getElementById("password").value = "";
    }

    function showDashboard() {
      // Protect this route: if not authenticated redirect to login
      if (!isAuthenticated()) {
        intendedHash = "#dashboard"; // remember where user wanted to go
        location.hash = "#login";
        return;
      }
      hideAll();
      pageDashboard.classList.remove("hidden");
      const user = getUser();
      welcomeText.textContent = `Welcome, ${user?.username || "Student"} — you logged in at ${new Date(user?.loggedAt).toLocaleTimeString()}.`;
      updateNav();
    }

    function show404() {
      hideAll();
      page404.classList.remove("hidden");
      updateNav();
    }

    function route() {
      const h = location.hash || "#home";
      const routeFn = routes[h];
      if (routeFn) routeFn();
      else show404();
    }

    // update nav visibility based on auth
    function updateNav() {
      if (isAuthenticated()) {
        navLogin.style.display = "none";
        navDashboard.style.display = "";
        navLogin.textContent = "Login";
      } else {
        navLogin.style.display = "";
        navDashboard.style.display = "";
      }
    }

    /**********************
     * Login form handling
     **********************/
    document.getElementById("loginForm").addEventListener("submit", (ev) => {
      ev.preventDefault();
      const username = document.getElementById("username").value.trim();
      const password = document.getElementById("password").value;

      if (!username || !password) {
        alert("Enter username and password (demo).");
        return;
      }

      // Demo: accept any credentials
      loginUser(username);

      // After login, redirect to intended or default
      const dest = intendedHash || "#dashboard";
      intendedHash = null;
      location.hash = dest;
    });

    /**********************
     * Logout handling
     **********************/
    document.getElementById("logoutBtn").addEventListener("click", () => {
      logoutUser();
      // After logout always go to login page
      location.hash = "#login";
    });

    /**********************
     * Keep UI in sync when hash changes or on load
     **********************/
    window.addEventListener("hashchange", route);
    window.addEventListener("load", () => {
      // If user navigates to protected hash directly and not auth, router will redirect them to login.
      route();
    });

    // For initial nav state
    updateNav();
  </script>
</body>
</html>
