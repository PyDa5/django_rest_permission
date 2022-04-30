## 功能

* 一个可独立加载的APP，能动态配置用户对视图的访问权限

* 能像python manage.py migrate一样，将视图权限自动迁移至数据库


## 安装(Install)

> pip install django-rest-permission

## 使用(How to use)
####  项目结构如下
```
myproject
    - settings.py
    - urls.py
    - ...
myapp
    - app.py
    - models.py
    - permissions.py(*)
    - urls.py
    - views.py
    - ...
```

#### 1、配置settings.py
```python
INSTALLED_APPS = [
    '...',
    'django_rest_permission',
    '...'
]
```

####  2、在APP的permissions.py导入getGenericViewPermission，获取GenericViewPermission

```python
from django_rest_permission.permissions import getGenericViewPermission

GenericAPIViewPermission = getGenericViewPermission()
...
```

#### 3、在views.py导入GenericViewPermission，作为permission_classes，
同时声明两个类属性：view_group、view_access_permissions
```python
from rest_framework.views import APIView
from django.views.generic.base import View
from myapp.permissions import GenericViewPermission


class MyAPIView01(APIView):
    # 视图权限分组，对应django_content_type表中的model字段，model = 购物车
    view_group = '购物车'
    # 用于生成auth_permissions中的信息
    view_access_permissions = {
        'GET': '查询购物车商品',  # name = 查询购物车商品，codename = view://myapp/购物车/查询购物车商品
        'POST': '创建购物车',  # name = 创建购物车， codename = view://myapp/购物车/创建购物车
        'PUT': '修改购物车商品',  # name = 修改购物车商品, codename = view://myapp/购物车/修改购物车商品
        'DELETE': '清空购物车'  # name = 清空购物车， codename = view://myapp/购物/清空购物车
    }
    # 引入权限校验类
    permission_classes = [GenericViewPermission]
    
    def get(self, request):
        # get请求且用户有“查询购物车商品”权限时走这里
        pass
    
    def post(self, request):
        # post请求且用户有“创建购物车”权限时走这里
        pass
    
    ...


class MyView02(View):
    # 视图权限分组，对应django_content_type表中的model字段，model = 用户管理
    view_group = '用户管理'
    # 用于生成auth_permissions中的信息
    view_access_permissions = {
        'GET': ('view_user_info', '查询用户信息'),  # name = 查询用户信息, codename = view://myapp/用户管理/view_user_info
        'POST': ('create_user', '新建用户'),# name = 新建用户, codename = view://myapp/用户管理/create_user
        'PUT': ('modify_user_profile', '修改用户资料'),  # name = 修改用户资料, codename = view://myapp/用户管理/modify_user_profile
        'DELETE': ('delete_user', '删除用户')  # name = 删除用户, codename = view://myapp/用户管理/delete_user
    }
    permission_classes = [GenericViewPermission]
    
    def get(self, request):
        # get请求且用户有“查询用户信息”权限时走这里
        pass
    
    def post(self, request):
        # post请求且用户有“新建用户”权限时走这里
        pass
    ...
```

#### 4、权限入库：自动收集已加载的APP中APIView视图类中声明的权限

> 方式1：python manage.py collectpermissions

> 方式2：python manage.py collectpermissions app_name

执行以上命令后，数据库的django_content_type、auth_permission会生成视图对应的权限信息

#### 5、数据库结构 —— django_content_type

| id  | app_label | model |
|-----|-----------|-------|
| 666 | drp_myapp | 购物车   |
| 777 | drp_myapp | 用户管理  |

#### 6、数据库结构 —— auth_permission

| id| name| content_type_id | codename                 |
|---|-----|-----------------|--------------------------|
| ... | 查询购物车商品 | 666 | view://myapp/购物车/查询购物车商品 |
| ... | 创建购物车 | 666 | view://myapp/购物车/创建购物车   |
| ... | 修改购物车商品 | 666 | view://myapp/购物车/修改购物车商品 |
| ... | 清空购物车 | 666 | view://myapp/购物车/清空购物车   |
| ... | 查询用户信息 | 777 | view://myapp/用户管理/查询用户信息 |
| ... | 新建用户 | 777 | view://myapp/用户管理/新建用户   |
| ... | 修改用户资料 | 777 | view://myapp/用户管理/修改用户资料 |
| ... | 删除用户 | 777 | view://myapp/用户管理/删除用户   |

#### 7、管理后台创建用户、用户组，并分配权限 

例如当为用户分配“drf_myapp.view://myapp/购物车/清空购物车”权限时，
用户就可以通过delete请求方法，访问myapp下MyAPIView01视图中的delete方法。
