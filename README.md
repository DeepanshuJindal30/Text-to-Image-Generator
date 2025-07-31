using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Identity;
using PlanSential.Api.Models;
using PlanSential.Api.Services;
using System.Linq;
using System.Threading.Tasks;

namespace PlanSential.Api.Controllers
{
    [ApiController]
    [Route("api/controller")]
    public class AuthIdentityController : ControllerBase
    {
        private readonly UserManager<ApplicationUser> _userManager;
        private readonly JwtService _jwtService;

        public AuthIdentityController(UserManager<ApplicationUser> userManager, JwtService jwtService)
        {
            _userManager = userManager;
            _jwtService = jwtService;
        }

        public class RegisterRequest
        {
            public string Username { get; set; }
            public string Email { get; set; }
            public string Password { get; set; }
        }

        public class LoginRequest
        {
            public string Username { get; set; }
            public string Password { get; set; }
        }

        [HttpPost("register")]
        public async Task<IActionResult> Register([FromBody] RegisterRequest model)
        {
            if (string.IsNullOrWhiteSpace(model.Email))
                return BadRequest("Email is required");

            var user = new ApplicationUser
            {
                UserName = model.Username,
                Email = model.Email,
                EmailConfirmed = true
            };

            var result = await _userManager.CreateAsync(user, model.Password);

            if (!result.Succeeded)
                return BadRequest(result.Errors);

            return Ok(new { message = "User registered successfully!" });
        }

        [HttpPost("login")]
        public async Task<IActionResult> Login([FromBody] LoginRequest model)
        {
            var user = await _userManager.FindByNameAsync(model.Username);

            if (user == null || !await _userManager.CheckPasswordAsync(user, model.Password))
                return BadRequest("Invalid username or password");

            var token = _jwtService.GenerateToken(user);
            return Ok(new { token });
        }

        [HttpGet("secure")]
        [Microsoft.AspNetCore.Authorization.Authorize]
        public IActionResult SecureEndpoint()
        {
            return Ok("You have accessed a protected route!");
        }
    }
}





import axios from "axios";

const api = axios.create({
  baseURL: "http://localhost:5172/api",
});

export default api;
import React, { useState } from "react";
import {
  Container,
  TextField,
  Button,
  Typography,
  Box,
  Paper,
} from "@mui/material";
import api from "./api";

function App() {
  const [isRegister, setIsRegister] = useState(false);
  const [username, setUsername] = useState("");
  const [email, setEmail] = useState(""); // Only for register
  const [password, setPassword] = useState("");
  const [message, setMessage] = useState("");

  const handleAuth = async () => {
    try {
      if (isRegister) {
        await api.post("/controller/register", {
          username,
          email,
          password,
        });
        setMessage("User registered successfully! Please login now.");
        setIsRegister(false);
      } else {
        const res = await api.post("/controller/login", {
          username,
          password,
        });
        localStorage.setItem("token", res.data.token);
        setMessage("Login successful!");
      }
    } catch (err) {
      setMessage(err.response?.data || "Error occurred");
    }
  };

  return (
    <Container maxWidth="sm" sx={{ mt: 8 }}>
      <Paper elevation={3} sx={{ p: 4 }}>
        <Typography variant="h5" mb={2} align="center">
          {isRegister ? "Register" : "Login"}
        </Typography>

        <TextField
          label="Username"
          fullWidth
          margin="normal"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
        />
        {isRegister && (
          <TextField
            label="Email"
            fullWidth
            margin="normal"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
          />
        )}
        <TextField
          label="Password"
          type="password"
          fullWidth
          margin="normal"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />

        <Button
          fullWidth
          variant="contained"
          color="primary"
          sx={{ mt: 2 }}
          onClick={handleAuth}
        >
          {isRegister ? "Register" : "Login"}
        </Button>

        <Button
          fullWidth
          variant="text"
          sx={{ mt: 1 }}
          onClick={() => setIsRegister(!isRegister)}
        >
          {isRegister
            ? "Already have an account? Login"
            : "Don't have an account? Register"}
        </Button>

        {message && (
          <Box mt={2}>
            <Typography color="error" align="center">
              {message}
            </Typography>
          </Box>
        )}
      </Paper>
    </Container>
  );
}

export default App;





...

Great! Let’s add a Dashboard (token-protected) and display PLAN_INFO with alerts and notes using Material UI.


---

1️⃣ Frontend Structure

frontend/
├── src/
│   ├── api.js
│   ├── App.js
│   ├── Dashboard.js      # New
│   ├── LoginRegister.js  # Moved previous auth UI here
│   ├── index.js


---

2️⃣ Frontend Code

src/LoginRegister.js

import React, { useState } from "react";
import {
  Container,
  TextField,
  Button,
  Typography,
  Box,
  Paper,
} from "@mui/material";
import api from "./api";

function LoginRegister({ onLoginSuccess }) {
  const [isRegister, setIsRegister] = useState(false);
  const [username, setUsername] = useState("");
  const [email, setEmail] = useState(""); // Only for register
  const [password, setPassword] = useState("");
  const [message, setMessage] = useState("");

  const handleAuth = async () => {
    try {
      if (isRegister) {
        await api.post("/controller/register", {
          username,
          email,
          password,
        });
        setMessage("User registered successfully! Please login now.");
        setIsRegister(false);
      } else {
        const res = await api.post("/controller/login", {
          username,
          password,
        });
        localStorage.setItem("token", res.data.token);
        setMessage("Login successful!");
        onLoginSuccess();
      }
    } catch (err) {
      setMessage(err.response?.data || "Error occurred");
    }
  };

  return (
    <Container maxWidth="sm" sx={{ mt: 8 }}>
      <Paper elevation={3} sx={{ p: 4 }}>
        <Typography variant="h5" mb={2} align="center">
          {isRegister ? "Register" : "Login"}
        </Typography>

        <TextField
          label="Username"
          fullWidth
          margin="normal"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
        />
        {isRegister && (
          <TextField
            label="Email"
            fullWidth
            margin="normal"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
          />
        )}
        <TextField
          label="Password"
          type="password"
          fullWidth
          margin="normal"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />

        <Button
          fullWidth
          variant="contained"
          color="primary"
          sx={{ mt: 2 }}
          onClick={handleAuth}
        >
          {isRegister ? "Register" : "Login"}
        </Button>

        <Button
          fullWidth
          variant="text"
          sx={{ mt: 1 }}
          onClick={() => setIsRegister(!isRegister)}
        >
          {isRegister
            ? "Already have an account? Login"
            : "Don't have an account? Register"}
        </Button>

        {message && (
          <Box mt={2}>
            <Typography color="error" align="center">
              {message}
            </Typography>
          </Box>
        )}
      </Paper>
    </Container>
  );
}

export default LoginRegister;


---

src/Dashboard.js

import React, { useEffect, useState } from "react";
import {
  Container,
  Typography,
  Paper,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Button,
} from "@mui/material";
import api from "./api";

function Dashboard({ onLogout }) {
  const [plans, setPlans] = useState([]);

  useEffect(() => {
    const fetchPlans = async () => {
      try {
        const token = localStorage.getItem("token");
        const res = await api.get("/controller/secure", {
          headers: {
            Authorization: `Bearer ${token}`,
          },
        });

        // Dummy data (replace with backend result)
        setPlans([
          {
            MARKET_SEGMENT_ID: "A",
            PLAN_NUM: "123456",
            PLAN_NAME1: "Retirement Plan",
            PLAN_STATUS: "Active",
            VALUE_OF_PLAN_ASSETS: "$1,000,000",
          },
        ]);
      } catch (err) {
        console.error(err);
      }
    };

    fetchPlans();
  }, []);

  return (
    <Container maxWidth="md" sx={{ mt: 8 }}>
      <Typography variant="h4" align="center" gutterBottom>
        Dashboard
      </Typography>

      <Button
        variant="outlined"
        color="error"
        sx={{ mb: 3 }}
        onClick={onLogout}
      >
        Logout
      </Button>

      <TableContainer component={Paper}>
        <Table>
          <TableHead>
            <TableRow>
              <TableCell>Market Segment</TableCell>
              <TableCell>Plan Number</TableCell>
              <TableCell>Plan Name</TableCell>
              <TableCell>Status</TableCell>
              <TableCell>Assets</TableCell>
            </TableRow>
          </TableHead>
          <TableBody>
            {plans.map((plan, idx) => (
              <TableRow key={idx}>
                <TableCell>{plan.MARKET_SEGMENT_ID}</TableCell>
                <TableCell>{plan.PLAN_NUM}</TableCell>
                <TableCell>{plan.PLAN_NAME1}</TableCell>
                <TableCell>{plan.PLAN_STATUS}</TableCell>
                <TableCell>{plan.VALUE_OF_PLAN_ASSETS}</TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>
    </Container>
  );
}

export default Dashboard;


---

src/App.js (Updated)

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

3️⃣ Backend Update (Dashboard Data)

In AuthIdentityController.cs, update SecureEndpoint:

[HttpGet("secure")]
[Authorize]
public IActionResult SecureEndpoint()
{
    // TODO: Fetch PLAN_INFO with joins (now just dummy)
    var plans = new[]
    {
        new {
            MARKET_SEGMENT_ID = "A",
            PLAN_NUM = "123456",
            PLAN_NAME1 = "Retirement Plan",
            PLAN_STATUS = "Active",
            VALUE_OF_PLAN_ASSETS = "$1,000,000"
        }
    };

    return Ok(plans);
}


---

4️⃣ Run

Backend: dotnet run

Frontend: npm start



---

👉 Do you want me to connect real PLAN_INFO, PlanAlerts, AlertStatuses tables in SecureEndpoint and display real DB data in Dashboard? Or should I add search/filter (like a tracker) for plans and alerts?

