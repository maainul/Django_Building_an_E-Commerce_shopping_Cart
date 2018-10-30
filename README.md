# Django_Building_an_E-Commerce_shopping_Cart

# Building an E-Commerce Shopping Cart (Using Django 2.0 and Python 3.6)
```
    1. Lesson 1  : Building an E-Commerce Shopping Cart 
    2. Lesson 2  : Creating Our Shop app and designing product model 
    3. Lesson 3  : Registering our models in admin site and creating views 
    4. Lesson 4  : Defining URL patterns and template for our shop products
    5. Lesson 5  : Completing product detail template for Our Shop products 
    6. Lesson 6  : Developing Shopping cart class for Our Shop products 
    7. Lesson 7  : Creating Shopping Cart views for Our Shop products
    8. Lesson 8  : Developing a context processor for our current cart
    9. Lesson 9  : Persisting customer orders to the database
    10.Lesson 10 : Completing persisting customer orders to database
```

## Lesson 1   : Building an E-Commerce Shopping Cart 

### 1.1.Creating Virtual Environment && Project
```
    mkdir dev && cd dev
    mkdir onlineshop && cd onlineshop
    virtualenv -p python3 .
    source bin/activate
    pip install Django==2.0.7
    mkdir src && cd src
    django-admin startproject ecommerce .
    python manage.py runsrver
    
```
# Lesson 2: Creating Our Shop app and designing product model 

## 2.1. Add apps to the project:
```
	django-admin startapp shop
```
## 2.2. Edit settings.py:
```
INSTALLED_APPS = [
	    'shop.apps.ShopConfig',
]
```

## 2.2. Install pillow in virtual environment
```	
pip install pillow 
```
## 2.3. shop/models.py: 
```
class Category(models.Model):
    name = models.CharField(max_length=150, db_index=True)
    slug = models.SlugField(max_length=150, unique=True ,db_index=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
 
    class Meta:
        ordering = ('name',)
        verbose_name = 'category'
        verbose_name_plural = 'categories'
 
    def __str__(self):
        return self.name
 
class Product(models.Model):
    category = models.ForeignKey(Category, related_name='products', on_delete=models.CASCADE)
    name = models.CharField(max_length=100, db_index=True)
    slug = models.SlugField(max_length=100, db_index=True)
    description = models.TextField(blank=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    available = models.BooleanField(default=True)
    stock = models.PositiveIntegerField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    image = models.ImageField(upload_to='products/%Y/%m/%d', blank=True)
 
    class Meta:
        ordering = ('name', )
        index_together = (('id', 'slug'),)
 
    def __str__(self):
        return self.name
```

## Description of fields
```
Category Model  (The category model consists of four fields)

name – which has a maximum length of 150 characters
slug – which is a unique field, this help us in building canonical urls later.
created_at – which tracks when category was created.
updated_at – which tracks when category was updated.

Product Model (The product model consists of ten fields)

name – which has a maximum length of 100 characters.
category – which is foreign-key pointing to our category model. A product belongs to a category. In this field we have added on_delete argument which is the behavior to adopt when referenced object is deleted.
slug – which we will use for building seo friendly url.
description – which is used to describe a product.
price – which used for holding price of a product. We are using Decimal Field to avoid rounding off issues.
image – used to hold image of a product.
available – this is boolean field used to show whether product is available or not.
stock – this is positive integer to show number of stocks of given product.
created_at – which tracks when product was created.
updated_at – which tracks when product was updated.


NB: In meta class of our product model, we are using index_together meta option to specify an index for id and slug fields. This will help improve performances of queries.
```
## 2.4. Run command:
```
Python manage.py makemigrations
Python manage.py migrate
```
## Lesson 3: Registering our models in admin site and creating views

## 3.1. shop/Admin.py
```
from django.contrib import admin
from .models import Category, Product
 
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug']
    prepopulated_fields = {'slug': ('name',)}
 
admin.site.register(Category, CategoryAdmin)
 
class ProductAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug', 'price', 'stock', 'available', 'created_at', 'updated_at']
    list_filter = ['available', 'created_at', 'updated_at']
    list_editable = ['price', 'stock', 'available']
    prepopulated_fields = {'slug': ('name',)}
 
admin.site.register(Product, ProductAdmin)
```

## 3.2. Create User:
```
python manage.py createsuperuser
```

## 3.3. shop/views.py 
```
from django.shortcuts import render, get_object_or_404
from .models import Category, Product
 
 
def product_list(request, category_slug=None):
    category = None
    categories = Category.objects.all()
    products = Product.objects.filter(available=True)
    if category_slug:
        category = get_object_or_404(Category, slug=category_slug)
        products = Product.objects.filter(category=category)
 
    context = {
        'category': category,
        'categories': categories,
        'products': products
    }
    return render(request, 'shop/product/list.html', context)
 
 
def product_detail(request, id, slug):
    product = get_object_or_404(Product, id=id, slug=slug, available=True)
    context = {
        'product': product
    }
    return render(request, 'shop/product/detail.html', context)
```

## Let’s Understand the code:
```
On line 1, I am importing shortcut methods to render my template and to check whether an object is found or to raise a 404 error.
On line 2 – I am importing my Category and Product model.
On line 5 – I am creating a function called product_list which I will use to display a list of products. In this function, I am also passing a second parameter called category_slug=None which I will use if products are filtered using a given category by our users.
On line 6 – I am specifying category is equal to none, in order to show the list of products without filtering them by any category.
On line 7 – I am querying and fetching all the categories from my database.
On line 8 – I am fetching all the products from my database which are available by passing(available=True) filter to my queryset.
On line 9 – line 11 I am creating an if statement to optionally filter the products based on the category_slug parameter passed from my url.
On line 10 – I am creating a variable called category and I am creating a queryset to the Category model and filtering the result with category_slug parameter.
On line 11 – I am creating a queryset to fetching products and filtering the products based on the category from line 10.
From line 13 – 17 I am simply creating a data dictionary to pass to my template.
On line 18 – I am rendering a template called list.html which I am yet to create and also passing data context.
We also need a view to retrieve and display a single product. On line 21 we create a product_detail function where we are passing the id and slug of the product as parameters to this method.
On line 22 – I am creating queryset and I am using id and slug to retrieve product instance. I am also checking whether the product is available or not.

NB: We can get this instance by just the ID since it’s a unique attribute. However, we include the slug in the URL to build SEO-friendly URLs for products.
From line 23 – 25 I am creating a dictionary of my data in order to pass this data to my template using a single context.
On line 26 I am rendering a template called detail.html which I am yet to create and passing the context data to this template.
```


