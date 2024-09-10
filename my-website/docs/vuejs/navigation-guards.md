# Exemples d'utilisations de navaigation guard en vue.js


Les **Navigation Guards** dans Vue.js permettent de contrôler l'accès aux routes dans une application Vue Router. Il existe plusieurs types de gardiens de navigation :

- Global Guards : qui s'appliquent à toutes les routes.
- Per-route Guards : définis directement sur des routes spécifiques.
- In-component Guards : définis dans les composants eux-mêmes.
Voici un exemple complet avec les différents types de gardes de navigation.

**Exemple de Global Guard (Garde global)**

Dans cet exemple, on va utiliser un guard global pour vérifier si l'utilisateur est authentifié avant de lui permettre de naviguer vers certaines routes.

```javascript
//CAS 01 : SIMULATION DE L'AUTHENTIFICATION VIA LE STORE (tester avec l'application scholl-management)
import {createRouter, createWebHistory} from 'vue-router'

import LoginView from "@/views/LoginView.vue";
import {useAuthentStore} from "@/store/authenticationStore.js";


const router = createRouter({
    history: createWebHistory(import.meta.env.BASE_URL),
    routes: [
        {
            path: '/',
            name: 'login',
            component: LoginView
        },
        {
            path: '/home',
            name: 'home',
            meta: {requresAuth: true},
            // route level code-splitting
            // this generates a separate chunk (About.[hash].js) for this route
            // which is lazy-loaded when the route is visited.
            component: () => import('../views/HomeView.vue'),
        },
        {
            path: '/students',
            name: 'students',
            meta: {requresAuth: true},
            // route level code-splitting
            // this generates a separate chunk (About.[hash].js) for this route
            // which is lazy-loaded when the route is visited.
            component: () => import('../views/Students.vue')
        },
        {
            path: '/dashboard',
            name: 'dashboard',
            meta: {requresAuth: true},
            // route level code-splitting
            // this generates a separate chunk (About.[hash].js) for this route
            // which is lazy-loaded when the route is visited.
            component: () => import('../views/Dashboard.vue')
        },
        {
            path: '/payments',
            name: 'payments',
            meta: {requresAuth: true},
            // route level code-splitting
            // this generates a separate chunk (About.[hash].js) for this route
            // which is lazy-loaded when the route is visited.
            component: () => import('../views/Payments.vue')
        }
    ]
})

router.beforeEach(
    (to, from, next) => {
        //Très important de faire de declare le store cf comment doc off vue ci dessous :
        // ✅ This will work because the router starts its navigation after
        // the router is installed and pinia will be installed too
        const store = useAuthentStore()
        if(to.matched.some(record =>record.meta.requresAuth)){
            if(!store.isUserLoged){
                next({path: '/'});
            }else {
                next()
            }
        }else {
            next()
        }

    }
)

export default router
```

```javascript
//CAS 01 : SIMULATION DE L'AUTHENTIFICATION VIA LE localStorage
// src/router/index.js
import Vue from 'vue';
import VueRouter from 'vue-router';
import Home from '@/components/Home.vue';
import Dashboard from '@/components/Dashboard.vue';
import Login from '@/components/Login.vue';

Vue.use(VueRouter);

// Définition des routes
const routes = [
    { path: '/', component: Home },
    { path: '/login', component: Login },
    {
        path: '/dashboard',
        component: Dashboard,
        meta: { requiresAuth: true } // Indique que cette route nécessite une authentification
    },
];

const router = new VueRouter({
    mode: 'history',
    routes,
});

// Simulons une fonction d'authentification (elle devrait venir de votre store ou autre)
function isAuthenticated() {
    return !!localStorage.getItem('auth'); // Simule une authentification en vérifiant un élément dans le localStorage
}

// Global Guard: Vérification de l'authentification avant chaque navigation
router.beforeEach((to, from, next) => {
    if (to.matched.some(record => record.meta.requiresAuth)) {
        // Si la route nécessite une authentification
        if (!isAuthenticated()) {
            // Redirection vers la page de connexion si l'utilisateur n'est pas authentifié
            next({ path: '/login' });
        } else {
            // Si l'utilisateur est authentifié, il peut naviguer vers la route demandée
            next();
        }
    } else {
        // Si la route ne nécessite pas d'authentification
        next();
    }
});

export default router;

export default router
```

**Exemple de Per-route Guard (Garde défini par route)**

Les gardes par route permettent de définir des comportements spécifiques pour certaines routes.

```javascript
// src/router/index.js
const routes = [
  {
    path: '/admin',
    component: AdminComponent,
    beforeEnter: (to, from, next) => {
      // Vérification spécifique pour cette route
      if (isAuthenticated() && isUserAdmin()) {
        next(); // L'utilisateur est un admin, il peut accéder
      } else {
        next('/login'); // Sinon, redirection vers la page de login
      }
    }
  }
];
```
**Exemple de In-component Guard (Garde dans un composant)**

Les gardes de navigation peuvent aussi être définis directement dans les composants.

```javascript
// src/components/Dashboard.vue
<template>
    <div>
        <h1>Dashboard</h1>
    </div>
</template>

<script>
    export default {
    beforeRouteEnter(to, from, next) {
    // Ce guard est appelé avant l'entrée dans le composant
    console.log('Navigation vers Dashboard');
    next();
},
    beforeRouteLeave(to, from, next) {
    // Ce guard est appelé avant de quitter le composant
    const answer = window.confirm('Voulez-vous vraiment quitter cette page ?');
    if (answer) {
    next(); // Permet la navigation
} else {
    next(false); // Annule la navigation
}
}
}
</script>

```
**Explication :**
- beforeEach (global) : Vérifie si l'utilisateur est authentifié avant chaque navigation. Si une route a un meta.requiresAuth, il redirige vers /login si l'utilisateur n'est pas connecté.
- beforeEnter (par route) : Assure qu'une vérification supplémentaire est faite avant d'entrer dans certaines routes, comme la route /admin.
- beforeRouteEnter et beforeRouteLeave (dans un composant) : Fournissent des contrôles sur la navigation à partir d'un composant spécifique.
Ces gardes permettent de gérer efficacement l'accès et les redirections dans votre application Vue.js.