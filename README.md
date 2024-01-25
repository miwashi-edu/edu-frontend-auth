# edu-frontend-auth

# Base Code

See: [edu-auth-token-based](https://github.com/miwashi-edu/edu-auth-token-based)

# 1. Session Storage

> Frontend: Use sessionStorage.setItem('access_token', result.token) and sessionStorage.removeItem('access_token') for setting and clearing the JWT.
Backend: No change needed, as the token is managed on the frontend.
Local Storage (Current Implementation):

> Replace localStorage with sessionStorage for storing and retrieving the JWT.

```js
// In handleLogin
sessionStorage.setItem('access_token', result.token);

// In handleLogout
sessionStorage.removeItem('access_token');

// In fetchData
const token = sessionStorage.getItem('access_token');
```
## Vulnerabilities: 

Vulnerable to Cross-Site Scripting (XSS) attacks. If an attacker can inject malicious scripts into your web application, they can access session storage and steal the JWT.

### Cross-tab/session limitations: 

Data stored in session storage is accessible only in the tab where it was created and is cleared when the tab is closed, limiting its use in multi-tab applications.

# 2. Local Storage (as is)

> Frontend: Use localStorage.setItem('access_token', result.token) and localStorage.removeItem('access_token').
Backend: No change needed.


```js
// In handleLogin
localStorage.setItem('access_token', result.token);

// In handleLogout
localStorage.removeItem('access_token');

// In fetchData
const token = localStorage.getItem('access_token');
```

## Vulnerabilities:

Similar to session storage, local storage is susceptible to XSS attacks. Malicious scripts can access local storage and extract the JWT.

## Persistence beyond session: 

Unlike session storage, local storage persists across browser sessions until explicitly cleared, which could be a risk if the user forgets to log out or the token is not managed properly.

# 3. Cookie

> Frontend: Instead of storing the token in local storage, send credentials using credentials: 'include' in the fetch API and expect the server to set a cookie.
Backend: On successful authentication, set the JWT in a cookie (res.cookie('token', token)) and ensure CORS is configured to allow credentials.


```bash
// In handleLogin and handleRegister
fetch('http://localhost:3001/auth/login', {
  method: 'POST',
  credentials: 'include', // Include credentials to handle cookies
  // ... other settings
});

// In handleLogout, instruct the backend to clear the cookie
fetch('http://localhost:3001/auth/logout', {
  method: 'GET',
  credentials: 'include',
});

// In fetchData
fetch('http://localhost:3002/data/users', {
  credentials: 'include', // Include credentials to send cookie
  // ... other settings
});
```

## Vulnerabilities

Susceptible to Cross-Site Request Forgery (CSRF) attacks unless additional protections (like CSRF tokens) are implemented.  

If not configured with the HttpOnly flag, cookies are also vulnerable to XSS attacks.  

### Cross-origin restrictions: 

Cookies have limitations when dealing with cross-origin requests and require proper CORS configuration.

# 4. HTTP-only Cookie
Similar to the regular cookie method, but ensure the backend sets the HTTP-only flag.

The frontend code remains the same as in the cookie example. Ensure your backend sets the httpOnly flag on the cookie.

## Vulnerabilities
Reduces the risk of XSS attacks since HTTP-only cookies are not accessible via JavaScript.  
Still vulnerable to CSRF attacks unless additional protections are implemented.

### Same cross-origin restrictions as regular cookies.
  
# 5. Bearer Token in Authorization Header

> Same as the regular cookie method, but set the cookie with the httpOnly flag (res.cookie('token', token, { httpOnly: true })).
This method enhances security by preventing client-side scripts from accessing the token.

```js
// In handleLogin
setAuthenticated(true);
this.setState({ token: result.token }); // Store token in state

// In fetchData
fetch('http://localhost:3002/data/users', {
  headers: {
    Authorization: `Bearer ${this.state.token}`
  }
});
```

### Vulnerabilities

If the token is stored in memory (e.g., React state), it reduces the risk of XSS attacks compared to local or session storage. However, it's still susceptible if an XSS attack can manipulate application state.

Vulnerable to CSRF attacks if not properly handled.

### Phishing risks: 

Users might be tricked into obtaining the token through social engineering or phishing attacks.

# 6. IndexedDB

> Frontend: Use IndexedDB API to store the JWT. This is more complex but useful for large amounts of data and provides asynchronous access.
Backend: No change needed.

```js
// After successful login, store the token in IndexedDB
// In handleLogin
indexedDBUtility.storeToken(result.token);

// In fetchData
indexedDBUtility.getToken().then(token => {
  fetch('http://localhost:3002/data/users', {
    headers: {
      Authorization: `Bearer ${token}`
    }
  });
  // ... rest of your fetchData code
});
```

## Vulnerabilities
Similar to local and session storage, IndexedDB is susceptible to XSS attacks. If a malicious script accesses the IndexedDB, it can compromise the JWT.

### Complexity and performance considerations: 

IndexedDB is more complex to implement and manage, and performance might be an issue for large-scale applications.

> Remember, for all methods involving cookies (standard and HTTP-only), your backend must be set up to handle setting and clearing cookies. Similarly, using the Authorization header or IndexedDB will require corresponding backend support for token validation.
> 



