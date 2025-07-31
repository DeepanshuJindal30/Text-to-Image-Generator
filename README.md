const handleSubmit = async (e) => {
  e.preventDefault();
  setError("");

  try {
    if (isRegister) {
      // Register API call
      const response = await axios.post(`${API_BASE}/register`, {
        username,
        email: `${username}@test.com`, // Provide dummy email
        password
      });

      alert("Registration successful! Please login.");
      setIsRegister(false);
    } else {
      // Login API call
      const response = await axios.post(`${API_BASE}/login`, {
        username,
        password
      });

      if (response.data.token) {
        localStorage.setItem("token", response.data.token);
        onLoginSuccess();
      } else {
        setError("Invalid login response. Please try again.");
      }
    }
  } catch (err) {
    console.error("API error:", err);
    setError(err.response?.data || "Something went wrong. Please try again.");
  }
};