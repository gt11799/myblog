title: 如何修改Django_Admin的页面
date: 2017-08-02 21:04:58
tags: Python
---

今天有人求助我如何修改Django Admin后台的页面，比如在右上角加个按钮

![](http://7xlo8n.com1.z0.glb.clouddn.com/WX20170802-210835.png)

原始需求是，他想做个导出按钮，可以导出当前页面的数据。

查了半天，Django Admin提供了一个change_list_template的一个方法，可以替代list的template

新建一个template，放在Django Admin规定的目录下，就会自动替代template。我这里直接修改了change_list_template

```
class MyModelAdmin(admin.ModelAdmin):
    model = MyModel
    list_display = ['name', 'description', 'created', 'updated']
    fields = ['name', 'description']
    change_list_template = 'add_button.html'
```

然后去改`add_button.html`

```
{% extends "admin/change_list.html" %}
{% block object-tools-items %}
<script>
function export_file() {
    var export_url = '/export';
    var url = window.location.href;
    var params = url.split('?')[1];
    if (typeof(params) == undefined) {
        export_url += '?' + params;
    };
    window.open(export_url);
}
</script>
<li>
    <a href="#" class="grp-state-focus" onclick="export_file()">导出</a>
</li>
{{ block.super }}
{% endblock %}
```

`admin/change_list.html`是原始模板，这个按钮所在的block是`object-tools-items`, 使用`block.super`来继承之前的内容

修改之后效果如下：

![](http://7xlo8n.com1.z0.glb.clouddn.com/WX20170802-211133.png)
