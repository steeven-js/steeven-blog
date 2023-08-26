---
external: false
title: "Laravel E-commerce Vue"
description: "Laravel E-commerce Vue"
date: 2023-08-23
---

## Laravel E-commerce Vue

```sh
composer create-project laravel/laravel laravel-ecommerce-vue && cd laravel-ecommerce-vue
```

```sh
composer require laravel/breeze --dev  
php artisan breeze:install
```
```sh
php artisan make:model Product -mfc
```
```php
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->text('description');
        $table->string('image');
        $table->string('slug');
        $table->integer('price');
        $table->boolean('active');
        $table->timestamps();
    });
```
```php
    public function definition()
    {
        // $table->string('name');
        // $table->text('description');
        // $table->string('slug');
        // $table->integer('price');
        // $table->boolean('active');

        $name = $this->faker->sentence();

        return [
            'name' => $name,
            'description' => $this->faker->sentences(rand(2, 5), true),
            'image' => $this->faker->imageUrl(),
            'slug' => Str::slug($name),
            'price' => rand(500, 10000),
            'active' => $this->faker->boolean(80)
        ];
    }
```

```php
    // web.php

    Route::get('products', [ProductController::class, 'index'])
        ->name('cart.index');
```

```php
// ProductController.php

    public function index()
    {
        $products = Product::inRandomOrder()
            ->whereActive(true)
            ->take(16)
            ->get();

        return view('products.index', compact('products'));
    }
```

```sh
npm create vite@latest
```

```js
    // App.js

    require('./bootstrap');

    import Alpine from 'alpinejs';
    import { createApp } from 'vue';

    window.Alpine = Alpine;

    Alpine.start();

    const app = createApp();

    app.mount('#app');
```

```html
    <body class="font-sans antialiased" id="app">
```

-Créer component/AddToCart.vue

