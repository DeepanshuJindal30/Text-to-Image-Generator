[HttpPost("login")]
public async Task<IActionResult> Login([FromBody] LoginRequest request)
{
    try
    {
        var user = await _userManager.FindByNameAsync(request.Username);
        if (user == null)
            return BadRequest(new { message = "User not found" });

        // Check password
        var isPasswordValid = await _userManager.CheckPasswordAsync(user, request.Password);
        if (!isPasswordValid)
            return BadRequest(new { message = "Invalid password" });

        // Generate token
        var token = _jwtService.GenerateToken(user);

        return Ok(new { token });
    }
    catch (Exception ex)
    {
        return StatusCode(500, new { message = "Internal Server Error", error = ex.Message });
    }
}







try {
  const response = await axios.post(`${API_BASE}/login`, {
    username,
    password,
  });

  localStorage.setItem("token", response.data.token);
  onLoginSuccess();
} catch (err) {
  if (err.response && err.response.data) {
    setError(err.response.data.message || "Something went wrong");
  } else {
    setError("Something went wrong");
  }
}