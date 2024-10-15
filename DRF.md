# Django JsonResponse vs DRF Response

```
from todo.models import Todo
from django.http import HttpRequest, JsonResponse

# this FBV returns a simple json view on webpage
def json_response(request: HttpRequest):
    # To get queryset you should always loop throw it or make list of queryset to run it. 
    todos = list(Todo.objects.order_by('priority').all().values('title','is_done'))
    return JsonResponse({'todos': todos})
```

```
from todo.models import Todo
from rest_framework.request import Request
from rest_framework.response import Response
from rest_framework.decorators import api_view
from rest_framework import status

@api_view(['GET'])
def json_response(request: Request):
    todos = list(Todo.objects.order_by('priority').all().values('title','is_done'))
    return Response({'todos': todos}, status.HTTP_200_OK)
```
# HTTP methods on DRF
1. GET
> Get one or all data
2. POST
> send data to server and create a new instance for model
3. PUT
> send data to server and replace the entire resource a new representation, meaning that all the fields of the resource are sent in the request body, even if they are not modified.
4. PATCH
>  it is used to apply partial updates to a resource, meaning that only the fields that need to be changed are sent in the request body.
5. DELETE
> delete a instance of model

# Serializers

> Serializers are used to convert complex data types, such as Django model instances, into Python data types that can be easily rendered into JSON, XML, or other content types. Serializers also provide deserialization, allowing parsed data to be converted back into complex types after first validating the incoming data. Serializers get data from models and pass it to views

serializer types:
1. serializer
2. modelserializers

=> make a serializer.py

```
from rest_framework import serializers
from .models import ModelName


class ModelNameSerializer(serializers.ModelSerializer):
    class Meta:
        model = ModelName
        fields = ['field_name', 'field_name', 'field_name']
        
```

# Function Base View(FBV)

> HINT: data => take data(Json, Xml or ...) and deserialize it(model data).\
> instance => take model data and serialize it(to Json, Xml or ...)

```
from .models import ModelName
from .serializers import ModelNameSerializer
from rest_framework.response import Response
from rest_framework.request import Request
from rest_framework import status
from rest_framework.decorators import api_view

@api_view(['GET', 'PUT', 'DELETE'])
def todo_detail_update_delete(request: Request, pk):
    try:
        queryset = ModelName.objects.get(id=pk)
    except ModelName.DoesNotExist:
        return Response(None, status.HTTP_404_NOT_FOUND)
    
    if request.method == 'GET':
        serializer = ModelNameSerializer(queryset)
        return Response(serializer.data, status.HTTP_200_OK)
    
    if request.method == 'PUT':
        # first deserialize data and then asign it too queryset(instance)
        serializer = ModelNameSerializer(instance= queryset, data= request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status.HTTP_202_ACCEPTED)
        
    if request.method == 'DELETE':
        queryset.delete()
        return Response(status.HTTP_202_ACCEPTED)

```
# Class Base View(CBV)
1. ApiView
> No if statement. \
> You can use all five methods by methods. \
> It uses 'queryset' and 'serializer' for each method.

```
from .models import ModelName
from .serializers import ModelNameSerializer
from rest_framework.response import Response
from rest_framework.request import Request
from rest_framework import status
from rest_framework.views import APIView

class ModelNameApiView(APIView):
    def get_object(self, pk: int):  
        try:
            queryset = ModelName.objects.get(id=pk)
            return queryset
        except ModelName.DoesNotExist:
            return Response(None, status.HTTP_404_NOT_FOUND)
        
    def get(self, request: Request, pk: int):
        queryset = self.get_object(pk)
        serializer = ModelNameSerializer(queryset, many=True)
        return Response(serializer.data, status.HTTP_200_OK)
    
    def put(self, request: Request, pk: int):
        queryset = self.get_object(pk)
        serializer = ModelNameSerializer(instance= queryset, data= request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status.HTTP_202_ACCEPTED)
        
    def delete(self, request, pk: int):
        queryset = self.get_object(pk)
        queryset.delete()
        return Response(status.HTTP_202_ACCEPTED) 
```

2. Mixins & Generics
> To create a view, mixin and generic both must be used. \
> Write 'queryset' and 'serializer' once, and they will used in all methods.

**mixins**: 
1. CreateModelMixin 
2. DestroyModelMixin
3. ListModelMixin
4. RetrieveModelMixin
5. UpdateModelMixin 

```
from .models import ModelName
from .serializers import ModelNameSerializer
from rest_framework.response import Response
from rest_framework.request import Request
from rest_framework import status
from rest_framework import generics, mixins

class ModelNameMixinView(mixins.RetrieveModelMixin, mixins.UpdateModelMixin, mixins.DestroyModelMixin, generics.GenericAPIView):
    queryset = ModelName.objects.order_by('id').all()
    serializer_class = ModelNameSerializer
    
    def get(self, request: Request, pk):
        return self.retrieve(request, pk)
    
    def put(self, request: Request, pk):
        return self.update(request, pk)
    
    def delete(self, request: Request, pk):
        return self.destroy(request, pk)
```

3. GenericViews
> No need to use Mixins anymore

**generics**:
1. CreateAPIView
2. DestroyAPIView
3. ListAPIView
4. RetrieveAPIView
5. UpdateAPIView
6. ListCreateAPIView 
7. RetrieveUpdateAPIView
8. RetrieveDestroyAPIView
9. RetrieveUpdateDestroyAPIView

```
from .models import ModelName
from .serializers import ModelNameSerializer
from rest_framework import generics

class ModelNameGenericView(generics.ListCreateAPIView):
    queryset = ModelName.objects.order_by('id').all()
    serializer_class = ModelNameSerializer
```
4. ViewSets
> need Router to create a url. 


**views.py**
```
from rest_framework import viewsets

class ModelNameViewSet(viewsets.ModelViewSet):
    queryset = ModelName.objects.order_by('id').all()
    serializer_class = ModelNameSerializer
```

**urls.py**
```
from django.urls import path, include
from .views import ModelNameViewSet
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register('', ModelNameViewSet)

urlpatterns = [
    path(prefix='viewset/', include(router.urls)) 
]
```



# Pagination
> This allows you to modify how large result sets are split into individual pages of data.

1. Global Pagination:
    1. LimitOffsetPagination

        => url combination is : /?limit=2&offset=2 \
        limit: elements per pages \
        ofsset: how many must be passed(not showed)

        settings.py
        ```
        REST_FRAMEWORK = {
            'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
            'PAGE_SIZE': 2
        }
        ```

    2. PageNumberPagination

        => url combination is : /?page=3 \
        page: which pages you are in

        settings.py

        ```
        REST_FRAMEWORK = {
            'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
            'PAGE_SIZE': 2
        }
        ```

2. Local Pagination:

    `` from rest_framework.pagination import LimitOffsetPagination, PageNumberPagination ``

    1. PageNumberPagination 

        views.py

        ```
        class TodoPageizeGenericView(PageNumberPagination):
            page_size = 3

        class TodoGenericView(generics.ListCreateAPIView):
            queryset = Todo.objects.order_by('id').all()
            serializer_class = TodoSerializer
            pagination_class = TodoPageizeGenericView
        ```
        
    2. LimitOffsetPagination

        views.py
        ```
        class TodoPageizeViewSet(LimitOffsetPagination):
            default_limit = 4
            
        class TodoViewSet(viewsets.ModelViewSet):
            queryset = Todo.objects.order_by('id').all()
            serializer_class = TodoSerializer
            pagination_class = TodoPageizeViewSet
        ```

# Authentication & Permission
**Authentication**
1. BasicAuthentication
2. TokenAuthentication
> User sed POST request to server in to get a token.\
> To send POST request must use API Handlers like POSTMAN.


urls.py
```
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    path('auth_token/', obtain_auth_token, name='auth_token')
]
```
**POSTMAN**

=> Set request type to POST. \
=> Enter value of: 
```
{
"username": "username",
"password": "password"
}
```
in body of request. Then you get a Token.

=> Go to HEADERS and set KEY = Authorization and VALUE = Token token value

3. JWT(Json Web Token)

    [JWT Doc](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/getting_started.html)

**Permission**
1. IsAuthenticated

**Local & Global**
1. Global Authentication & Permission:

    settings.py
    ```
    REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': ['rest_framework.permissions.IsAuthenticated'],
    'DEFAULT_AUTHENTICATION_CLASSES': ['rest_framework.authentication.BasicAuthentication',
                                        'rest_framework.authentication.TokenAuthentication',
                                        'rest_framework_simplejwt.authentication.JWTAuthentication']
    }  
    ```
2. Local Authentication & Permission:

    views.py
    ```
    from rest_framework.authentication import BasicAuthentication
    from rest_framework.permissions import IsAuthenticated

    class ModelNameApiVIew(APIView):
    permission_classes = [IsAuthenticated]
    authentication_classes = [BasicAuthentication, TokenAuthentication]
    ```

# Validation

serializers.py 
```
from rest_framework import serializers
from .models import ModelName

class ModelNameSerializer(serializers.ModelSerializer):
    class Meta:
        model = ModelName
        fields = '__all__'
    def validate_fieldname(self, fieldname):
        pass
```