---
layout: post
title: AngularJS ngRepeat的一点说明
categories: angularjs
lastUpdated: 6.2
---

## {{ page.title }}

{{ page.date | date: "%Y.%-m.%-d" }} - 南京 | <a href="/archive#{{ page.categories }}">{{ page.categories}}</a>

angularjs的ng-repeat的默认生成格式是**ng-repeat="(key, value) in object"**，它可以输出键值对，打印 key=**&#123;{key}}**, value=**&#123;{value}}**，这是输出对象的，如果是输出单纯数组的，直接**ng-repeat="item in collection"**，打印**&#123;{item}}**，如果输出对象数组的，也一样直接**ng-repeat="item in collection"**，打印**&#123;{item.prop}}**

**JavaScript**

```javascript
$scope.data = {
  'id': 1,
  'project': 'equip2012',
  'date': '2012-02-26',
  'description': 'equip',
  'equip_no': 'e001',
};
var array = [];
for (var key in $scope.data) {
  var map = {};
  map[key] = $scope.data[key];
  array.push(map);
}
$scope.data = array;
```

**HTML**

```html
<p ng-repeat="obj in data">
  <font ng-repeat="(key, value) in obj">
    {% raw %}{{key}} : {{value}}{% endraw %}
  </font>
</p>
```

**完整的小例子**

```html
<!DOCTYPE html>
<html ng-app>
<!-- www.java2s.com -->
<head>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.13/angular.js"></script>
</head>
<body ng-app="myApp">
  <div ng-controller="MyCtrl">
    <ul>
      <li ng-repeat="(key, value) in school.sub">first language is = {% raw %}{{value}}{% endraw %}</li>
    </ul>
  </div>
  <script type='text/javascript'>
    var myApp = angular.module('myApp', []);
    function MyCtrl($scope) {
      $scope.school = {
        name: 'A',
        sub: {
          firstlang: 'Java'
        }
      }
    }
  </script>
</body>
</html>
```

**更新列表：**

* 2016-5-27
* 修改输出花括号格式



**参考文章：**

* [AngularJS Tutorial - ng-repeat with key and value pair][1]
* [How can I iterate over the keys, value in ng-repeat in angular][2]
* [THEDATAPOINTENGINEERING8 FEATURES OF NG-REPEAT
8 Features of ng-repeat][3]


[1]: http://www.java2s.com/Tutorials/Javascript/AngularJS_Example/Directives/2920__ng_repeat_with_key_and_value_pair.htm
[2]: http://stackoverflow.com/questions/15127834/how-can-i-iterate-over-the-keys-value-in-ng-repeat-in-angular
[3]: https://blog.rjmetrics.com/2015/09/02/8-features-of-ng-repeat/
