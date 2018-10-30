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
