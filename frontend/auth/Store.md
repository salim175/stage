# 🧠 userStore – Authentication Store
This store handles authentication state, including login, logout, and user profile management using Pinia and Axios.

### 🧩 State
| **Property**	| **Type**	| **Description**| 
|--------------------|----------------|-----------------------------|
| `user` |  `User or null` |Holds the authenticated user's data if logged in

---
### 🔐 User Interface
```ts
export interface User {
  id?: string;
  email: string;
  password: string;
  firstname?: string;
  lastname?: string;
  pseudo?: string;
  role?: string;
  last_logins?: Array<any>;
  oldPassword?: string;
  blocked?: boolean;
  rememberMe?: boolean;
}
```

---

### 🧪 Actions
``isLoggedIn(): Promise<boolean>``

Checks if a valid token exists in ``localStorage`` and fetches the authenticated user's profile.
- ✅ Sets userStore.user if authenticated
- ❌ Sets null if token is missing/invalid

``logIn(userData: User, rememberMe: boolean): Promise<{ status: boolean, apiStatus?: any }>``   
Authenticates the user via API:
- Sends ``POST /auth/login``
- On success:
    - Stores the token in ``localStorage``
    - Sets ``axios.defaults.headers.common['Authorization']``
    - Fetches user profile (``/auth/profile``)
    - Sets ``userStore.user``

- Returns:
```json
{ status: true } // on success
{ status: false, apiStatus: 401 | 403 | 'network_error' } // on failure
```
``logOut(): Promise<void>``

Logs the user out:  
- Clears ``localStorage``
- Resets ``userStore.user`` to ``null``

---

### ⚙️ Usage Example
```ts
const userStore = useUserStore()

// Login
await userStore.logIn({ email: 'user@mail.com', password: 'pass123' }, true)

// Check if still authenticated
const isAuth = await userStore.isLoggedIn()

// Access user profile
console.log(userStore.user?.firstname)

// Logout
await userStore.logOut()
```

---

### 🛡️ Auth Token Handling
On login, the token is stored and injected globally into Axios:

```ts
axios.defaults.headers.common['Authorization'] = `Bearer ${token}`
Also ensure this is re-injected in main.ts:
```
**📌 WHY?**    
Axios does not automatically reuse the token stored in localStorage. Without manually attaching it to every request, your backend will reject protected API calls (with 401 Unauthorized), especially right after login.

Also ensure this is re-injected in ``main.ts``:

```ts
const token = localStorage.getItem('Auth')
if (token) {
  axios.defaults.headers.common['Authorization'] = `Bearer ${token}`
}
```
**📌 WHY?**  
When the user refreshes the page:
- Pinia store resets to default
- Token is still in ``localStorage``, ✅
- But Axios header is reset ❌

This line ensures API requests stay authenticated even after reload — until logout or token expiry.

# The Code complete:
```ts
import axios from "axios";
import { defineStore } from "pinia";

export interface User {
    id?: string;
    email: string;
    password: string;
    firstname?: string;
    lastname?: string;
    pseudo?: string;
    role?: string;
    last_logins?: Array<any>;
    oldPassword?: string;
    blocked?: boolean;
    rememberMe?: boolean;
}

export const useUserStore = defineStore('user', {
    state: () => ({
        user: null as User | null,
    }),
    getters: {

    },
    actions: {
        async isLoggedIn() {
            let token = localStorage.getItem("Auth");

            try {
                if (token != null) {
                    const response = await axios.get<any>(import.meta.env.VITE_API_URL + '/auth/profile', { headers: { Authorization: `Bearer ${token}` } });
                    console.log('IsLoged IN: ', response.data)
                    if (response.status == 200) {
                        this.user = response.data;
                        return true
                    }
                } else {
                    this.user = null;
                    return false;
                }
            } catch (error) {
                this.user = null;
                return false;
            }

        },
        async logIn(userData: User, rememberMe: Boolean) {
            try {
                if (rememberMe == true) {
                    userData.rememberMe = true;
                } else {
                    userData.rememberMe = false;
                }
                const response = await axios.post<any>(import.meta.env.VITE_API_URL + '/auth/login', userData, { headers: { 'User-Agent': 'Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.111 Safari/537.36' } });
                console.log('TOKEN:', response)
                if (response.status == 200) {
                    localStorage.setItem("Auth", response.data);
                    axios.defaults.headers.common['Authorization'] = `Bearer ${response.data}`
                    let token = response.data;
                    if (token) {
                        const res = await axios.get<any>(import.meta.env.VITE_API_URL + '/auth/profile', { headers: { Authorization: `Bearer ${token}` } });
                        if (res.status == 200) {
                            this.user = response.data;
                            return { status: true }
                        }
                    } else {
                        this.user = null;
                        return { status: false };
                    }

                }

            } catch (error: unknown) {
                if (axios.isAxiosError(error) && error.response) {
                    return { status: false, apiStatus: error.response.status }
                } else {
                    return { status: false, apiStatus: 'network_error' }
                }
            }



        },
        async logOut() {
            await localStorage.clear();
            this.user = null;
        },
    },
});
```