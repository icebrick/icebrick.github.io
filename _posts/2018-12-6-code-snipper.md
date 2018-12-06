---
title: Code Snipper
key: 20181206
tags: code
---

常用代码片段

<!--more-->

### 将类中的属性转换为字典形式

```
class Sample:
    def to_dict(self):
        return json.loads(json.dumps(self, default=lambda o: o.__dict__))
```

### 带默认值的有序字典

```
from collections import OrderedDict, Callable


class DefaultOrderedDict(OrderedDict):
    # Source: http://stackoverflow.com/a/6190500/562769
    def __init__(self, default_factory=None, *a, **kw):
        if (default_factory is not None and
           not isinstance(default_factory, Callable)):
            raise TypeError('first argument must be callable')
        OrderedDict.__init__(self, *a, **kw)
        self.default_factory = default_factory

    def __getitem__(self, key):
        try:
            return OrderedDict.__getitem__(self, key)
        except KeyError:
            return self.__missing__(key)

    def __missing__(self, key):
        if self.default_factory is None:
            raise KeyError(key)
        self[key] = value = self.default_factory()
        return value

    def __reduce__(self):
        if self.default_factory is None:
            args = tuple()
        else:
            args = self.default_factory,
        return type(self), args, None, None, self.items()

    def copy(self):
        return self.__copy__()

    def __copy__(self):
        return type(self)(self.default_factory, self)

    def __deepcopy__(self, memo):
        import copy
        return type(self)(self.default_factory,
                          copy.deepcopy(self.items()))

    def __repr__(self):
        return 'OrderedDefaultDict(%s, %s)' % (self.default_factory,
                                               OrderedDict.__repr__(self))
```

### 在django的view中返回excel文件流

```
import io

from django.http.response import HttpResponse

from xlsxwriter.workbook import Workbook


def your_view(request):

    output = io.BytesIO()

    workbook = Workbook(output, {'in_memory': True})
    worksheet = workbook.add_worksheet()
    worksheet.write(0, 0, 'Hello, world!')
    workbook.close()

    output.seek(0)

    response = HttpResponse(output.read(), content_type="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet")
    response['Content-Disposition'] = "attachment; filename=test.xlsx"

    output.close()

    return response
```

### 预激协程的装饰器

<fluent python> 16.4

```
from functools import wraps

def coroutine(func):
    @wraps(func)
    def primer(*args, **kwargs):
        gen = func(*args, **kwargs)
        next(gen)
        return gen
    return primer
```

### 使用动态属性访问JSON类数据
《流畅的Python》19.1
```
from collections import abc

class FrozenJSON:
    """一个只读接口，使用属性表示法访问JSON类对象"
    def __new__(cls, arg):
        if isinstance(arg, abc.Mapping):
            return super().__new__(cls)
        elif isinstance(arg, abc.MutableSequence):
            return [cls(item) for item in arg]
        else:
            return arg
            
    def __init__(self, mapping):
        self.__data = dict(mapping)
    
    def __getattr__(self, name):
        if hasattr(self.__data, name):
            return getattr(self.__data, name)
        else:
            return FrozenJSON(self.__data[name])
```

### 字典key转化为对象属性

tornado source code: `tornado.util.ObjectDict`
```
try:
    import typing

    _ObjectDictBase = typing.Dict[str, typing.Any]
except ImportError:
    _ObjectDictBase = dict
    
    
class ObjectDict(_ObjectDictBase):
    """Makes a dictionary behave like an object, with attribute-style access.
    """
    def __getattr__(self, name):
        # type: (str) -> Any
        try:
            return self[name]
        except KeyError:
            raise AttributeError(name)

    def __setattr__(self, name, value):
        # type: (str, Any) -> None
        self[name] = value
        
        
# 简化版
class ObjectDict:
    def __init__(self, d):
        self.__dict__ = d
```

### python对象转化为json格式字符串
```
def json_conv(data):
    """将字典中为list或dict类型的值转换为json格式的字符串，用于上传到mysql数据库"""
    for key in data.keys():
        if isinstance(data[key], list) or isinstance(data[key], dict):
            data[key] = json.dumps(data[key], ensure_ascii=False)
```

### 从data字典中仅筛选出在model_class中定义的字段的映射(用于django orm)
```
def get_model_field(model_class, data):
    return {key: value for key, value in data.items() if key in model_class._meta.get_all_field_names()}
```

### django从大到小排序pg数据库数据时null值排在前问题

原理是利用django的Func，新建一个字段，原始字段为null时为False，否则为True，排序时先对这个新建字段排序，再对原始字段排序，这样，为null的值都将在列表尾端。
```
from django.db.models import Func

class IsNull(Func):
    template = '%(expression)s IS NULL'
    
queryset = queryset.annotate(myfield_isnull=IsNull('myfield')).order_by('myfield_isnull', order_by)
```

### 发送邮件
```
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email.mime.text import MIMEText
from email import encoders


def send_email(send_from, send_to, subject, text, server_host, server_port, user, pw, file_bin_stream=None, file_name=None, is_tls=False):
    server = smtplib.SMTP(server_host, server_port)
    if is_tls:
        server.starttls()
    server.login(user, pw)

    msg = MIMEMultipart()
    msg['subject'] = subject
    msg['from'] = send_from
    msg['to'] = ','.join(send_to) if isinstance(send_to, list) else send_to

    msg.attach(MIMEText(text.encode('utf8'), 'plain', 'utf8'))

    if file_bin_stream is not None and file_name is not None:
        part = MIMEBase('application', "octet-stream")
        part.set_payload(file_bin_stream)
        encoders.encode_base64(part)
        part.add_header('Content-Disposition', 'attachment; filename="{}"'.format(file_name))
        msg.attach(part)

    server.sendmail(send_from, send_to, msg.as_string())
    server.quit()
```

### 获取Excel文件内存字节

```
import io

from openpyxl import Workbook
from openpyxl.writer.excel import save_virtual_workbook


def gen_xlsx_stream(data):
    wb = Workbook()
    ws = wb.active
    for col_num, column in enumerate(data, start=1):
        for row_num, value in enumerate(column, start=1):
            _ = ws.cell(row=row_num, column=col_num, value=value)
    ret = save_virtual_workbook(wb)
    return ret
```

### logging使用

```
logger = logging.getLogger("ad_profit_center")
logger.setLevel(logging.DEBUG)
# create formatter
fmt = "[%(asctime)-15s %(levelname)s %(filename)s %(lineno)d %(process)d] %(message)s"
datefmt = "%a %d %b %Y %H:%M:%S"
formatter = logging.Formatter(fmt, datefmt)
# create handler
sh = logging.StreamHandler(sys.stdout)
sh.setLevel(logging.DEBUG)
sh.setFormatter(formatter)
fh = logging.FileHandler("./ad_profit_center.log", mode="a", encoding="utf8", delay=False)
fh.setLevel(logging.INFO)
fh.setFormatter(formatter)

logger.addHandler(sh)
logger.addHandler(fh)
```