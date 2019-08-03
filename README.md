# django-pagenator
总结了3种分页方式：分别是常规分页、model分页、rest_framework API分页


## 第一种  常规分页
感受：不推荐使用,小数据量还行，但是对几十万数据分页时会很慢

```
#########
urls.py
#########
# -*- coding: utf-8 -*-

from django.conf.urls import url
from . import apis

urlpatterns = [
    url(r'table/', apis.show_table_data, name="table"),
]

#########
views.py
#########
# -*- coding: utf-8 -*-

from rest_framework.decorators import api_view
from rest_framework.response import Response
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from app.models import SomeModel
from app.serializers import DataSerializer

@api_view(['GET'])
def show_table_data(request):
    """
           request.GET示例: <QueryDict: {'order': ['asc'], 'offset': ['0'], 'limit': ['9']}>
           limit_num_per_page: 每页显示数据的条数
           query_start_num: 获取从第query_start_num条开始的数据
    """

    context = {}
    table_data_to_show = []
    limit_num_per_page = int(request.GET.get("limit", 8))
    query_start_num = int(request.GET.get("offset", 0))
    model_data = SomeModel.objects.all().order_by('-id')
    paginator = Paginator(model_data, limit_num_per_page)
    current_page = int(query_start_num / limit_num_per_page + 1) 
    try:
        table_data_paginator = paginator.page(current_page)
    except PageNotAnInteger:
        table_data_paginator = paginator.page(1)
    except EmptyPage:
        table_data_paginator = paginator.page(paginator.num_pages)
    for query_set in table_data_paginator:
        serialize_data = DataSerializer(query_set).data
        table_data_to_show.append(serialize_data)

    context['rows'] = table_data_to_show
    context['total'] = len(interface_data)
    return Response(context)
```

## 第三种  rest_framework API分页
感受：代码清晰，对几十万数据分页也很快

```
#########
urls.py
#########
# -*- coding: utf-8 -*-

from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'table/', views.DataViewSet.as_view(), name="table"),
]

#########
views.py
#########
from rest_framework.views import APIView
from rest_framework.pagination import PageNumberPagination,LimitOffsetPagination
from rest_framework.response import Response
from app.models import DataModel
from app.serializers import DataSerializer


class StandardResultsSetPagination(LimitOffsetPagination):
    # 默认每页显示的数据条数
    default_limit = 8
    # URL中传入的显示数据条数的参数
    limit_query_param = 'limit'
    # URL中传入的数据位置的参数
    offset_query_param = 'offset'
    # 最大每页显示条数
    max_limit = None


class DataViewSet(APIView):
    def get(self, request, *args, **kwargs):
        model_data = DataModel.objects.all().order_by('-id')

        # 实例化分页对象，获取数据库中的分页数据
        paginator = StandardResultsSetPagination()
        page_inter_list = paginator.paginate_queryset(model_data, self.request, view=self)

        # 序列化对象
        serializer = DispatcherRequestSerializer(page_inter_list, many=True)

        # 生成分页和数据
        response = paginator.get_paginated_response(serializer.data)
        return response

```

# 示例中引用的model和serialazer代码

```
######
model.py
######
from django.db import models
# Create your models here.

class DataModel(models.Model):
   
    name = models.CharField(primary_key=True, max_length=255)
    desc = models.CharField(max_length=255)
    
    class Meta:
        ordering = ('name',)
        verbose_name = "beautiful_name"
        verbose_name_plural = "beautiful_name"

    def __str__(self):
        return str(self.name)
        
        
######
serialazer.py
######

from rest_framework import serializers
from .models import DataModel


class DataSerializer(serializers.ModelSerializer):
    name =  serializers.CharField(source="rand_val")
    desc =  serializers.CharField(source="intf_ver")
    
    class Meta:
        model = DataModel
        fields = "__all__"
```










































