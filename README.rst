.. raw:: html

    <p align="center">
        <a href="#readme">
            <img alt="Pre-request logo" src="https://raw.githubusercontent.com/Eastwu5788/pre-request/master/docs/static/logo.jpg">
        </a>
    </p>
    <p align="center">
        <a href="https://www.travis-ci.org/Eastwu5788/pre-request"><img alt="Travis" src="https://www.travis-ci.org/Eastwu5788/pre-request.svg?branch=master"></a>
        <a href="https://coveralls.io/github/Eastwu5788/pre-request?branch=master"><img alt="Coveralls" src="https://coveralls.io/repos/github/Eastwu5788/pre-request/badge.svg?branch=master"></a>
        <a href="https://github.com/Eastwu5788/pre-request/blob/master/LICENSE"><img alt="License" src="https://img.shields.io/pypi/l/pre-request?color=brightgreen"></a>
        <a href="https://pre-request.readthedocs.io/en/master/"><img alt="Docs" src="https://readthedocs.org/projects/pre-request/badge/?version=master"></a>
        <a href="https://pypi.org/project/pre-request/"><img alt="PyPI" src="https://img.shields.io/pypi/v/pre-request?color=brightgreen"></a>
    </p>


========

欢迎您使用pre-request框架，pre-request致力于简化请求参数验证工作。为Flask的
网络请求参数验证提供了解决方案。

pre-request提供了非常方便的使用的方法，也提供了灵活的扩展接口方便您实现自定义的
业务逻辑。

特点
----

1. 验证邮箱、手机号等特殊字段是否符合要求
2. 格式限制和转换处理，如果类型不符合或者无法转换成需求的类型，则抛出错误
3. 取值范围限制，显示参数的取值内容的范围
4. 请求参数为空和默认值处理，如果允许为空则可以设置默认值
5. 用户可以自定义callback, 自己处理任何参数（callback的调用在所有filter处理之后）
6. 可以将字段映射为内部使用的字段

安装
----

::

    pip install pre-request

快速开始
--------

::

    from pre_request import pre
    from pre_request import Rule

    field = {
        "age": Rule(direct_type=int, enum=[1, 2]),
        "name": Rule(gt=6, lt=12),
        "email": Rule(email=True),
        "mobile": Rule(mobile=True),
        "empty": Rule(required=False, default="sssss_empty"),
        "range": Rule(direct_type=int, gt=10, lt=30),
        "reg": Rule(reg=r'^h\w{3,5}o$', key_map="reg_exp"),
        "trim": Rule(trim=True, json=True),
        "call": Rule(direct_type=int, callback=lambda x: x+100)
    }

    # 不指定get和post时，不论get请求或者post请求都会使用同一个过滤参数
    # 如果指定了get或者post时，直接设置的过滤参数会被覆盖
    @app.route("/test", methods=['get', 'post'])
    @pre.catch(field)
    def test_handler(params=None):
        return str(params)


    # 单独设置get请求的过滤参数
    @app.route("/get", methods=['get'])
    @pre.catch(get=get_field)
    def get_handler(params=None):
        return str(params)

    # 单独设置post请求的过滤参数
    @app.route("/post", methods=['post'])
    @pre.catch(post=post_field)
    def post_handler(params=None):
        return str(params)

    # 同时设置get和post的过滤参数
    @app.route("/all", methods=['get', 'post'])
    @pre.catch(get=get_field, post=post_field)
    def all_handler(params=None):
        return str(params)

    # 方法视图
    @pre.catch(get=get_field)
    def get(self, params=None):
    return str(params)

    @pre.catch(post=post_field)
    def post(self, params=None):
    return str(params)

Rule 规则概览
----------------

::

    # 字段目标数据类型
    self.direct_type = kwargs.get("direct_type", str)
    # 不进行过滤，仅把参数加到结果集中
    self.skip = kwargs.get("skip", False)

    # 当前字段是否是必填项
    self.required = kwargs.get("required", True)
    self.required_with = kwargs.get("required_with", None)

    # 当前字段默认值，如果不允许为空，则次字段无意义
    self.default = kwargs.get("default", None)
    # 去除前后的空格
    self.trim = kwargs.get("trim", False)

    # 字段枚举值设置
    self.enum = kwargs.get("enum", list())

    # 正则表达式
    self.reg = kwargs.get("reg", None)
    # Email判断
    self.email = kwargs.get("email", False)
    # 手机号判断
    self.mobile = kwargs.get("mobile", False)

    # 判断字符串中包含某个子串
    self.contains = kwargs.get("contains", list())
    # 判断字符串包含任意子串
    self.contains_any = kwargs.get("contains_any", list())
    # 判断字符串中禁止包括某个子串
    self.excludes = kwargs.get("excludes", list())
    # 判断字符串开头
    self.startswith = kwargs.get("startswith", None)
    # 判断字符串结尾
    self.endswith = kwargs.get("endswith", None)
    # 字符串小写
    self.lower = kwargs.get("lower", False)
    # 字符串大写
    self.upper = kwargs.get("upper", False)

    # 判断入参是否为ipv4/ipv6
    self.ipv4 = kwargs.get("ipv4", False)
    self.ipv6 = kwargs.get("ipv6", False)
    self.mac = kwargs.get("mac", False)

    # 判断入参是否为地理坐标 经度/维度
    self.latitude = kwargs.get("latitude", False)
    self.longitude = kwargs.get("longitude", False)

    # 跨字段验证
    self.eq_key = kwargs.get("eq_key", None)
    self.neq_key = kwargs.get("neq_key", None)
    self.gt_key = kwargs.get("gt_key", None)
    self.gte_key = kwargs.get("gte_key", None)
    self.lt_key = kwargs.get("lt_key", None)
    self.lte_key = kwargs.get("lte_key", None)

    # 等于/不等于
    self.eq = kwargs.get("eq", None)
    self.neq = kwargs.get("neq", None)

    # 范围限定 direct_type 为数字时限定数字大小，为字符串时限定字符串长度
    self.gt = kwargs.get("gt", None)
    self.gte = kwargs.get("gte", None)
    self.lt = kwargs.get("lt", None)
    self.lte = kwargs.get("lte", None)

    # key映射
    self.key_map = kwargs.get("key_map", None)

    # 是否需要进行json解析
    self.json_load = kwargs.get("json", False)

    # 自定义处理callback, 在所有的filter处理完成后，通过callback回调给用户进行自定义处理
    self.callback = kwargs.get("callback", None)


Links
------------
* Documentaion: https://pre-request.readthedocs.io/en/master/index.html
* Release: https://pypi.org/project/pre-request/
* Code: https://github.com/Eastwu5788/pre-request
* Issue tracker: https://github.com/Eastwu5788/pre-request/issues
* Test status: https://coveralls.io/github/Eastwu5788/pre-request
