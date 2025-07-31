
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace PlanSential.Api.Controllers
{
    [Route("api/secure")]
    [ApiController]
    public class SecureController : ControllerBase
    {
        // 🔒 Secure endpoint (requires JWT Identity authentication)
        [HttpGet("data")]
        [Authorize]
        public IActionResult GetSecureData()
        {
            return Ok("This is a secure endpoint using Identity + JWT!");
        }

        // If you want a test endpoint without authentication:
        [HttpGet("public")]
        [AllowAnonymous]
        public IActionResult PublicData()
        {
            return Ok("This is a public endpoint.");
        }
    }
}






DECLARE @i INT = 1;

WHILE @i <= 1000
BEGIN
    INSERT INTO AspNetUsers
    (Id, UserName, NormalizedUserName, Email, NormalizedEmail, EmailConfirmed, PasswordHash, SecurityStamp, ConcurrencyStamp, PhoneNumberConfirmed, TwoFactorEnabled, LockoutEnabled, AccessFailedCount)
    VALUES
    (NEWID(), 
     CONCAT('user', @i, '@test.com'), 
     UPPER(CONCAT('user', @i, '@test.com')), 
     CONCAT('user', @i, '@test.com'), 
     UPPER(CONCAT('user', @i, '@test.com')), 
     1,
     -- Default hashed password for "Password@123" (you can generate it with Identity hasher)
     'AQAAAAEAACcQAAAAEOX5xMzsa0mPwS8v53RbsH+iG0VgGFO6pK/zgT0p8bAtOlq19V2cnr7O8vAeqFLdOQ==',
     NEWID(), 
     NEWID(), 
     0, 0, 0, 0);
    
    SET @i += 1;
END



Sure! Let's create a React + Material-UI frontend for your AuthIdentity (Register + Login) system.


---

1. Project Setup

If you don't already have the React project:

npx create-react-app plansential-frontend
cd plansential-frontend
npm install @mui/material @emotion/react @emotion/styled axios react-router-dom


---

2. Folder Structure

frontend/
 └── src/
      ├── api/
      │    └── authService.js
      ├── components/
      │    ├── Login.js
      │    ├── Register.js
      │    └── Dashboard.js
      ├── App.js
      └── index.js


---

3. API Service (src/api/authService.js)

import axios from "axios";

const API_URL = "http://localhost:5172/api/authidentity"; // Adjust to your backend URL

export const registerUser = async (username, password) => {
  return axios.post(`${API_URL}/register`, null, {
    params: { username, password }
  });
};

export const loginUser = async (username, password) => {
  return axios.post(`${API_URL}/login`, null, {
    params: { username, password }
  });
};


---

4. Login Component (src/components/Login.js)

import React, { useState } from "react";
import { loginUser } from "../api/authService";
import { TextField, Button, Container, Typography, Box } from "@mui/material";

function Login({ setToken }) {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");

  const handleLogin = async () => {
    try {
      const res = await loginUser(username, password);
      localStorage.setItem("token", res.data.token);
      setToken(res.data.token);
      alert("Login successful!");
    } catch (error) {
      alert("Invalid credentials!");
    }
  };

  return (
    <Container maxWidth="sm">
      <Box sx={{ mt: 5 }}>
        <Typography variant="h4" gutterBottom>Login</Typography>
        <TextField 
          label="Username" fullWidth margin="normal"
          value={username} onChange={e => setUsername(e.target.value)} />
        <TextField 
          label="Password" type="password" fullWidth margin="normal"
          value={password} onChange={e => setPassword(e.target.value)} />
        <Button variant="contained" fullWidth sx={{ mt: 2 }} onClick={handleLogin}>
          Login
        </Button>
      </Box>
    </Container>
  );
}

export default Login;


---

5. Register Component (src/components/Register.js)

import React, { useState } from "react";
import { registerUser } from "../api/authService";
import { TextField, Button, Container, Typography, Box } from "@mui/material";

function Register() {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");

  const handleRegister = async () => {
    try {
      await registerUser(username, password);
      alert("User registered successfully!");
    } catch (error) {
      alert("Error registering user!");
    }
  };

  return (
    <Container maxWidth="sm">
      <Box sx={{ mt: 5 }}>
        <Typography variant="h4" gutterBottom>Register</Typography>
        <TextField 
          label="Username" fullWidth margin="normal"
          value={username} onChange={e => setUsername(e.target.value)} />
        <TextField 
          label="Password" type="password" fullWidth margin="normal"
          value={password} onChange={e => setPassword(e.target.value)} />
        <Button variant="contained" fullWidth sx={{ mt: 2 }} onClick={handleRegister}>
          Register
        </Button>
      </Box>
    </Container>
  );
}

export default Register;


---

6. Dashboard Component (src/components/Dashboard.js)

import React, { useEffect, useState } from "react";
import axios from "axios";
import { Container, Typography } from "@mui/material";

function Dashboard() {
  const [data, setData] = useState("");

  useEffect(() => {
    const token = localStorage.getItem("token");
    axios
      .get("http://localhost:5172/api/secure/data", {
        headers: { Authorization: `Bearer ${token}` }
      })
      .then((res) => setData(res.data))
      .catch(() => setData("Unauthorized"));
  }, []);

  return (
    <Container maxWidth="sm" sx={{ mt: 5 }}>
      <Typography variant="h5">Secure Data:</Typography>
      <Typography variant="body1" sx={{ mt: 2 }}>{data}</Typography>
    </Container>
  );
}

export default Dashboard;


---

7. App.js

import React, { useState } from "react";
import { BrowserRouter as Router, Route, Routes, Link } from "react-router-dom";
import Login from "./components/Login";
import Register from "./components/Register";
import Dashboard from "./components/Dashboard";
import { AppBar, Toolbar, Button } from "@mui/material";

function App() {
  const [token, setToken] = useState(localStorage.getItem("token"));

  return (
    <Router>
      <AppBar position="static">
        <Toolbar>
          <Button color="inherit" component={Link} to="/">Login</Button>
          <Button color="inherit" component={Link} to="/register">Register</Button>
          {token && <Button color="inherit" component={Link} to="/dashboard">Dashboard</Button>}
        </Toolbar>
      </AppBar>
      <Routes>
        <Route path="/" element={<Login setToken={setToken} />} />
        <Route path="/register" element={<Register />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </Router>
  );
}

export default App;


---

8. Run the frontend

npm start


---

Features

1. Register: Calls /api/authidentity/register


2. Login: Calls /api/authidentity/login and stores JWT in localStorage


3. Dashboard: Calls api/secure/data with JWT (protected)


4. Material UI: Clean and responsive




---

👉 Do you want me to add pagination + display all users (1000 rows) in a Material UI table? Or should I first integrate Logout & token expiry handling in this frontend?


# IMAGINATE HUB

Welcome to Imaginate Hub, a state-of-the-art image generation platform where imagination meets machine learning. Utilizing advanced models, this app offers a seamless experience for users to input prompts and visualize the resulting images.

🚀 **Explore Live Application**: Dive into the Imaginate Hub experience right now! Check out the [Live Model Here](https://image-generator-app.streamlit.app/).

![Imaginate Hub Screenshot](./imaginatehub_v2.png) 

## Features

- **User-friendly Interface**: An intuitive, clean UI allowing you to focus on creativity.
  
- **Model Variety**: Choose from several image generation models: "Stable Diffusion", "Anything-v4.0", "Waifu Diffusion", and "Vintedios Diffusion".
  
- **Rapid Image Generation**: Simply type in your desired prompt and watch the magic happen.

## Detailed Steps for Usage

1. **Launch the App**: Navigate to the application URL or start the Streamlit app.

2. **Sample Preview**: For insights on what to expect, click on the **"Prompt & Result Sample"** section to view a sample prompt and the resultant image.

3. **Model Selection**:
    - Navigate to the dropdown menu.
    - Select your desired image generation model.

4. **API Key Configuration**:
    - Locate the sidebar on the left.
    - Input your Replicate API key into the provided field. If you don't have one, you can obtain it by [following this guide](https://gist.github.com/MonishSoundarRaj/76d1d6ef9a806d879ef4357ae5111f00).

5. **Input Your Prompt**:
    - In the main section, locate the text area labeled "Enter Image Generation Prompt".
    - Type in your creative prompt.

6. **Generate Image**:
    - Click the "Submit Prompt" button.
    - Give the system a moment to process.
    - Once completed, the generated image will display on screen.

7. **Download the Result**:
    - Below the generated image, there will be an option to download it for personal use.

## Requirements

- Streamlit
- Replicate
- os module
- requests

## Installation

To set up and run Imaginate Hub on your own system:

1. Clone this repository:
```
git clone https://github.com/MonishSoundarRaj/image-generator-streamlit
```

2. Navigate to the project directory:
```
cd iamge-generator-streamlit
```

3. Install the required Python packages:
```
pip install -r requirements.txt
```

4. Run the Streamlit app:
```
streamlit run app.py
```

## License

This project falls under the MIT License. For more details, please check the [LICENSE](LICENSE) file.

