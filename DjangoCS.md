# **DJANGO**

# **CREATING VE** 
`py -m venv v-name`

# start a project
`django-admin startproject project-name`

# Django architecture
* it's MVT
    * Model
    * View
    * Template

# Template 
fillup with html and django template language. it show's a html page. \
django automatically look for templates folder.

> app -> templates -> app -> sth.html


to show template files they have to rendered in views.py 

` render(http request, 'path to html file', context = {'key' : value}) `

## django template language:
    1. variable => {{variable}}
    2. tag => {% for / if  / ... %}
    3. filter => {{variable | filter condititon}}

## template inheritance:
* make a skelton for your webpage and use it in all webpages.
    1. make base html file to inherit from and use block( to override later)
        + `{% block name %}  {% endblock %}`
    2. use {% extends 'base file path' %}
    3. override block/blocks

_base.html :

{% url 'about' %} : define 'name=' for path in urls.py then after changer url patten no need to change template. 
```
<html>
    <body>
        <div>
            <a href = {% url 'about' %}> about </a>
            <a href = {% url 'home' %}> home </a>
        </div>
        
        {% block content %}  {% endblock %}
    </body>
</html>
```
inherited.html :
```
{% extends "movies/_base.html" %}

<html>
    <body>
        {% block content %}
            <h1>
                my fav movies are
            </h1>
            {% if  movies%}
                {% for movie in movies %} 
                    <li>
                        {{movie | title}}
                    </li>
                {% endfor %}
                {% else %}
                    <h4>
                        no fav movies was found
                    </h4>
            {% endif %}
        {% endblock %}
        </body>
</html>
```
# static files
* Static files (CSS, JavaScript, Images)
> app -> static -> app -> image or CSS or JS file

to show static file:

-> sth.html
```
{% load static %}
    <image style='width:500px;' src = {% static 'movies/image.jpg'%} alt = 'shitty picture'>

```

# ***CSRF TOKEN***
* Cross-Site Request Forgery
* When a user logs in or starts a session, Django generates a random and unique CSRF token for that session.
* CSRF token is usually a long string of characters.
* CSRF token is associated with the user’s session and stored on the server.

**Token Validation on Submission**

When the user submits the form, the CSRF token is sent along with the request, either as a POST parameter or a request header (e.g., X-CSRFToken). The token is extracted from the request by the server. It is then verified that if this token (received in request) matches with the token which is linked with the user’s session. If the token matches, the request is considered as valid and we can proceed with it. If they don’t match, then the server interprets it as it may be a CSRF attack and rejects the request.

-> sth.html

` {% csrf_token %} `


# View
* interact with db by using 'model manager'
* request handler
* load template files
* CBV(class base view) and FBV(function base view)
```
def index(request):
    context = {
        'movies': ['black swan', 'requiem for a dream']
    }
    return render(request, 'movies/movie.html', context)
```
## **get_object_or_404()**
> Use get() to return an object, or raise an Http404 exception if the object does not exist.
```
from django.shortcuts import get_object_or_404

def get_object_or_404(
    klass: type[_T@get_object_or_404] | Manager[_T@get_object_or_404] | QuerySet[_T@get_object_or_404],
    *args: Any,
    **kwargs: Any
) -> _T@get_object_or_404
```
=> klass may be a Model, Manager, or QuerySet object.

=> MultipleObjectsReturned is raised if more than one object is found.

## **Class Base View**
* **ListView**: automatically search for model_list.html to render html page. it sends GET request

```
from django.views.generic import ListView
from .models import ModelName

class ClassNameListView(ListView):
    model = ModelName
```

to set the url for this class you have to use ".as_view()" to make a response process 

-> urls.py

```
fro django,urls import path

urlpattern = [
    path('sth', ClassNameListView.as_view())
]
```

* **CreateView**: automatically search for model_form.html to render html page. it sends POST request

```
class ListCreatView(CreateView):
    model = ModelName
    # models fields
    fields = ['field_name']
    # redirect user to the defined page, after success process
    success_url = reverse_lazy('links')
```

* **UpdateView**: it uses same template(html page) as CreateView uses. it sends PATCH request

```
class ListUpdateView(UpdateView):
    model = ModelName
    # models fields
    fields = ['field_name']
    # redirect user to the defined page, after success process
    success_url = reverse_lazy('links')
```

* **DeleteView**: it uses model_confirm_delete.html. it is DELETE request boy in html file we set method="POST"

```
class ListDeleteView(DeleteView):
    model = ModelName
    # redirect user to the defined page, after success process
    success_url = reverse_lazy('links')
```

* **TemplateView**: it only return a template.
```
class HomeTemplateView(TemplateView):
    template_name = 'trip/home.html'
```

* **DetailView**: 
**reverse() vs reverse_lazy()**

* `reverse()` is used in FBV and `reverse_lazy()` is used CBS.
* If we are using `success_url` we have to use `reverse_lazy()`.
* If we are reversing inside a function we can use `reverse()`.
* `reverse()` returns a string & `reverse_lazy()` returns an object.
* the `reverse()` is evaluated at the moment the func_name method is called.
* `reverse_lazy()` is also used to generate a URL, but it defers the moment of evaluation until it is actually needed. 




# cleaned_data()
> it makes a dictionary. sort keys by alphabatic. if data is not validated just store validated data. f your data does not validate, the cleaned_data dictionary contains only the valid fields. if you pass extra data when you define the Form; cleaned_data contains only the form’s fields.When the Form is valid, cleaned_data will include a key and value for all its fields, even if the data didn’t include a value for some optional fields

# Model
* django uses ORM to create database
* classes are tables
* attrs are fields in tables
* every table(class) has id field that automatically made by django, start at number 1 and autoincrements

## **Null = True & Blank = True**
`null=True` sets NULL (versus NOT NULL) on the column in your DB. Blank values for Django field types such as DateTimeField or ForeignKey will be stored as NULL in the DB.
blank determines whether the field will be required in forms. This includes the admin and your custom forms. If `blank=True` then the field will not be required, whereas if it's False the field cannot be blank( this doesn't mean that the field will be stored as an empty string ('') in the database if left blank. Instead, Django will store `NULL` (or equivalent depending on the database) in the database for fields that are left blank).

The combo of the two is so frequent because typically if you're going to allow a field to be blank in your form, you're going to also need your database to allow NULL values for that field. The exception is CharFields and TextFields, which in Django are never saved as NULL. Blank values are stored in the DB as an empty string ('').

A few examples:

`models.DateTimeField(blank=True) # raises IntegrityError if blank`

`models.DateTimeField(null=True) # NULL allowed, but must be filled out in a form`

Obviously, Those two options don't make logical sense to use (though there might be a use case for null=True, blank=False if you want a field to always be required in forms, optional when dealing with an object through something like the shell.)

models.CharField(blank=True) # No problem, blank is stored as ''

models.CharField(null=True) # NULL allowed, but will never be set as NULL

CHAR and TEXT types are never saved as NULL by Django, so null=True is unnecessary. However, you can manually set one of these fields to None to force set it as NULL. If you have a scenario where that might be necessary, you should still include null=True

# **RELATIONS**
> one to one => each record in one model is associated with one and only one record in the other model, and vice versa.

` field_name = models.OneToOneField(ModelName) `

> many to one => a record in one table relates to multiple records in another table. 

` field_name = models.ForeignKey(ModelName) `

> many to many =>  multiple records in one table relate to many records in another table.

` field_name = models.ManyToManyField(ModelName) `

### **on_delete methods**
>  CASCADE: This implies that if you delete the referenced object from the database, all entries for that object will be deleted from the entire database.

` field_name = models.ForeignKey(ModelName, on_delete = models.CASCADE) `

>  PROTECT: when there is an impact on the actual object, all instances of the data on the referenced object are not deleted.

` field_name = models.ForeignKey(ModelName, on_delete = models.PROTECT) `

> RESTRICT: when the deletion is targeted on the referenced object, then the ON_DELETE will raise an error called the RestrictedError. But the RESTRICT will allow deletion to happen when the referencing object and the object which is referenced object is allotted concerning a different common object, then the deletion is permitted.

` field_name = models.ForeignKey(ModelName, on_delete = models.RETRICT) `

> SET_NULL: When a deletion occurs on the referenced object, the referencing object’s value will update to NULL.

` field_name = models.ForeignKey(ModelName, on_delete = models.SET_NULL) `

> SET_DEFAULT: When a deletion occurs on the referenced object, the referencing object’s value will be updated with its allotted default value.

` field_name = models.ForeignKey(ModelName, on_delete = models.SET_DEFAULT) `

# **USER**
* it is prebuilt django model(table).
* it is used for authentication 

to get User model:

```
from django.contrib.auth import get_user_model

# create an instance for User model to use it in model relations.
User = get_user_model()
```


# use create db :
> create instructions and tell django that how db has changed.

` py manage.py makemigrations `

> apply change to db.

` py manage.py migrate `

# CRUD operations :
1. create
2. read
3. update
4. delete

use 'model manager' to do all operations.
* .objects

> retrieve all data from db.

` ClassName.objects.all() `

> retrieve specific row from db

` ClassName.objects.get(id = 1) `

> filtering data with specific value

` ClassName.objects.filter() `

* **How to active django shell :**

` py manage.py shell `

# **humanize**
> A set of Django template filters useful for adding a “human touch” to data.

* add this to the INSTALLED_APPS

` django.contrib.humanize `

* the use this tag and then filter

` {% load humanize%} `

## filters
* apnumber
* intcomma
* intword
* naturalday
* naturaltime
* ordinal

# **DJANGO ADMIN**
* it is graphical interface
* you can creat,update or delete your models.

> you can register models in admin.py

`admin.site.register(ModelName) `

> create admin acount:

` python manage.py createsuperuser `

> change password :

` py manage.py changepassword admin `

## **DJANGO ADMIN INTERFACE**

> change site header(default = Django administration)

`admin.site.header_site = 'new site header' `

> change index title(default = Site administration)

`admin.site.index_title = 'new index title' `

> change site title(default = Django site admin)

`admin.site.site_title = 'new site title' `

> ordering models data with Meta
define Meta as a subclss of your model.
```
class Meta:
    ordering = ['field_name']
```
> customizing list page

```
@admin.register(ModelName)
class ModelNameAdmin(admin.ModelAdmin):
    # the are more options to customize. search 'django modeladmin'.
    # show fields 
    list_display = ['field_name', 'computed_field']
    # editable fields
    list_editable = ['field_name']
    # objects per page
    list_per_page = int
    # you can set ordering from here too(insead of Meta subcalss in models.py -> ModelName)
    ordering = ['field_name']
    # search in field. 
    #to search in terms of 'startwith' use '__lookup'
    search_fields = ['field_name__startwith']

    # active ordeering
    @admin.display(ordering = ['field_name'])
    # enter a computed field to the list page
    def computed_field(self, second_argument):
        ...
```
##  **FORMS IN ADMIN PANEL**
> fields to show in form panel

` fields = ['field_name'] `

> shows all except this/these fields

` exclude = ['field_name'] `

> read only fields 

` readonly_fields = ['fields_name'] `

# **FORMS**

> creating form

-> create forms.py

```
from djagno import forms

class FormName(forms.Form):
    # field attrs fillsup exactily like models

```

better way is use ModelForm

```
from djagno import forms
from .models import ModelName

class FormName(forms.ModelForm):
    class Meta:
        model = ModelName
        fields = ['field_name']
```

# SESSIONS
* it framework to store users arbitrary  data temporarily(per-site-visit)

# AUTHENTICATION

* automaticaly installed in INSTALL_APPS
* it has view and url but doesnt have template

default urls:
1. login/
2. logout/
3. password_reset/
4. password_reset/done/
5. reset/< uidb64 >/< token >/
6. reset/done/

**Login**

=> views.py 

``
from django.shortcuts import render, redirect
from django.contrib.auth import login, authenticate
from django.contrib import messages

def func_name(request):
    context= {}
    
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        
        # if user autheticated, it returns User object otherwise it returns None.
        user = authenticate(request, username= username, password = password)
        
        if user is not None: 
            login(request, user)
            messages.success(request, message= 'You have been loggedin')
            return redirect('home')
        else:
            messages.success(request, message= ' You are not able to login')
            return redirect('home')
    else:
        return render(request, 'home.html', context)
``















