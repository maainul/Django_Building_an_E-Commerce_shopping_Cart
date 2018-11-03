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

# Lesson 4: Defining Url patterns and template for Our Shop products

## 4.1. Create a file inside shop apps name urls.py

## shop/urls.py:
```
from django.conf.urls import url
from . import views

app_name = 'shop'

urlpatterns = [
    url(r'^$', views.product_list, name='product_list'),
    url(r'^(?P<category_slug>[-\w]+)/$', views.product_list,name='product_list_by_category'),
    url(r'^(?P<id>\d+)/(?P<slug>[-\w]+)/$', views.product_detail, name='product_detail'),
]
```
## 4.2.Let’s understand the code in urls.py file:
```
      Line 1 – We import url from django urls
      Line 2 – We import methods in our views.py file
      Line 4 – We create a namespace for our shop application.
      Line 6 to 10 – We create a python list for our url patterns for our application.
      Line 7 – This url is mapped to our product_list method and it will fetch all products.
      Line 8 – This url is also mapped to product_list method but we pass category_slug as our parameter hence, products we be filtered by a given category.
Line 9 – This url is mapped to product_detail method and this will fetch a specific product.
```

## 4.3. Include urls in the main apps urls.py:
```
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('shop.urls')),
] 
```
## 4.4. shop/models.py
```
from django.db import models
from django.urls import reverse
 
 
class Category(models.Model):
    name = models.CharField(max_length=150, db_index=True)
    slug = models.SlugField(max_length=150, unique=True ,db_index=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
 
    class Meta:
        ordering = ('name', )
        verbose_name = 'category'
        verbose_name_plural = 'categories'
 
    def __str__(self):
        return self.name
 
    def get_absolute_url(self):
        return reverse('shop:product_list_by_category', args=[self.slug])
 
 
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
 
    def get_absolute_url(self):
        return reverse('shop:product_detail', args=[self.id, self.slug])


```
## Lets understand what we have done:
```
       Line 2 – We import reverse function from django urls.
       Line 19 and 20 – We have created get_absolute_url method to create SEO-friendly url from our shop url patterns.
       Line 42 and 43 – We have created get_absolute_url method to create SEO-friendly url from our shop url patterns. 	    	   Remember the app_name we defined in the shop app urls.py file, that is how we use it in the get_absolute_url()
```
## 4.5.Creating Product Templates
```
in our shop app, create a folder called templates.
Inside templates folder, create another folder called shop,
inside shop folder create another folder called product.
Inside product folder create list.html,detail.html,base.html and nav.html

Go inside the shop add and run these command:

(onlineshop) ➜   mkdir templates     
(onlineshop) ➜   mkdir templates/shop
(onlineshop) ➜   mkdir templates/shop/product
(onlineshop) ➜   touch templates/shop/product/detail.html
(onlineshop) ➜   touch templates/shop/product/list.html
(onlineshop) ➜   touch templates/shop/base.html
(onlineshop) ➜   touch templates/shop/navbar.html 
```
## 4.5. base.html
```
<!DOCTYPE html>
{% load staticfiles %}
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}On-line Shop{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
    <link rel="stylesheet" href="{% static 'css/styles.css' %}">
</head>
<body>
{% include 'shop/navbar.html' %}
 
{% block content %}
 
{% endblock %}
 
<script src="{% static 'js/jquery.min.js' %}" type="text/javascript"></script>
<script src="{% static 'js/bootstrap.min.js' %}" type="text/javascript"></script>
</body>
</html>
```


## Lets understand the code in this file:
```
       Line 7 and 8 – We are loading bootstrap.min.css and styles.css in this file.
       Line 17 and 18 – We are loading jquery and bootstrap javascript in this file.
       Line 11 – We are including our navbar.html file so that it’s available for use.
       
NB: In order to use the required css and javascript in this file, you need to download bootstrap and jquery. In my case I am using bootstrap 4, after downloading bootstrap and jquery.
Inside our shop application, 
create a folder called static. 
Inside static folder,
create css and js folder.
Place bootstrap css inside css folder and bootstrap js and jquery js in inside js folder. 
Your folder should look like this.

(onlineshop) ➜   mkdir static     
(onlineshop) ➜   mkdir static/css
(onlineshop) ➜   mkdir static/js     
(onlineshop) ➜   mkdir templates/img
```
                                    

## 4.6. shop/product/list.html
```
{% extends 'shop/base.html' %}
{% load static %}
{% block title %}
    {% if category %}{{ category.name }} {% else %} Products {% endif %}
{% endblock %}
 
{% block content %}
   <div class="container-fluid">
      <div class="row" style="margin-top: 6%">
 
        <div class="col-sm-8 blog-main">
 
          <div class="blog-post">
              <div class="row">
                  {% for product in products %}
                      <div class="col-md-4">
                        <div class="thumbnail">
                            <a href="{{ product.get_absolute_url }}">
                                <img src="{% if product.image %} {{ product.image.url }} {% else %} {% static 'img/default.jpg' %} {% endif %}" alt="..." style="height: 130px; width: auto">
                            </a>
                            <div class="caption">
                                <h3 class="text-center">
                                    <a href="{{ product.get_absolute_url }}">{{ product.name }}</a>
                                </h3>
                                <p class="text-center">Kshs. {{ product.price }}</p>
                            </div>
                        </div>
                      </div>
                  {% endfor %}
              </div>
 
 
          </div><!-- /.blog-post -->
 
        </div><!-- /.blog-main -->
 
        <div class="col-sm-3 col-sm-offset-1 blog-sidebar">
          <div class="sidebar-module">
            <h4>Categories</h4>
            <ol class="list-unstyled">
              <li {% if not category %} class="active"{% endif %}><a href="{% url 'shop:product_list' %}">All</a></li>
            {% for c in categories %}
 
              <li {% if category.slug == c.slug %} class="active"{% endif %}>
                  <a href="{{ c.get_absolute_url }}">{{ c.name }}</a>
              </li>
            {% endfor %}
            </ol>
          </div>
        </div><!-- /.blog-sidebar -->
 
      </div><!-- /.row -->
 
    </div><!-- /.container -->
{% endblock %}
```
## Let’s understand the code:
```
      Line 1 – We make sure list.html extends base.html.
      Line 3 to 5 – We have our block title where we check if products are filtered by category or not.
      Line 14 to 30 – We have created a bootstrap row.
      Line 15 to 29 – We have created a loop that loops though our products. In this loop we will display 3 products per row.
      Line 37 to 47 – We have created a side bar where list products by categories. User can filter products by selecting a given category.
On line 19, we are checking whether the products has image or not. 
Since product image can be blank, we need to provide a default image. 
In our static folder, create a folder called img and place an image of you choice there. 
In my case, I have an image called default, this is how you folder should look like.
 ```                                    


## 4.7.settings.py (Add these lines end of the file)
```
MEDIA_URL = '/media/' (new  we define base url that serves our products images)
MEDIA_ROOT = os.path.join(BASE_DIR, 'products/') ( new his is where the uploaded image is located and this path is build dynamically.)
```
## 4.8. main project url.py file settings, make sure it has the following code:
```
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static
 
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('shop.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
```
On line 18 we import settings and on line 19 we import static. On line 24, we serve our static file by adding our MEDIA_URL and MEDIA_ROOT settings.
NB: This is only done in development environment.
```

## 4.9.shop/navbar.html

```
{% load static %}
<nav class="navbar navbar-default navbar-fixed-top" role="navigation" style="margin-bottom: 0px; background-color: #5AC8FA;">
  <!-- Brand and toggle get grouped for better mobile display -->
  <div class="container-fluid">
      <div class="navbar-header">
    <button type="button" class="navbar-toggle" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1">
      <span class="sr-only">Toggle navigation</span>
      <span class="icon-bar"></span>
      <span class="icon-bar"></span>
      <span class="icon-bar"></span>
    </button>
    <a class="navbar-brand" href="{% url 'shop:product_list' %}" style="color: #ffffff">Henrylab Network</a>
  </div>
 
  <!-- Collect the nav links, forms, and other content for toggling -->
  <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
 
    <ul class="nav navbar-nav navbar-right">
      <li class="dropdown">
          {% if request.user.is_authenticated %}
        <a href="#" class="dropdown-toggle" data-toggle="dropdown" style="color: #ffffff">{{ request.user.username }}<b class="caret"></b></a>
        <ul class="dropdown-menu">
          <li><a href="">logout</a></li>
            {% else %}
          <li class="divider"></li>
          <li><a href="#" style="color: #ffffff">Sign in</a></li>
         <li><a href="#" style="color: #ffffff">Sign up</a></li>
        </ul>
         {% endif %}
      </li>
    </ul>
  </div><!-- /.navbar-collapse -->
  </div>
</nav>
```
## 4.10. Runserver
```
	python manage.py runserver
```

# Lesson 5: Completing product detail template for Our Shop products


## detail.html:
```
{% extends 'shop/base.html' %}
{% load static %}
{% block title %}
    {% if category %}{{ category.name }} {% else %} Products {% endif %}
{% endblock %}
 
{% block content %}
   <div class="container">
      <div class="row" style="margin-top: 6%">
 
        <div class="col-sm-8 blog-main">
 
          <div class="blog-post">
              <div class="row">
                <div class="card">
                    <div class="card-body">
                        <div class="col-md-6 text-right">
                            <img src="{% if product.image %} {{ product.image.url }} {% else %} {% static 'img/default.jpg' %} {% endif %}" alt="..." style="height: 170px; width: auto">
                        </div>
                        <div class="col-md-6" style="padding-left: 20px">
                            <h3>{{ product.name }}</h3>
                            <h6><a href="{{ product.category.get_absolute_url }}">{{ product.category }}</a></h6>
                            <p class="text-muted">Kshs. {{ product.price }}</p>
                            <p>{{ product.description|safe|linebreaksbr }}</p>
                        </div>
                    </div>
                </div>
              </div>
 
 
          </div><!-- /.blog-post -->
 
        </div><!-- /.blog-main -->
      </div><!-- /.row -->
 
    </div><!-- /.container -->
{% endblock %}
```

## Lets understand the code in this file:
```
Line 1 – We make sure that detail.html extends our base.html file.
Line 2 – We load static files for this template if any.
Line 3 to 5 – We display our product title in the block title.
Line 7 to 37 – We create a block content where our single product detail will be displayed.
Line 18 – We display the image of the product.
Line 21 – We display the name of the product.
Line 22 – We call get_absolute_url method on related category object to display the available product that belong in the same category.
Line 23 – We display the price of the product.
Line 24 – We display the product description.
```
```
python manage.py runserver 
```
# Lesson 6: Developing shopping cart class for Our Shop products
```
In this lesson we will use django session, learn about django session settings and store our cart in django session. Django session framework support anonymous and user session, the session framework allows developer to store arbitrary data for each user who visit your site and session data is stored on the server side. 
```

## 6.1.Create app name (cart):
```
	django-admin startapp cart
```
## 6.2.Edit settings.py:
```
INSTALLED_APPS = [
    
    'cart.apps.CartConfig',
    …….
    …….
    ……
]

CART_SESSION_ID = 'cart'
```
## 6.3.Create cart.py inside the cart app:

## Cart/Cart.py
```
from decimal import Decimal
from django.conf import settings
from shop.models import Product
 
 
class Cart(object):
    def __init__(self, request):
        self.session = request.session
        cart = self.session.get(settings.CART_SESSION_ID)
        if not cart:
            cart = self.session[settings.CART_SESSION_ID] = {}
        self.cart = cart
 
    def add(self, product, quantity=1, update_quantity=False):
        product_id = str(product.id)
        if product_id not in self.cart:
            self.cart[product_id] = {'quantity': 0, 'price': str(product.price)}
        if update_quantity:
            self.cart[product_id]['quantity'] = quantity
        else:
            self.cart[product_id]['quantity'] += quantity
        self.save()
 
    def save(self):
        self.session[settings.CART_SESSION_ID] = self.cart
        self.session.modified = True
 
    def remove(self, product):
        product_id = str(product.id)
        if product_id in self.cart:
            del self.cart[product_id]
            self.save()
 
    def __iter__(self):
        product_ids = self.cart.keys()
        products = Product.objects.filter(id__in=product_ids)
        for product in products:
            self.cart[str(product.id)]['product'] = product
 
        for item in self.cart.values():
            item['price'] = Decimal(item['price'])
            item['total_price'] = item['price'] * item['quantity']
            yield item
 
    def __len__(self):
        return sum(item['quantity'] for item in self.cart.values())
 
    def get_total_price(self):
        return sum(Decimal(item['price']) * item['quantity'] for item in self.cart.values())
 
    def clear(self):
        del self.session[settings.CART_SESSION_ID]
        self.session.modified = True
```
## Let’s understand the code in this file:
```
Line 1 – We import decimal data type in order to avoid issue of rounding off with regard to price.
Line 2 – We import our settings configuration located in the settings.py file.
Line 3 – We import our product model from our shop application
Line 6 – We create a cart class that will help us manage our shopping cart.
Line 7 – We require the cart to be initialized with request object.
Line 8 – We store the current session using self.session = request.session and this help in making sure that the cart is available for other method in our Cart class.
Line 9 – We try to get the cart using get method in the current session.
Line 10 to 12 – If there is not cart in the session, we set an empty cart on line 11 by settings an empty dictionary in the session.
Line 14 to 22 – We create a method to add product into our cart. The add method takes ( self, product, quantity=1, update_quantity=False) as it’s parameters.
Line 15 – We use product_id as the key in the cart content dictionary, we convert product id into a string because Django uses json to serialize session data and json only allows string names.
Line 17 – The product id is the key and the value we persist is the a dictionary with quantity and price of the product. We also convert the price of the product to string in order to serialize it.
Line 22 – We save the cart in the session.
Line 24 to 26 – We create save method which tracks changes in the cart and marks sessions as modified usingself.session.modified = True
Line 28 to 32 – We create a method to remove a single product from the cart and save the cart in the session.
Line 34 to 43 – We define an __iter__ (self) method which help us iterate through the items in contained in our cart and get the related product instances.
Line 45 to 46 – We define a len method to return the total number of items store in our cart.
Line 48 to 49 – We define get_total_price method to get the total cost of the items in the cart.
Line 51 to 53 – We define a method to clear the cart session.
```
# Lesson 7: Creating Shopping Cart Views for Our Shop products
```
Now that we have a cart class to manage the cart, we need to create views to add, update or remove items from it. In order to add items to the cart we need a form that allow a user to select quantity. Inside our cart application, create a file called forms.py
```

# 7.1. Create forms.py
```
from django import forms

PRODUCT_QUANTITY_CHOICES = [(i, str(i)) for i in range(1, 26)]

class CartAddProductForm(forms.Form):
    quantity = forms.TypedChoiceField(choices=PRODUCT_QUANTITY_CHOICES, coerce=int)
    update = forms.BooleanField(required=False, initial=False, widget=forms.HiddenInput)
```

## Let’s understand this code:
```
Line 1 – we import django forms.
Line 4 – we create a range that start from 1 to 26. We will use this as drop down for user to select number of items.
Line 7 – we create a class that we will use to add items to the cart.
Line 8 – we define a quantity field in the form and we pass our choices from line 4.
Line 9 – we define an update field that will help in either adding or updating number of item to the cart.
```

## 7.2. cart/views.py
```
from django.shortcuts import render, redirect, get_object_or_404
from django.views.decorators.http import require_POST
from shop.models import Product
from .cart import Cart
from .forms import CartAddProductForm
 
 
@require_POST
def cart_add(request, product_id):
    cart = Cart(request)
    product = get_object_or_404(Product, id=product_id)
    form = CartAddProductForm(request.POST)
    if form.is_valid():
        cd = form.cleaned_data
        cart.add(product=product, quantity=cd['quantity'], update_quantity=cd['update'])
    return redirect('cart:cart_detail')
 
 
def cart_remove(request, product_id):
    cart = Cart(request)
    product = get_object_or_404(Product, id=product_id)
    cart.remove(product)
    return redirect('cart:cart_detail')
 
 
def cart_detail(request):
    cart = Cart(request)
    for item in cart:
        item['update_quantity_form'] = CartAddProductForm(initial={'quantity': item['quantity'], 'update': True})
    return render(request, 'cart/detail.html', {'cart': cart})
```
## Let’s understand the code in this file:
```
	Line 1 – we import django shortcuts to helps render, redirect and query our database.
	Line 2 – we import django decorator to post our form.
	Line 3 – we import our Product model from our shop.
	Line 4 – we import our Cart class from cart.py file
	Line 5 – we import our form from forms.py file.
	Line 8 – We use @require_POST decorator to make sure that only post request are allowed.
	Line 9 – we define a method to add items to the Cart. This method takes product id as a parameters.
	Line 10 – we create a cart object by passing request to our cart class.
	Line 11 – we retrieve our product from database based on the product id.
	Line 12 – we validated our form from the request.
	Line 13 – 15 if our form is valid we add our item to the cart and update our cart.
	Line 16 – we redirect our cart to cart detail page.
	Line 19 – we define a method to remove items from the cart. This method takes product id as a parameter.
	Line 20 – we create a cart object from our request.
	Line 21 – we retrieve product by id.
	Line 22 – we remove item from our shopping cart.
	Line 23 – we redirect user to our cart detail page.
	Line 26 – we define a method to display our cart detail page.
	Line 27 – we create a cart object from our request.
	Line 28 to 29 – we create a for loop to iterate through all products in the cart.
	Line 30 – we render a template called detail.html
```

# 7.3. Create urls.py inside cart app

```
from django.conf.urls import url
from . import views
 
app_name = 'cart'
 
urlpatterns = [
    url(r'^$', views.cart_detail, name='cart_detail'),
    url(r'^add/(?P<product_id>\d+)/$', views.cart_add, name='cart_add'),
    url(r'^remove/(?P<product_id>\d+)/$', views.cart_remove, name='cart_remove'),
]
```


## Let’s understand the code in this file:
```
	Line 1 – We import django url helper class.
	Line 2 – we import views from cart views.py file.
	Line 4 – we create a namespace for our cart application called ‘cart’
	Line 6 to 10 – we create a list of url patterns.
	Line 7 – we define a url that will display cart detail page.
	Line 8 – we define url to add items to the cart.
	Line 9 – we define url to remove items from our cart.
```
# 7.4. ecommerce/urls.py
```
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static
 
urlpatterns = [
    path('admin/', admin.site.urls),
    path('cart', include('cart.urls')),
    path('', include('shop.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
```
	We need to create a template to display our cart detail page. 
	Inside our cart application, create a folder called templates. 
	Inside templates folder create another folder called cart.
	Inside cart folder, create a file called detail.html.
```
# 7.5. cart/detail.html
```
(onlineshop) ➜  cart touch urls.py
(onlineshop) ➜  cart mkdir templates
(onlineshop) ➜  cart mkdir templates/cart
(onlineshop) ➜  cart touch templates/cart/detail.html
```
```
{% extends 'shop/base.html' %}
{% load static %}
{% block title %}
    Your Shopping Cart
{% endblock %}
 
 
{% block content %}
    <div class="container">
        <div class="row" style="margin-top: 6%">
        <h2>Your Shopping Cart
            <span class="badge pull-right">
                {% with totail_items=cart|length %}
                    {% if cart|length > 0 %}
                        My Shopping Order:
                        <a href="{% url "cart:cart_detail" %}" style="color: #ffffff">
                            {{ totail_items }} item {{ totail_items|pluralize }}, Kshs. {{ cart.get_total_price }}
                        </a>
                        {% else %}
                        Your cart is empty.
                    {% endif %}
                {% endwith %}
            </span>
        </h2>
            <table class="table table-striped table-hover">
                <thead style="background-color: #5AC8FA">
                    <tr>
                        <th>Image</th>
                        <th>Product</th>
                        <th>Quantity</th>
                        <th>Remove</th>
                        <th>Unit Price</th>
                        <th>Price</th>
                    </tr>
                </thead>
                <tbody>
                {% for item in cart %}
                    {% with product=item.product  %}
                        <tr>
                            <td>
                                <a href="{{ product.get__absolute_url }}">
                                    <img src="{% if product.image %} {{ product.image.url }} {% else %} {% static 'img/default.jpg' %} {% endif %}" alt="..." style="height: 130px; width: auto">
                                </a>
                            </td>
                            <td>{{ product.name }}</td>
                            <td>
                                <form action="{% url "cart:cart_add" product.id %}" method="post">
                                    {% csrf_token %}
                                    {{ item.update_quantity_form.quantity }}
                                    {{ item.update_quantity_form.update }}
                                    <input type="submit" value="Update" class="btn btn-info">
                                </form>
                            </td>
                            <td>
                                <a href="{% url "cart:cart_remove" product.id %}">Remove</a>
                            </td>
                            <td>kshs. {{ item.price }}</td>
                            <td>kshs. {{ item.total_price }}</td>
                        </tr>
                    {% endwith %}
                {% endfor %}
                <tr style="background-color: #5AC8FA">
                    <td><b>Total</b></td>
                    <td colspan="4"></td>
                    <td colspan="num"><b>kshs. {{ cart.get_total_price }}</b></td>
                </tr>
                </tbody>
            </table>
        <p class="text-right">
            <a href="{% url "shop:product_list" %}" class="btn btn-default">Continue Shopping</a>
            <a href="" class="btn btn-primary">Checkout</a>
        </p>
        </div>
    </div>
{% endblock %}
```
```
	We use this template to display items in our cart.
	From line 25 to 68, we have created a bootstrap table that we use to display items in our cart. 
	From line 37 to 61, we have created a loop that iterate over our items in our cart. 
	From line 47 to 52, we have define a form where we allow user to change quantity of a given product in the cart. 
	On line 55, we provide a link to allow user to remove item from our cart.
```
# 7.6. shop/views.py
```
	The next thing we need to do is to add a button in the product detail.html page in order to allow user to add 		product to our cart. Edit the views.py file of the shop application 
```
```

from django.shortcuts import render, get_object_or_404
from .models import Category, Product
from cart.forms import CartAddProductForm
 
 
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
    cart_product_form = CartAddProductForm()
    context = {
        'product': product,
        'cart_product_form': cart_product_form
    }
    return render(request, 'shop/product/detail.html', context)


	On line 3, we import CartAddProductForm package from our cart forms. 
	On line 24, we have created our cart_product_form object. 
	On line 27, we pass our form to render function. 
```

# 7.7. shop/detail.html
```

{% extends 'shop/base.html' %}
{% load static %}
{% block title %}
    {% if category %}{{ category.name }} {% else %} Products {% endif %}
{% endblock %}
 
{% block content %}
   <div class="container">
      <div class="row" style="margin-top: 6%">
 
        <div class="col-sm-8 blog-main">
 
          <div class="blog-post">
              <div class="row">
                <div class="card">
                    <div class="card-body">
                        <div class="col-md-6 text-right">
                            <img src="{% if product.image %} {{ product.image.url }} {% else %} {% static 'img/default.jpg' %} {% endif %}" alt="..." style="height: 170px; width: auto">
                        </div>
                        <div class="col-md-6" style="padding-left: 20px">
                            <h3>{{ product.name }}</h3>
                            <h6><a href="{{ product.category.get_absolute_url }}">{{ product.category }}</a></h6>
                            <p class="text-muted">Kshs. {{ product.price }}</p>
                            <form action="{% url "cart:cart_add" product.id %}" method="post">
                                {% csrf_token %}
                                {{ cart_product_form }}
                                <input type="submit" value="add to cart" class="btn btn-primary">
                            </form>
                            <p>{{ product.description|safe|linebreaksbr }}</p>
                        </div>
                    </div>
                </div>
              </div>
 
 
          </div><!-- /.blog-post -->
 
        </div><!-- /.blog-main -->
      </div><!-- /.row -->
 
    </div><!-- /.container -->
{% endblock %}
```
```
From line 24 to 28, we have added a form to allow users to add products to the cart.
```

