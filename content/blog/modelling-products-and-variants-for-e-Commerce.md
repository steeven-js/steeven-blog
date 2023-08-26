---
external: false
title: "Modelling Products and Variants for E-Commerce"
description: "Database design for e-commerce product variants with laravel."
date: 2023-08-26
---

## Products and SKUs

```php
public function up()
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->text('description')->nullable();
        $table->timestamps();
    });
}
```

```php
public function up()
{
    Schema::create('skus', function (Blueprint $table) {
        $table->id();
        $table->foreignId('product_id')->constrained()->cascadeOnDelete();
        $table->string('sku')->unique();
        $table->char('currency_code', 3);
        $table->unsignedInteger('unit_amount');
        $table->timestamps();
    });
}
```

## Product attributes

```php
public function up()
{
    Schema::create('product_attributes', function (Blueprint $table) {
        $table->id();
        $table->foreignId('product_id')->constrained()->cascadeOnDelete();
        $table->string('name');
        $table->timestamps();

        $table->unique(['product_id', 'name']);
    });
}
```

```php
public function up()
{
    Schema::create('product_attribute_sku', function (Blueprint $table) {
        $table->primary(['product_attribute_id', 'sku_id']);
        $table->foreignId('product_attribute_id')->constrained()->cascadeOnDelete();
        $table->foreignId('sku_id')->constrained()->cascadeOnDelete();
        $table->string('value');
    });
}
```

## ProductSeeder

```php
<?php

namespace Database\Seeders;

use App\Models\Sku;
use App\Models\Product;
use Illuminate\Support\Str;
use Illuminate\Database\Seeder;
use App\Models\ProductAttribute;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;

class ProductSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run()
    {
        $productName = 'iPhone 14';
        $sizes = ['6.1 pouces', '6.7 pouces'];
        $capacities = ['128 Go', '256 Go', '512 Go', '1024 Go'];
        $colors = ['Argent', 'Noir', 'Or', 'Violet'];

        // Créer une instance du produit avec le nom et la description
        $product = Product::create(['name' => $productName, 'description' => 'Description du produit']);

        // Créer une instance d'attribut pour la taille d'écran
        $attributeSize = ProductAttribute::create(['product_id' => $product->id, 'name' => 'Taille d\'écran']);

        // Créer une instance d'attribut pour la capacité
        $attributeCapacity = ProductAttribute::create(['product_id' => $product->id, 'name' => 'Capacité']);

        // Créer une instance d'attribut pour la couleur
        $attributeColor = ProductAttribute::create(['product_id' => $product->id, 'name' => 'Couleur']);

        foreach ($sizes as $size) {
            foreach ($capacities as $capacity) {
                foreach ($colors as $color) {
                    // Générer un code SKU unique en fonction des informations données
                    $skuCode = $this->generateSkuCode($productName, $size, $capacity, $color);

                    // Créer une instance de SKU avec le code SKU, l'ID du produit, le code de devise et le montant unitaire
                    $sku = Sku::create([
                        'sku' => $skuCode,
                        'product_id' => $product->id,
                        'currency_code' => 'USD',
                        'unit_amount' => 100
                    ]);

                    // Attacher les attributs appropriés aux SKU
                    $sku->attributes()->attach($attributeSize, ['value' => $size]);
                    $sku->attributes()->attach($attributeCapacity, ['value' => $capacity]);
                    $sku->attributes()->attach($attributeColor, ['value' => $color]);
                }
            }
        }
    }

    /**
     * Génère un code SKU unique en fonction des informations données.
     */
    private function generateSkuCode($productName, $size, $capacity, $color)
    {
        return Str::slug($productName) . '-' . Str::slug($size) . '-' . Str::slug($capacity) . '-' . Str::slug($color);
    }
}
```
## Models

### Product

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'description'];

    public function attributes()
    {
        return $this->hasMany(ProductAttribute::class);
    }

    public function skus()
    {
        return $this->hasMany(Sku::class);
    }
}
```

### ProductAttribute

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class ProductAttribute extends Model
{
    use HasFactory;

    protected $fillable = ['name'];

    public function product()
    {
        return $this->belongsTo(Product::class);
    }

    public function skus()
    {
        return $this->belongsToMany(Sku::class, 'product_attribute_skus')
            ->withPivot('value');
    }
}
```

### Sku

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Sku extends Model
{
    use HasFactory;

    protected $fillable = ['sku', 'currency_code', 'unit_amount'];

    public function product()
    {
        return $this->belongsTo(Product::class);
    }

    public function attributes()
    {
        return $this->belongsToMany(ProductAttribute::class, 'product_attribute_skus')
            ->withPivot('value');
    }
}
```
