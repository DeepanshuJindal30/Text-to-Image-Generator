code-

import React from "react";
import { Routes, Route, Link } from "react-router-dom";
import Login from "./components/Login";
import AMLList from "./components/AMLList";
import TrusteeList from "./components/TrusteeList";
import SPList from "./components/SPList";
import Register from "./components/Register";

export default function App() {
  return (
    <div>
      <nav style={{ padding: "10px", background: "#f5f5f5" }}>
        <Link to="/aml" style={{ marginRight: "15px" }}>AML</Link>
        <Link to="/trustee" style={{ marginRight: "15px" }}>Trustee</Link>
        <Link to="/sp" style={{ marginRight: "15px" }}>Service Providers</Link>
        <Link to="/register" style={{ marginRight: "15px" }}>Register</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Login />} />
        <Route path="/aml" element={<AMLList />} />
        <Route path="/trustee" element={<TrusteeList />} />
        <Route path="/sp" element={<SPList />} />
        <Route path="/register" element={<Register />} />
      </Routes>
    </div>
  );
}




register--




import React, { useState } from "react";
import { Button, TextField, Container, Typography } from "@mui/material";
import api from "../api"; // your axios instance
import { useNavigate } from "react-router-dom";

export default function Register() {
  const [username, setUsername] = useState("");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [role, setRole] = useState("User"); // default role
  const navigate = useNavigate();

  const handleRegister = async () => {
    try {
      await api.post("/auth/register", { username, email, password, role });
      alert("User Registered Successfully");
      navigate("/"); // redirect to login after registration
    } catch (err) {
      alert("Registration Failed: " + (err.response?.data || err.message));
    }
  };

  return (
    <Container maxWidth="sm">
      <Typography variant="h4" gutterBottom>
        Register
      </Typography>
      <TextField
        label="Username"
        fullWidth
        margin="normal"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
      />
      <TextField
        label="Email"
        fullWidth
        margin="normal"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <TextField
        label="Password"
        type="password"
        fullWidth
        margin="normal"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <TextField
        label="Role (Admin/User)"
        fullWidth
        margin="normal"
        value={role}
        onChange={(e) => setRole(e.target.value)}
      />
      <Button variant="contained" fullWidth onClick={handleRegister}>
        Register
      </Button>
    </Container>
  );
}


login- 


import React, { useState } from "react";
import { Button, TextField, Container, Typography } from "@mui/material";
import api from "../api";
import { useNavigate } from "react-router-dom";
import { useDispatch } from "react-redux";
import { setAuth } from "../redux/slices/authSlice";

export default function Login() {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const navigate = useNavigate();
  const dispatch = useDispatch();

  const handleLogin = async () => {
    try {
      const res = await api.post("/auth/login", { username, password });

      // Save token & roles
      localStorage.setItem("token", res.data.token);
      localStorage.setItem("roles", JSON.stringify(res.data.roles));

      // Set token for API
      api.defaults.headers.common["Authorization"] = `Bearer ${res.data.token}`;
      dispatch(setAuth({ token: res.data.token, roles: res.data.roles }));

      alert("Login Successful");

      // Redirect based on role
      if (res.data.roles.includes("Admin")) {
        navigate("/aml");
      } else if (res.data.roles.includes("Manager")) {
        navigate("/trustee");
      } else {
        navigate("/sp");
      }
    } catch (err) {
      alert("Login Failed: " + (err.response?.data || err.message));
    }
  };

  return (
    <Container maxWidth="sm">
      <Typography variant="h4" gutterBottom>
        Login
      </Typography>
      <TextField
        label="Username"
        fullWidth
        margin="normal"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
      />
      <TextField
        label="Password"
        type="password"
        fullWidth
        margin="normal"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <Button variant="contained" fullWidth onClick={handleLogin}>
        Login
      </Button>
    </Container>
  );
}