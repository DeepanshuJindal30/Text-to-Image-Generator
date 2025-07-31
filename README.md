import React, { useState } from "react";
import axios from "axios";
import {
  Container,
  Box,
  TextField,
  Button,
  Typography,
  Paper,
} from "@mui/material";

const API_BASE = "http://localhost:5172/api/controller"; // Backend URL

function LoginRegister({ onLoginSuccess }) {
  const [isRegister, setIsRegister] = useState(false);
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");

  // Handle register
  const handleRegister = async () => {
    try {
      setError("");
      await axios.post(`${API_BASE}/register`, { username, password });
      alert("Registration successful! You can now log in.");
      setIsRegister(false);
    } catch (err) {
      if (err.response && err.response.data) {
        setError(err.response.data.message || "Something went wrong");
      } else {
        setError("Something went wrong. Please try again.");
      }
    }
  };

  // Handle login
  const handleLogin = async () => {
    try {
      setError("");
      const response = await axios.post(`${API_BASE}/login`, {
        username,
        password,
      });

      // Save token to localStorage
      localStorage.setItem("token", response.data.token);

      // Notify parent
      onLoginSuccess();
    } catch (err) {
      if (err.response && err.response.data) {
        setError(err.response.data.message || "Something went wrong");
      } else {
        setError("Something went wrong. Please try again.");
      }
    }
  };

  return (
    <Container maxWidth="xs">
      <Paper elevation={3} sx={{ mt: 10, p: 4 }}>
        <Typography variant="h5" align="center" gutterBottom>
          {isRegister ? "Register" : "Login"}
        </Typography>

        <TextField
          label="Username"
          variant="outlined"
          fullWidth
          sx={{ mb: 2 }}
          value={username}
          onChange={(e) => setUsername(e.target.value)}
        />

        <TextField
          label="Password"
          type="password"
          variant="outlined"
          fullWidth
          sx={{ mb: 2 }}
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />

        {error && (
          <Typography color="error" align="center" sx={{ mb: 2 }}>
            {error}
          </Typography>
        )}

        <Button
          fullWidth
          variant="contained"
          color="primary"
          onClick={isRegister ? handleRegister : handleLogin}
        >
          {isRegister ? "Register" : "Login"}
        </Button>

        <Typography
          align="center"
          sx={{ mt: 2, cursor: "pointer", color: "primary.main" }}
          onClick={() => setIsRegister((prev) => !prev)}
        >
          {isRegister
            ? "Already have an account? Login"
            : "Don't have an account? Register"}
        </Typography>
      </Paper>
    </Container>
  );
}

export default LoginRegister;