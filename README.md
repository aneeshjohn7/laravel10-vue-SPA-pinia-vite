

## Single-Page Authentication using Laravel 10 Sanctum, Vue 3, Pinia and Vite


A complete register and login feature for a single-page application with Laravel 10 Sanctum, Bootstrap5, Vue 3, Pinia and Vite.


## Install and configure Laravel

If you're developing on a Mac and the Docker Desktop is already installed, you can use a simple terminal command to create a new Laravel project. 
```
curl -s "https://laravel.build/laravel10-vue-SPA-pinia-vite" | bash
```
After the project has been created, you can navigate to the application directory and start Laravel Sail. Laravel Sail provides a simple command-line interface for interacting with Laravel's default Docker configuration:
```
cd laravel10-vue-SPA-pinia-vite && ./vendor/bin/sail up
```
Install authentication scaffolding with Vue and Bootstrap using the following commands

```
composer require laravel/ui
php artisan ui vue --auth
```

Run the following commands to compile your fresh scaffolding.

```
npm install && npm run dev
```
To run all of your outstanding migrations, execute the migrate Artisan command:
```
php artisan migrate
```
We are using Pinia for the Vue state management here. Install pinna using the below command
```
npm install pinia
```
To validate the form using Vue we are using vee-validate plugin. Please install the below commands
```
npm install vee-validate --save
npm install @vee-validate/rules
```

Install eslint to ensure Vue code quality
```
npm install eslint --save-dev
```
Run the following command. This command will instruct the vite to run the code through eslint for code quality.
```
npm install vite-plugin-eslint --save-dev --force
```
Let's install vue-router. Vue Router helps link between the browser's URL / History and Vue's components allowing for certain paths to render whatever view is associated with it.
```
npm install vue-router
```

Next, if you plan to utilize Sanctum to authenticate a SPA, you should add Sanctum's middleware to your api middleware group within your application's app/Http/Kernel.php file:
```
'api' => [
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
    \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],
```
Open .env file and add these two line
```
SESSION_DOMAIN=127.0.0.1
SANCTUM_STATEFUL_DOMAINS=localhost,127.0.0.1,localhost:8000,127.0.0.1:8000
```
In .env, update session driver file to cookie.
```
SESSION_DRIVER=cookie
```
Configure CORS
Open config/cors.php and update the following code into the file:
```
'paths' => [
    'api/*',
    '/login',
    '/logout',
    '/sanctum/csrf-cookie'
],
```

Run the following artisan command to create a User controller
```
php artisan make:controller UserController
```
Add the below line of codes to UserController.php
```
<?php
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;

class UserController extends Controller
{
   /**
    * Store a new user.
    *
    * @param  \Illuminate\Http\Request  $request
    * @return \Illuminate\Http\Response
    */
   public function register(Request $request)
   {
       $request->validate([
           'name' => 'required|string|max:250',
           'email' => 'required|email|max:250|unique:users',
           'password' => 'required|min:8|confirmed'
       ]);

       User::create([
           'name' => $request->name,
           'email' => $request->email,
           'password' => Hash::make($request->password)
       ]);

       $credentials = $request->only('email', 'password');
       Auth::attempt($credentials);
       $request->session()->regenerate();
       return response()->json([
           'status' => 'success',
           'message' => 'You have successfully registered & logged in!',
       ]);
   }

   /**
    * Authenticate the user.
    *
    * @param  \Illuminate\Http\Request  $request
    * @return \Illuminate\Http\Response
    */
   public function login(Request $request)
   {
       $credentials = $request->validate([
           'email' => 'required|email',
           'password' => 'required'
       ]);

       if(Auth::attempt($credentials))
       {
           $request->session()->regenerate();
           return response()->json([
               'status' => 'success',
               'data' => Auth::user(),
               'message' => 'Successfully Logged In!',
           ]);
       }

       return response()->json([
           'status' => 'error',
           'message' => 'Your provided credentials do not match in our records.',
       ]);

   }

   /**
    * Fetch user data
    */
   public function user()
   {
       return response()->json([
           'data' => Auth::user()
       ]);
   }

   /**
    * Log out the user from the application.
    *
    * @param  \Illuminate\Http\Request  $request
    * @return \Illuminate\Http\Response
    */
   public function logout(Request $request)
   {
       Auth::logout();
       $request->session()->invalidate();
       $request->session()->regenerateToken();
       return response()->json([
           'status' => 'success',
           'message' => 'Logout',
       ]);
   }  
}
```

Replace scripts in your routes/web.php with the below line of codes
```
Route::get('{any}', function () {
   return view('welcome');
})->where('any', '.*');
```
Open your routes/api.php and add the below line of codes
```
use App\Http\Controllers\UserController;
Route::controller(UserController::class)->group(function() {
   Route::post('/register', 'register')->name('register');
   Route::post('/login', 'login')->name('login');
   Route::post('/logout', 'logout')->name('logout');
});
```

Create view components
Create the below view component files in your resources/js/components directory and paste the associated scrips
Create Login.vue and paste the below code

```
<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">Login</div>

                    <div class="card-body">
                        <vee-form
                            :validation-schema="schema"
                            @submit="login"
                            method="POST"
                        >
                            <div class="row" v-if="this.errorInfo">
                                <div class="col-md-12 text-danger text-center">
                                    <p class="mb-1">{{ this.errorInfo }}</p>
                                </div>
                            </div>
                            <div class="row mb-3">
                                <label
                                    for="email"
                                    class="col-md-4 col-form-label text-md-end"
                                    >Email Address</label
                                >
                                <div class="col-md-6">
                                    <vee-field
                                        id="email"
                                        type="email"
                                        v-model="auth.email"
                                        class="form-control"
                                        name="email"
                                        required
                                        autocomplete="email"
                                        autofocus
                                    />
                                    <ErrorMessage
                                        class="text-danger"
                                        name="email"
                                    />
                                </div>
                            </div>

                            <div class="row mb-3">
                                <label
                                    for="password"
                                    class="col-md-4 col-form-label text-md-end"
                                    >Password</label
                                >

                                <div class="col-md-6">
                                    <vee-field
                                        id="password"
                                        v-model="auth.password"
                                        type="password"
                                        class="form-control"
                                        name="password"
                                        required
                                    />
                                    <ErrorMessage
                                        class="text-danger"
                                        name="password"
                                    />
                                </div>
                            </div>

                            <div class="row mb-3">
                                <div class="col-md-6 offset-md-4">
                                    <div class="form-check">
                                        <input
                                            class="form-check-input"
                                            type="checkbox"
                                            name="remember"
                                            id="remember"
                                        />

                                        <label
                                            class="form-check-label"
                                            for="remember"
                                        >
                                            Remember Me
                                        </label>
                                    </div>
                                </div>
                            </div>

                            <div class="row mb-0">
                                <div class="col-md-8 offset-md-4">
                                    <button
                                        type="submit"
                                        class="btn btn-primary"
                                    >
                                        Login
                                    </button>
                                </div>
                            </div>
                        </vee-form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import { mapActions, mapWritableState } from "pinia";
import useUserStore from "@/stores/user";
export default {
    name: "Login",
    computed: {
        ...mapWritableState(useUserStore, ["errorInfo"]),
    },
    data() {
        return {
            schema: {
                email: "required|email",
                password: "required|min:5|max:25",
            },
            auth: {
                email: "",
                password: "",
            },
        };
    },
    methods: {
        ...mapActions(useUserStore, ["authenticate"]),

        login(values) {
            this.authenticate(values);
        },
    },
};
</script>
```

Create Register.vue and paste the below code

```
<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">Register</div>

                    <div class="card-body">
                        <vee-form
                            :validation-schema="schema"
                            @submit="register"
                        >
                            <div class="row" v-if="errorInfo">
                                <div class="col-md-12 text-danger text-center">
                                    <p class="mb-1">{{ errorInfo }}</p>
                                </div>
                            </div>
                            <div class="row mb-3">
                                <label
                                    for="name"
                                    class="col-md-4 col-form-label text-md-end"
                                    >Name</label
                                >

                                <div class="col-md-6">
                                    <vee-field
                                        id="name"
                                        type="text"
                                        class="form-control"
                                        name="name"
                                        v-model="user.name"
                                        required
                                        autocomplete="name"
                                        autofocus
                                    />
                                    <ErrorMessage
                                        class="text-danger"
                                        name="name"
                                    />
                                </div>
                            </div>

                            <div class="row mb-3">
                                <label
                                    for="email"
                                    class="col-md-4 col-form-label text-md-end"
                                    >Email Address</label
                                >

                                <div class="col-md-6">
                                    <vee-field
                                        id="email"
                                        type="email"
                                        class="form-control"
                                        name="email"
                                        v-model="user.email"
                                        required
                                        autocomplete="email"
                                    />
                                    <ErrorMessage
                                        class="text-danger"
                                        name="email"
                                    />
                                </div>
                            </div>

                            <div class="row mb-3">
                                <label
                                    for="password"
                                    class="col-md-4 col-form-label text-md-end"
                                    >Password</label
                                >

                                <div class="col-md-6">
                                    <vee-field
                                        id="password"
                                        type="password"
                                        class="form-control"
                                        name="password"
                                        v-model="user.password"
                                        required
                                        autocomplete="new-password"
                                    />
                                    <ErrorMessage
                                        class="text-danger"
                                        name="password"
                                    />
                                </div>
                            </div>

                            <div class="row mb-3">
                                <label
                                    for="password-confirm"
                                    class="col-md-4 col-form-label text-md-end"
                                    >Confirm Password</label
                                >

                                <div class="col-md-6">
                                    <vee-field
                                        id="password-confirm"
                                        type="password"
                                        class="form-control"
                                        name="password_confirmation"
                                        v-model="user.password_confirmation"
                                        required
                                        autocomplete="new-password"
                                    />
                                    <ErrorMessage
                                        class="text-danger"
                                        name="password_confirmation"
                                    />
                                </div>
                            </div>

                            <div class="row mb-0">
                                <div class="col-md-6 offset-md-4">
                                    <button
                                        type="submit"
                                        class="btn btn-primary"
                                    >
                                        Registers
                                    </button>
                                </div>
                            </div>
                        </vee-form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import { mapActions } from "pinia";
import useUserStore from "@/stores/user";
import router from "@/router";
export default {
    name: "Register",
    data() {
        return {
            user: {
                name: "",
                email: "",
                password: "",
                password_confirmation: "",
            },
            schema: {
                name: "required|min:5|max:50|alpha_spaces",
                email: "required|email",
                password: "required|min:5|max:20",
                password_confirmation: "required|confirmed:@password",
            },
            errorInfo: "",
        };
    },
    methods: {
        ...mapActions(useUserStore, ["authCheck"]),
        register() {
            axios.get("/sanctum/csrf-cookie").then((response) => {
                axios
                    .post("/api/register", this.user)
                    .then((response) => {
                        if (response.data.status === "success")
                            //this.authCheck();
                            this.$router.push({ name: "dashboard" });
                    })
                    .catch((error) => {
                        if (error.response.status === 422) {
                            this.errorInfo = error.response.data.message;
                        }
                    });
            });
        },
    },
};
</script>
```
Create Header.vue and paste the below script

```
<template>
    <ul class="navbar-nav ms-auto">
        <!-- Authentication Links -->
        <template v-if="!this.userStore.isLoggedIn">
            <li class="nav-item">
                <router-link class="nav-link" :to="{ name: 'login' }"
                    >Login</router-link
                >
            </li>

            <li class="nav-item">
                <router-link class="nav-link" :to="{ name: 'register' }"
                    >Register</router-link
                >
            </li>
        </template>
        <template v-else>
            <li class="nav-item">
                <router-link class="nav-link" :to="{ name: 'about' }"
                    >About</router-link
                >
            </li>

            <li class="nav-item">
                <router-link class="nav-link" :to="{ name: 'contact' }"
                    >Contact</router-link
                >
            </li>
            <li class="nav-item dropdown">
                <a
                    id="navbarDropdown"
                    class="nav-link dropdown-toggle"
                    role="button"
                    data-bs-toggle="dropdown"
                    aria-haspopup="true"
                    aria-expanded="false"
                >
                    {{ userStore.user.name }}
                </a>

                <div
                    class="dropdown-menu dropdown-menu-end"
                    aria-labelledby="navbarDropdown"
                >
                    <a class="dropdown-item" @click.prevent="logout" href="#">
                        Logout
                    </a>
                </div>
            </li>
        </template>
    </ul>
</template>
<script>
import { mapStores } from "pinia";
import useUserStore from "@/stores/user";
export default {
    name: "Header",

    computed: {
        ...mapStores(useUserStore),
    },
    methods: {
        logout() {
            axios
                .post("/api/logout")
                .then((response) => {
                    if (response.data.status == "success") {
                        this.userStore.logout();
                    }
                })
                .catch((error) => {
                    console.log(error);
                });
        },
    },
};
</script>
```
Create Guest.vue and paste the below script
```
<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">Laravel SPA</div>

                    <div class="card-body">
                        Sinle Page Application (Authentication) in Laravel and Vue JS.
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
```
Create Dashboard.vue and paste the below script
```
<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">Laravel SPA Dashboard</div>

                    <div class="card-body">
                        You are logged in as {{ this.userStore.user.name }}
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import { mapStores } from "pinia";
import useUserStore from "@/stores/user";
export default {
    name: "Dashboard",
    computed: {
        ...mapStores(useUserStore),
    },
};
</script>
```
Create Contact.vue and add the below script
```
<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">Contact</div>

                    <div class="card-body">Sample text</div>
                </div>
            </div>
        </div>
    </div>
</template>
```
Create About.vue and add the below script
```
<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">About</div>

                    <div class="card-body">sample text</div>
                </div>
            </div>
        </div>
    </div>
</template>
```
Form validation - Login and Register
Create a new folder named includes in your resources/js directory and create a file in the includes folder named validation.js and add the below code 
```
import {
    Form as VeeForm,
    Field as VeeField,
    defineRule,
    ErrorMessage,
} from "vee-validate";
import {
    required,
    email,
    min,
    max,
    alpha_spaces,
    confirmed,
} from "@vee-validate/rules";
export default {
    install(app) {
        app.component("VeeForm", VeeForm);
        app.component("VeeField", VeeField);
        app.component("ErrorMessage", ErrorMessage);

        defineRule("required", required);
        defineRule("email", email);
        defineRule("min", min);
        defineRule("max", max);
        defineRule("alpha_spaces", alpha_spaces);
        defineRule("confirmed", confirmed);
    },
};
```
View Router
Create a directory named router in your resources/js directory and create a file named index.js in it and paste the below code
```
import { createWebHistory, createRouter } from "vue-router";
import Login from "@/components/Login.vue";
import Register from "@/components/Register.vue";
import Guest from "@/components/Guest.vue";
import Dashboard from "@/components/Dashboard.vue";
import About from "@/components/About.vue";
import Contact from "@/components/Contact.vue";
import useUserStore from "@/stores/user";

const routes = [
    {
        name: "guest",
        path: "/",
        component: Guest,
        meta: {
            middleware: "guest",
            title: "Guest",
        },
    },
    {
        name: "login",
        path: "/login",
        component: Login,
        meta: {
            middleware: "guest",
            title: "Login",
        },
    },
    {
        name: "register",
        path: "/register",
        component: Register,
        meta: {
            middleware: "guest",
            title: "Register",
        },
    },
    {
        name: "dashboard",
        path: "/dashboard",
        component: Dashboard,
        meta: {
            middleware: "auth",
            title: "Dashboard",
        },
    },
    {
        name: "about",
        path: "/about",
        component: About,
        meta: {
            middleware: "auth",
            title: "About",
        },
    },
    {
        name: "contact",
        path: "/contact",
        component: Contact,
        meta: {
            middleware: "auth",
            title: "Contact",
        },
    },
];

const router = createRouter({
    history: createWebHistory(),
    routes, // short for `routes: routes`
});

router.beforeEach((to, from, next) => {
    document.title = to.meta.title
    const store = useUserStore();
    store.authCheck().then((data) => {
        if (to.meta.middleware) {
            if (to.meta.middleware == "guest") {
                if (store.isLoggedIn) {
                    next({ name: "dashboard" });
                }
                next();
            } else {
                if (store.isLoggedIn) {
                    next();
                } else {
                    next({ name: "login" });
                }
            }
        }
    });
});

export default router;
```
State management 


Create a new directory stores in your resources/js directory and create a new file user.js and paster the below code
```
import { defineStore } from "pinia";
import router from "@/router";
export default defineStore("user", {
    state: () => ({
        isLoggedIn: false,
        errorInfo: "",
        user: {},
    }),
    getters: {
        setLoggedIn(state) {
            return state.isLoggedIn;
        },
    },
    actions: {
        authenticate(values) {
            return axios.get("/sanctum/csrf-cookie").then((response) => {
                axios
                    .post("/api/login", values)
                    .then((response) => {
                        this.user = response.data.data;

                        if (response.data.status == "success") {
                            this.errorInfo = "";
                            router.push({ name: "dashboard" });
                        } else {
                            this.errorInfo = response.data.message;
                            console.log(this.errorInfo);
                        }
                    })
                    .catch((error) => {
                        console.log(error);
                        if (error.response.status === 422) {
                            this.errorInfo = error.response.data.message;
                        }
                    }); // credentials didn't match
            });
        },
        async authCheck() {
            await axios
                .get("/api/user")
                .then((response) => {
                    if (response.data) {
                        this.user = response.data;

                        this.isLoggedIn = true;
                    } else {
                        this.user = {};
                        this.isLoggedIn = false;
                    }
                })
                .catch((error) => {
                    return error;
                });
        },
        logout() {
            this.isLoggedIn = false;
            router.push({ name: "login" });
        },
    },
});
```
Alter your resources/js/app.js like below 
```
/**
 * First we will load all of this project's JavaScript dependencies which
 * includes Vue and other libraries. It is a great starting point when
 * building robust, powerful web applications using Vue and Laravel.
 */

import "./bootstrap";
import { createApp } from "vue";
import { createPinia } from "pinia";
import router from "@/router";
import VeeValidatePlugin from "@/includes/validation";

/**
 * Next, we will create a fresh Vue application instance. You may then begin
 * registering components with the application instance so they are ready
 * to use in your application's views. An example is included for you.
 */

const app = createApp({});

import Header from "./components/Header.vue";
import Login from "./components/Login.vue";
import Register from "./components/Register.vue";
import About from "./components/about.vue";
import Contact from "./components/contact.vue";
import Dashboard from "./components/Dashboard.vue";
app.component("header-block", Header);
app.component("login", Login);
app.component("register", Register);
app.component("dashboard", Dashboard);
app.component("about", About);
app.component("contact", Contact);
app.use(createPinia());
app.use(router);
app.use(VeeValidatePlugin);
/**
 * The following block of code may be used to automatically register your
 * Vue components. It will recursively scan this directory for the Vue
 * components and automatically register them with their "basename".
 *
 * Eg. ./components/ExampleComponent.vue -> <example-component></example-component>
 */

// Object.entries(import.meta.glob('./**/*.vue', { eager: true })).forEach(([path, definition]) => {
//     app.component(path.split('/').pop().replace(/\.\w+$/, ''), definition.default);
// });

/**
 * Finally, we will attach the application instance to a HTML element with
 * an "id" attribute of "app". This element is included with the "auth"
 * scaffolding. Otherwise, you will need to add an element yourself.
 */

app.mount("#app");
```
Replace the scripts your views/layouts/app.blade.php with the below script
```
<!doctype html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- CSRF Token -->
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ config('app.name', 'Laravel') }}</title>

    <!-- Fonts -->
    <link rel="dns-prefetch" href="//fonts.bunny.net">
    <link href="https://fonts.bunny.net/css?family=Nunito" rel="stylesheet">

    <!-- Scripts -->
    @vite(['resources/sass/app.scss', 'resources/js/app.js'])
</head>
<body>
    <div id="app" v-cloak>
        <nav class="navbar navbar-expand-md navbar-light bg-white shadow-sm">
            <div class="container">
                <a class="navbar-brand" href="{{ url('/') }}">
                    {{ config('app.name', 'Laravel') }}
                </a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="{{ __('Toggle navigation') }}">
                    <span class="navbar-toggler-icon"></span>
                </button>

                <div class="collapse navbar-collapse" id="navbarSupportedContent">
                    <!-- Left Side Of Navbar -->
                    <ul class="navbar-nav me-auto">

                    </ul>

                    <!-- Right Side Of Navbar -->
                    <header-block></header-block>
                </div>
            </div>
        </nav>
        
        <main class="py-4">
            @yield('content')
        </main>
    </div>
</body>
</html>
```

If you have any suggestions, please send an e-mail to Aneesh John via [aneesh85@gmail.com](mailto:aneesh85@gmail.com).

