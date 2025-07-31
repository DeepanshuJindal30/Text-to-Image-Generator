const handleSubmit = async (e) => {
  e.preventDefault();
  setError("");

  try {
    if (isRegister) {
      // Register API with proper body (use JSON instead of query params)
      const res = await axios.post(`${API_BASE}/register`, {
        username,
        email: `${username}@test.com`,  // identity requires email, so dummy email
        password
      });

      if (res.status === 200) {
        alert("Registration successful! Please login.");
        setIsRegister(false);
      }
    } else {
      // Login API
      const res = await axios.post(`${API_BASE}/login`, {
        username,
        password
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