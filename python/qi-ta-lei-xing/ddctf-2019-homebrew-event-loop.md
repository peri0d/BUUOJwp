# \[DDCTF 2019]homebrew event loop

## \[DDCTF 2019]homebrew event loop

## 考点

* Python源码审计
* 购买功能中的逻辑漏洞

## wp

给了源码，下载审计

从路由入手，只有一个路由`/d5afe1f66147e857/`，`request.query_string`是获取`?`之后的所有内容，其格式为`action:ACTION;ARGS0#ARGS1#ARGS2`，通过给定格式的query\_string实现不同的功能。在初始状态对session中的值做一系列定义

```python
@app.route(url_prefix+'/')
def entry_point():
    querystring = urllib.unquote(request.query_string)
    request.event_queue = []
    if querystring == '' or (not querystring.startswith('action:')) or len(querystring) > 100:
        querystring = 'action:index;False#False'
    if 'num_items' not in session:
        session['num_items'] = 0
        session['points'] = 3
        session['log'] = []
    request.prev_session = dict(session)
    trigger_event(query_string)
    return execute_event_loop()
```

然后把获取的query\_string传给trigger\_event这个函数，这个函数把query\_string按照先后顺序放在`request.event_queue`列表中

```python
def trigger_event(event):
    session['log'].append(event)
    if len(session['log']) > 5:
        session['log'] = session['log'][-5:]
    if type(event) == type([]):
        request.event_queue += event
    else:
        request.event_queue.append(event)
```

在路由的最后执行`execute_event_loop`函数，它会获取`request.event_queue`列表中的第一个元素，然后把剩下的元素重新赋值给`request.event_queue` ，并且限制了query\_string的组成，只能由字母数字和`_:;#`组成，然后截取`ACTION` 和`ARGS` 的值，对`ACTION` 进行拼接后用eval执行，用于获取`ACTION()` 这个函数，最后再把参数传入`ACTION()` 函数。

这里action用了拼接的方式，会导致可以执行任意函数，在action后面加上`#`就可以绕过对后缀的限制。那么这里肯定会问，为什么不直接执行FLAG()函数呢，因为FLAG()函数没有参数，这里在执行函数时是传入参数的，所以只能用get\_flag\_handler()函数获取flag

```python
# 简化之后的函数
def execute_event_loop():
    event = request.event_queue[0]
    request.event_queue = request.event_queue[1:]
    action = get_mid_str(event, ':', ';') # ACTION
    args = get_mid_str(event, action+';').split('#') # [ARGS0,ARGS1,ARGS2]
    event_handler = eval(
        action + ('_handler' if is_action else '_function'))
    ret_val = event_handler(args)
```

然后去看相关的功能，view\_handler()用于展示页面，index\_handler()用于展示和下载源码，get\_flag\_handler()用于获取flag，条件是num\_items大于等于5，这个在一开始定义为0个，然后是关键的buy\_handler()和consume\_point\_function()

```python
def buy_handler(args):
    num_items = int(args[0])
    if num_items <= 0:
        return 'invalid number({}) of diamonds to buy<br />'.format(args[0])
    session['num_items'] += num_items
    trigger_event(['func:consume_point;{}'.format(
        num_items), 'action:view;index'])


def consume_point_function(args):
    point_to_consume = int(args[0])
    if session['points'] < point_to_consume:
        raise RollBackException()
    session['points'] -= point_to_consume
```

在正常购买逻辑中的`request.event_queue`是由`['action:buy;1']`变成`['func:consume_point;1', 'action:view;index']` ，它是先在session中增加完num\_items再去consume\_point\_function()中验证的

可以通过构造`request.event_queue` 为`['action:buy;5','action:get_flag;']` ，这样在执行完buy\_handler之后，trigger\_event会把剩下的内容进行拼接`request.event_queue += event` ，变成`['action:get_flag;','func:consume_point;5', 'action:view;index']`，然后会执行get\_flag\_handler()，这时候的num\_items为5，所以可以获取flag

而构造`request.event_queue` 可以通过前面的任意函数执行，执行trigger\_event函数完成

最后的payload

```
?action:trigger_event%23;action:buy;5%23action:get_flag;%23
```

## 总结

1. 代码审计跟着功能搞明白执行顺序
2. 在购买的逻辑中，如果是先把商品加1再判断余额，可能存在逻辑漏洞
