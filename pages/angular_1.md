# Angular简单使用

一个独立完成的小项目，前端使用angular(最新的已经是2.几版本了，博主开发时候使用当时的最新版1.3)和bootstrap，后端php，写一些笔记。

下面是项目首页，还有其他页面，隐私信息比较多，所以就不都展示了。整体逻辑和数据处理都放在了前端，ng处理。

![首页](http://ocaya4boy.bkt.clouddn.com/zhongbao1.jpeg)



##### angular：

开发Web页面的框架、模板以及数据绑定和丰富UI组件

##### 目录结构：

angular

--app

​    --app.component.js

​    --app.module.js

​    --main.js

--index.html

--styles.css

这里把官网示例的目录拿过来了，无论什么结构，条理清晰，方便使用即可。

##### Bootstrap

bootstrap是一个开源的前端框架，快捷、灵活、很容易上手使用。

[Bootstrap起步](http://v3.bootcss.com/getting-started/) 



#### 笔记

#### 路由：ui-router

```javascript
var app = angular.module('project', ['ui.router']);	// 加载ui-router

app.config(function($stateProvider, $urlRouterProvider) {
    $urlRouterProvider.otherwise('/index');	// 默认路由
    $stateProvider	
        .state('index.read', { 	// 通过state来匹配路由
            url: '/index/read/{id}',	// 花括号内容为参数
            templateUrl: '/www/project/' + 'read.html',	// 页面地址
            resolve: {
                load : asyncjs('/www/project/' + 'js/read.js')	// 动态加载js文件
            }
        })
        .state('index.desc', {
            url: '/index/desc',
            templateUrl: '/www/project/' + 'desc.html',
            resolve: {
                load : asyncjs('/www/project/' + 'js/desc.js')
            }
        })
```

##### ####动态文件加载：

```javascript
function asyncjs(js) {
  	return function ($q, $rootScope) {
            var deferred = $q.defer(); 	
            var dependencies = js;

            if (Array.isArray(dependencies)) {
                for (var i = 0; i < dependencies.length; i++) {
                    dependencies[i] += "?v=" + v;
                }
            } else {
                dependencies += "?v=" + v;//v是版本号
            }
            /**
             * 用$script加载js文件
             */
            $script(dependencies, function () {
                $rootScope.$apply(function () {
                    deferred.resolve();
                });
            });
            return deferred.promise;
      }
}
```

- $q是Angular的一种内置服务，用于异步执行函数，并且可以返回值或者异常。 $q.defer()是创建延时实列，defer中的resolve()表示执行成功并派生promise，当执行defer.promise时候会显示defer的执行结果。

- $script 是一个异步加载器，加载js文件 

- angular中也有个 $ocLoayLoad, 按需加载。

  ​

#### 配置HTTP服务

angular 提供$http服务，来用于通信，并且参数和返回结构都是对象。所以需要转换一下,在后台get或者post才能获取到数据

第一种：

```javascript
//	通过$httpProvider拦截器 ，修改头和数据信息。
app.config(function($httpProvider) {
    $httpProvider.defaults.headers.put['Content-Type'] = 'application/x-www-form-urlencoded';
    $httpProvider.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
    // Override $http service's default transformRequest
    $httpProvider.defaults.transformRequest = [function(data) {
        /**
         * The workhorse; converts an object to x-www-form-urlencoded serialization.
         * @param {Object} obj
         * @return {String}
         */
        var param = function(obj) {
            var query = '';
            var name, value, fullSubName, subName, subValue, innerObj, i;

            for (name in obj) {
                value = obj[name];

                if (value instanceof Array) {
                    for (i = 0; i < value.length; ++i) {
                        subValue = value[i];
                        fullSubName = name + '[' + i + ']';
                        innerObj = {};
                        innerObj[fullSubName] = subValue;
                        query += param(innerObj) + '&';
                    }
                } else if (value instanceof Object) {
                    for (subName in value) {
                        subValue = value[subName];
                        fullSubName = name + '[' + subName + ']';
                        innerObj = {};
                        innerObj[fullSubName] = subValue;
                        query += param(innerObj) + '&';
                    }
                } else if (value !== undefined && value !== null) {
                    query += encodeURIComponent(name) + '='
                            + encodeURIComponent(value) + '&';
                }
            }

            return query.length ? query.substr(0, query.length - 1) : query;
        };

        return angular.isObject(data) && String(data) !== '[object File]'
                ? param(data)
                : data;
    }];
});
```

第二种：

```javascript
// 封装一层 在header头中使用json格式，对后端也需要处理，博主使用php的yii框架，getParams方法获取
var doRequestPost = function(data, path) {
  return $http({
    method      : 'POST',
    url         : path,
    data        : data,
    headers: {
      'Content-type': 'application/json'
    }
    /*headers: { 'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'}*/
  });
}
```

第三种：

后台获取对象在解析，PHP中可通过 file_get_contents('php://input', true)获取。



#### 自定义服务

##### factory() : 最简单的创建方法，里面可以使用$http、$q等内置指令，可以return任何。

用于只处理一个方法或者一些简单逻辑

```javascript
//	通过factory来封装一层$http
app.factory('httpService', ['$http', 
    function($http) {
        var doRequestPost = function(data, path) {
            return $http({
                method      : 'POST',
                url         : path,
                data        : data,
                headers: {
                    'Content-type': 'application/json'
                }
                /*headers: { 'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'}*/
            });
        }
        var doRequestGet = function(path){
            return $http({
                method      : 'GET',
                url         : path,
                headers: {
                    'Content-type': 'application/json'
                }
            });
        }
        return {
            /**
             * listPost post请求
             * @param  {object} data [请求数据]
             * @param  {string} path [请求路径]
             * @return {object}      [返回angular的$http请求]
             */
            post : function(data, path) {
                return doRequestPost(data, path);
            },
            /**
             * listGet get请求
             * @param  {string} path [请求路径]
             * @return {object}      [返回angular的$http请求]
             */
            get : function(path){
                return doRequestGet(path);
            }
        };
    }
```

##### server：相当于一个单列，可以不用return返回

可以用来共享控制器变量或者用于功能比较多的controller

```javascript
app.service('indexName', function () {
  this.name = 'NAME';
});
```

##### provider：相对于前两个方法，provider可以用在.config中，必须实现$get方法

```javascript
app.provider('User', function() {
  this.backendUrl = "http://localhost:3000";
  this.setBackendUrl = function(newUrl) {
    if (url) this.backendUrl = newUrl;
  }
  this.$get = function($http) {
    var self = this;
    var service = {
      user: {},
      setName: function(newName) {
        service.user['name'] = newName;
      },
      setEmail: function(newEmail) {
        service.user['email'] = newEmail;
      },
      save: function() {
        return $http.post(self.backendUrl + '/users', {
          user: service.user
        })
      }
    };
    return service;
  }
});

app.config(function(UserProvider) {
  UserProvider.setBackendUrl("http://127.0.0.1/api");
});

app.controller('MainCtrl', function($scope, User) {
  $scope.saveUser = User.save;
});
```



#### 页面绑定：ng-controller

通过ng-controller 为页面绑定控制器

```ht
<div ng-controller="index">
	<p>{{name}}</p>
</div>
```

```javascript
app.controller('index',function($scope, $rootScope, $location, $stateParams, $sce, httpService){ // httpService 自定义的服务不带$符号， 且放在后面
  var name = $scope.name; // 与界面中的{{name}}值双向数据绑定
  name = httpService.get('index/name').success(function(data, status){
    if(data.code == 1000){
      name = data.name;
    }

  }).error(function(data, status){
    console.log(status);
  });
}
  
```



#### 过滤器filter

```javascript
app.filter('sort', ['$sce' , function($sce) {	//	过滤器中也可以使用指令
    return function (info) {	
        if(info){
            return '是';
        }else{
            return '否';
        }
    }
}]);
```



#### 常用服务指令：

$scope : 作用域 

$rootScope : 根作用域 

$location：解析url功能

$stateParams : 获取路由参数

$sce：通过$sce.trustAsHtml()和ng-bind-html来安全的使用html标记的内容。

$interval、$timeout ：定时器

$http : 请求服务



