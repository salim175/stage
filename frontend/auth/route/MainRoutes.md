# The MainRoute:
```ts
const MainRoutes = {
    path: '/main',
    redirect: '/main',
    component: () => import('@/layouts/dashboard/DashboardLayout.vue'),
    children: [
        {
            name: 'LandingPage',
            path: '/',
            component: () => import('../views/dashboard/DefaultDashboard.vue'),
            meta: { requiresAuth: true }
        },
        {
            name: 'Dashboard',
            path: '/dashboard',
            component: () => import('../views/dashboard/DefaultDashboard.vue'),
            meta: { requiresAuth: true }
        },
        {
            path: '/profile',
            name: 'Profile',
            component: () => import('../views/pages/Profile_Page.vue'),
            meta: { layout: 'DashboardLayout', requiresAuth: true }
        },
        {
            name: 'Details',
            path: '/details/:id',
            component: () => import('../views/pages/MailPage.vue'),
            meta: { requiresAuth: true },
            props: true,
        },
        {
            name: 'IPDB',
            path: "/ip-report/",
            component: () => import('../views/pages/Ipdb_Page.vue'),
            meta: { requiresAuth: true },
            props: true,
        },
        {
            name: 'activty of Ip',
            path: "/ip-activity",
            component: () => import('../views/pages/ActivityOf_IpPage.vue'),
            meta: { requiresAuth: true },
            props: true,
        },
        {
            name: 'upload mail',
            path: "/upload-mails",
            component: () => import('@/views/pages/MailUploader.vue'),
            meta: { requiresAuth: true }
        },
        {
            name: 'AllMails',
            path: "/all-mails",
            component: () => import('@/views/pages/AllMailsPage.vue'),
            meta: { requiresAuth: true }
        },
    ]
};

export default MainRoutes;
```