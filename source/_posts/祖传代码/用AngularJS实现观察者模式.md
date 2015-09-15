postid: "observe-pattern-by-angularjs"
title: 用AngularJS实现观察者模式
date: 2014-07-18 16:55:03
categories: [祖传代码]
tags: [javascript, ng, 实践]
---

观察者模式一种应用非常广泛的设计模式。在AngularJS构建的webapp中也有很多的应用场景。众所周知，AngularJS不同的Controller之间的通信、数据共享有多重途径，比如：官方提供的`$broadcast`，`$emit`服务，使用全局对象等。当然，我们也可以使用**观察者模式**来达到不同控制器之间的通信。

下面我们将会介绍，如何在angularjs的service中封装一个观察者。


直接上代码：

```javascript
var services = angular.module('services', []);

services.factory('ob', function() {
    var exports,
        channels = {};

    exports = {
        subscribe: function(topic, callback) {
            if (!_.isArray(channels[topic])) {
                channels[topic] = [];
            }
            var handlers = channels[topic];
            handlers.push(callback);
        },
        unsubscribe: function(topic, callback) {
            if (!_.isArray(channels[topic])) {
                return;
            }
            var handlers = channels[topic];
            var index = _.indexOf(handlers, callback);
            if (index >= 0) {
                handlers.splice(index, 1);
            }
        },
        publish: function(topic, data) {
            var self = this;
            var handlers = channels[topic] || [];
            _.each(handlers, function(handler) {
                try {
                    handler.apply(self, [data]);
                } catch (ex) {
                    console.log(ex);
                }
            });
        }
    };

    return exports;
});
```

上面就是观察者模式的核心实现。在实际项目中，可能会根据需求作一些微调。

