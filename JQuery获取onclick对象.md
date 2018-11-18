### JQuery获取onclick对象

​	当页面中存在多个同类按钮，一般会分配相同的class用以相同的onclick事件，通过$(this)来获取单机的对象。这样就不需要通过class定位从而获取到所有的按钮了。

```javascript
//删除单个panel响应事件

$('.remove-panel-button').on('click',function () {

    var button = $(this);

    var panel = button.parents('.col-lg-4');

    // alert(panel.html());

    panel.remove();


})
```



### JQuery.on()

当通过jquery动态生成button时候，单机无法触发其绑定的事件，需要通过以下方法来实现。

```javascript
  $('父元素').on('click','选择器',function(){

			////

})
```

例：

```javascript
//添加panel中子skill行响应事件

$('.main').on('click','.add-panel-row',function () {

​    var button = $(this);

​    var row = "<div class='form-group'><label class = 'col-sm-2 control-label'>子skill</label><div class='col-sm-4'><input type='text' class='form-control' placeholder='标题'> </div><div class='col-sm-4'><input type='text' class='form-control' placeholder='数值'></div><div class='col-sm-2'><button type='button' class='btn btn-danger remove-panel-row'>删除</button></div></div>";

​    // alert(row);

​    button.parents('.panel-headline').children('.panel-body').append(row);

})


//删除panel中子skill行响应事件

$('.main').on('click','.remove-panel-row',function () {

​    var button = $(this);

​    var row = button.parents('.form-group')

​    // alert(row.html());

​    row.remove();

})
```

