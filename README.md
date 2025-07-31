Here's the full working code for App.js and all related components (LoginRegister.js, Dashboard.js) for your frontend with authentication (JWT + Identity) integrated:


---

1️⃣ src/App.js

import React, { useState } from "react";
import LoginRegister from "./LoginRegister";
import Dashboard from "./Dashboard";

function App() {
  const [isLoggedIn, setIsLoggedIn] = useState(
    !!localStorage.getItem("token")
  );

  const handleLogin = () => setIsLoggedIn(true);

  const handleLogout = () => {
    localStorage.removeItem("token");
    setIsLoggedIn(false);
  };

  return isLoggedIn ? (
    <Dashboard onLogout={handleLogout} />
  ) : (
    <LoginRegister onLoginSuccess={handleLogin} />
  );
}

export default App;


---

2️⃣ src/LoginRegister.js

import React, { useState } from "react";
import axios from "axios";

const API_BASE = "http://localhost:5172/api/controller";

function LoginRegister({ onLoginSuccess }) {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [isRegister, setIsRegister] = useState(false);
  const [error, setError] = useState("");

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError("");

    try {
      if (isRegister) {
        // Call register API
        await axios.post(`${API_BASE}/register`, null, {
          params: { username, password },
        });
        alert("Registration successful! Please login.");
        setIsRegister(false);
      } else {
        // Call login API
        const res = await axios.post(`${API_BASE}/login`, null, {
          params: { username, password },
        });

        if (res.data.token) {
          localStorage.setItem("token", res.data.token);
          onLoginSuccess();
        }
      }
    } catch (err) {
      console.error(err);
      setError(
        err.response?.data || "Something went wrong. Please try again."
      );
    }
  };

  return (
    <div style={styles.container}>
      <h2>{isRegister ? "Register" : "Login"}</h2>
      {error && <p style={styles.error}>{error}</p>}
      <form onSubmit={handleSubmit} style={styles.form}>
        <input
          type="text"
          placeholder="Username"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
          required
          style={styles.input}
        />
        <input
          type="password"
          placeholder="Password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
          style={styles.input}
        />
        <button type="submit" style={styles.button}>
          {isRegister ? "Register" : "Login"}
        </button>
      </form>
      <button
        style={styles.switchButton}
        onClick={() => setIsRegister(!isRegister)}
      >
        {isRegister
          ? "Already have an account? Login"
          : "Don't have an account? Register"}
      </button>
    </div>
  );
}

const styles = {
  container: {
    width: "300px",
    margin: "100px auto",
    textAlign: "center",
    padding: "20px",
    border: "1px solid #ccc",
    borderRadius: "8px",
  },
  form: { display: "flex", flexDirection: "column", gap: "10px" },
  input: { padding: "8px" },
  button: {
    padding: "10px",
    background: "#007bff",
    color: "#fff",
    border: "none",
    cursor: "pointer",
  },
  switchButton: {
    marginTop: "10px",
    background: "none",
    border: "none",
    color: "#007bff",
    cursor: "pointer",
  },
  error: { color: "red" },
};

export default LoginRegister;


---

3️⃣ src/Dashboard.js

import React, { useState, useEffect } from "react";
import axios from "axios";

const API_BASE = "http://localhost:5172/api/controller";

function Dashboard({ onLogout }) {
  const [secureData, setSecureData] = useState("");

  useEffect(() => {
    const fetchSecureData = async () => {
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get(`${API_BASE}/secure`, {
          headers: { Authorization: `Bearer ${token}` },
        });
        setSecureData(res.data);
      } catch (err) {
        console.error(err);
        setSecureData("Error fetching secure data.");
      }
    };

    fetchSecureData();
  }, []);

  return (
    <div style={styles.container}>
      <h2>Welcome to Dashboard</h2>
      <p>{secureData}</p>
      <button style={styles.button} onClick={onLogout}>
        Logout
      </button>
    </div>
  );
}

const styles = {
  container: {
    textAlign: "center",
    marginTop: "50px",
  },
  button: {
    marginTop: "20px",
    padding: "10px",
    background: "red",
    color: "#fff",
    border: "none",
    cursor: "pointer",
  },
};

export default Dashboard;


---

4️⃣ Folder structure

plansential-frontend/
│
├── src/
│   ├── App.js
│   ├── index.js
│   ├── LoginRegister.js
│   ├── Dashboard.js
│   └── ... (other files)
│
├── package.json
└── ...


---

5️⃣ Run

npm install axios
npm start


---

👉 Do you want me to upgrade it with Material-UI (beautiful forms and dashboard)? Or should I add your Plan Info / Plan Alerts tracker as a dashboard feature using your backend database?

