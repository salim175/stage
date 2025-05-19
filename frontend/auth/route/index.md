# 🧭 Route Configuration
```ts
createRouter({
  history: createWebHashHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/auth/login',
      name: 'Login',
      component: () => import('@/views/pages/LoginPage.vue'),
      meta: { requiresAuth: false }
    },
    MainRoutes
  ]
})
```
- Uses createWebHashHistory to support hash-based routing
- Includes a dedicated Login route (/auth/login)
- Imports additional nested/protected routes from MainRoutes

# 🔐 Global Route Guard: beforeEach
```ts
router.beforeEach(async (to, from, next) => {
  const uiStore = useUIStore()
  uiStore.isLoading = true

  const userStore = useUserStore()
  const isAuth = await userStore.isLoggedIn()

  if (to.meta.requiresAuth && !isAuth) {
    next('/auth/login') // 🔐 Block access to protected route
  } else if (to.path === '/auth/login' && isAuth) {
    next('/') // 🔁 Already logged in → redirect home
  } else {
    next() // ✅ Proceed as normal
  }
})
```
### 🔍 Why this matters:
- Protects routes by checking to.meta.requiresAuth
- Prevents authenticated users from accessing /auth/login
- Validates token via userStore.isLoggedIn() before proceeding