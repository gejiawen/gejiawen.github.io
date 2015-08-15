title: "通读SuperAgent文档"
date: 2015-08-14 16:51:17
categories: [Nodejs]
tags: [superagent, nodejs, 翻译, 中文文档, 漫游NodeJS系列]

---

本文主要参考superagent的[官方文档](http://visionmedia.github.io/superagent/)，基本上就是它的翻译。

题外话，superagent真是一个不错的nodejs模块，推荐使用。

# 前言

[superagent](https://github.com/visionmedia/superagent)是一个流行的nodejs第三方模块，专注于处理服务端/客户端的http请求。

在nodejs中，我们可以使用内置的http等模块来进行请求的发送、响应处理等操作，不过superagent提供了更加简单、优雅的API，让你在处理请求时更加方便。而且它很轻量，学习曲线平滑，内部其实就是对内置模块的封装。

下面让我们先来看一个简单的例子，大致的了解下request的基本用法，以及它和使用内置模块的区别。

```javascript
var request = require('superagent');
var http = require('http')
var queryString = require('queryString');

// 使用superagent
request
    .post('/api/pet')
    .send({ name: 'Manny', species: 'cat' })
    .set('X-API-Key', 'foobar')
    .set('Accept', 'application/json')
    .end(function(err, res){
        if (res.ok) {
            alert('yay got ' + JSON.stringify(res.body));
        } else {
            alert('Oh no! error ' + res.text);
        }
    });

// 使用http
var postData = queryString.stringify({name: 'Manny', species: 'cat'});
var options = {
    path: '/api/pet',
    method: 'POST',
    headers: {
        'X-API-Key': 'foobar',
        'Accept': 'application/json'
    }
};

var req = http.request(options, function (res) {
    res.on('data', function (chunk) {
        console.log(chunk);
    });
});

req.on('error', function (err) {
    console.log(err);
});

req.write(postData);
req.end();
```

从上面的代码中，我们可以看出，使用superagent发送一个post请求，并设置了相关请求头信息（可以链式操作），相比较使用内置的模块，要简单很多。

# 基本用法

在`var request = require('superagent')`之后，`request`对象上将会有许多方法可供使用。我们可以使用`get`、`post`等方法名来声明一个get或者post请求，然后再通过`end()`方法来发出请求。下面是个例子。

```javascript
request
    .get('/search')
    .end(function (err, res) {
    
    })
```

这里有一点需要注意，就是只要当调用了`end()`方法之后，这个请求才会发出。在调用`end()`之前的所有动作其实都是对请求的配置。

我们在具体使用时，还可以将请求方法作为字符串参数，比如

```javascript
request('GET', '/search').end(function (err, res) {

});
```

在nodejs客户端中，还可以提供绝对路径，

```javascript
request
    .get('http://www.baidu.com/search')
    .end(function (err, res) {
    
    });
```

其余的http谓语动词也是可以使用，比如**DELETE**、**HEAD**、**POST**、**PUT**等等。使用的时候，我们只需要变更`request[METHOD]`中的`METHOD`即可。

```javascript
request
    .head('/favicon.ico')
    .end(function (err, res) {
    
    });
```

不过，针对**`DELETE`**方法，有一点不同，因为`delete`是一个保留关键字，所以我在使用的时候应该是使用**`del()`**而不是`delete()`。

```javascript
request
    .del('/user/111')
    .end(function (err, res) {
    
    });
```

superagent默认的http方法是**GET**。就是说，如果你的请求是`get`请求，那么你可以省略http方法的相关配置。懒人必备。

```javascript
request('/search', function(err, res) {

});
```

# 设置请求头

在之前的例子中，我们看到使用内置的http模块在设置请求头信息时，要求传递给`http.request`一个`options`，这个`options`包含了所有的请求头信息。相对这种方式，superagent提供了一种更佳简单优雅的方式。在superagent中，设置头信息，使用`set()`方法即可，而且它有多种形式，必能满足你的需求。

```javascript
// 传递key-value键值对，可以一次设置一个头信息
request
    .get('/search')
    .set('X-API-Key', 'foobar')
    .set('Accept', 'application/json')
    .end(function (err, res) {
    
    });

// 传递对象，可以一次设置多次头信息
request
    .get('/search2')
    .set({
        'API-Key': 'foobar',
        'Accept': 'application/json'
    })
    .end(function (err, res) {
    
    });
```

# GET请求

在使用`super.get`处理GET请求时，我们可以使用`query()`来处理请求字符串。比如，

```javascript
request
    .get('/search')
    .query({name: 'Manny'})
    .query({range: '1..5'})
    .query({order: 'desc'})
    .end(function (err, res) {
    
    });
```

上面的代码将会发出一个`/search?name=Manny&range=1..5&order=desc`的请求。

我们可以使用`query()`传递一个大对象，将所有的查询参数都写入。

```javascript
request
    .get('/search')
    .query({
        name: 'Manny',
        range: '1..5',
        order: 'desc'
    })
    .end(function (err, res) {
    
    });
```

我们还可以简单粗暴的直接将整个查询字符串拼接起来作为`query()`的参数，

```javascript
request
    .get('/search')
    .query('name=Manny&range=1..5&order=desc')
    .end(function (err, res) {
    
    });
```

或者是实时拼接字符串，

```javascript
var name = 'Manny';
var range = '1..5';
var order = 'desc';

request
    .get('/search')
    .query('name=' + name)
    .query('range=' + range)
    .query('order=' + order)
    .end(function (err, res) {
    
    });
```

# HEAD请求

在`request.head`请求中，我们也可以使用`query()`来设置参数，

```javascript
request
    .head('/users')
    .query({
        email: 'joe@gmail.com'
    })
    .end(function (err, res) {
    
    });
```

上面的代码将会发出一个`/users?email=joe@gamil.com`的请求。

# POST/PUT请求

一个典型的json post请求看起来像下面这样，

```javascript
request
    .post('/user')
    .set('Content-Type', 'application/json')
    .send('{"name": "tj", "pet": "tobi"}')
    .end(callback);
```

我们通过`set()`设置一个合适的`Content-Type`，然后通过`send()`写入一些post data，最后通过`end()`发出这个post请求。注意这里我们`send()`的参数是一个json字符串。

因为json格式现在是一个通用的标准，所以默认的`Content-Type`就是json格式。下面的代码跟前一个示例是等价的，

```javascript
request
    .post('/user')
    .send({name: "tj", pet: "tobi"})
    .end(callback);
```

我们也可以使用多个`send()`拼接post data，

```javascript
request
    .post('/user')
    .send({name: "tj"})
    .send({pet: 'tobi'})
    .end(callback);
```

`send()`方法发送字符串时，将默认设置`Content-Type`为`application/x-www-form-urlencoded`，多次`send()`的调用，将会使用`&`将所有的数据拼接起来。比如下面这个例子，我们发送的数据将为`name=tj&per=tobi`。

```javascript
request
    .post('/user')
    .send('name=tj')
    .send('pet=tobi')
    .end(callback);
```

superagent的请求数据格式化是可以扩展的，不过默认支持`form`和`json`两种格式，想发送数据以`application/x-www-form-urlencoded`类型的话，则可以简单的调用`type()`方法传递`form`参数就行，`type()`方法的默认参数是`json`。

下面的代码将会使用`"name=tj&pet=tobi"`来发出一个post请求。

```javascript
request
    .post('/user')
    .type('form')
    .send({name: 'tj'})
    .send({pet: 'tobi'})
    end(callback);
```

# 设置Content-Type

设置`Content-Type`最常用的方法就是使用`set()`，

```javascript
request
    .post('/user')
    .set('Content-Type', 'application/json')
    .end(callback);
```

另一个简便的方法是调用`type()`方法，传递一个规范的`MIME`名称，包括`type/subtype`，或者是一个简单的后缀就像`xml`，`json`，`png`这样。比如，

```javascript
request.post('/user').type('application/json')

request.post('/user').type('json')

request.post('/user').type('png')
```

# 设置Accept

跟`type()`类似，设置accept也可以简单的调用`accept()`。参数接受一个规范的`MIME`名称，包括`type/subtype`，或者是一个简单的后缀就像`xml`，`json`，`png`这样。这个参数值将会被`request.types`所引用。

```javascript
request.get('/user').accept('application/json')

request.get('/user').accept('json')

request.get('/user').accept('png')
```

# 查询字符串

当用`send(obj)`方法来发送一个post请求，并且希望传递一些查询字符串时，可以调用`query()`方法，比如向`?format=json&dest=/login`发送post请求，

```javascript
request
    .post('/')
    .query({format: 'json'})
    .query({dest: '/login'})
    .send({post: 'data', here: 'wahoo'})
    .end(callback);
```

# 解析响应体

superagent会解析一些常用的格式给请求者，当前支持`application/x-www-form-urlencoded`，`application/json`，`multipart/form-data`。

## JSON/Urlencoded

此种情况下，`res.body`是一个解析过的对象。比如一个请求的响应是一个json字符串`'{"user": {"name": "tobi"}}'`，那么`res.body.user.name`将会返回`tobi`。

同样的，`x-www-form-urlencoded`格式的`user[name]=tobi`解析完的值是一样的。

## Multipart

nodejs客户端通过[Formidable](https://github.com/felixge/node-formidable)模块来支持`multipart/form-data`类型，当解析一个`multipart`响应时，`res.files`属性就可以用了。假设一个请求响应下面的数据，

```
--whoop
Content-Disposition: attachment; name="image"; filename="tobi.png"
Content-Type: image/png

... data here ...
--whoop
Content-Disposition: form-data; name="name"
Content-Type: text/plain

Tobi
--whoop--
```

你将可以获取到`res.body.name`名为`'Tobi'`，`res.files.image`为一个**file**对象，包括一个磁盘文件路径，文件名称，还有其它的文件属性等。

# 响应体属性

`response`对象设置了许多有用的标识和属性。包括`response.text`，解析后的`respnse.body`，头信息，返回状态标识等。

## Response Text

`res.text`包含未解析前的响应内容。基于节省内存和性能因素的原因，一般只在mime类型能够匹配`text/`，`json`，`x-www-form-urlencoding`的情况下默认为nodejs客户端提供。因为当响应以文件或者图片等大体积文件的情况下将会影响性能。

## Response Body

跟请求数据自动序列化一样，响应数据也会自动的解析。当`Content-Type`定义一个解析器后，就能自动解析，默认解析的Content-Type包含`application/json`和`application/x-www-form-urlencoded`，可以通过访问`res.body`来访问解析对象。

## Response header fields

`res.header`包含解析之后的响应头数据，键值都是小写字母形式，比如`res.header['content-length']`。

## Response Content-Type

`Content-Type`响应头字段是一个特列，服务器提供`res.type`来访问它，同时`res.charset`默认是空的。比如，Content-Type值为`text/html; charset=utf8`，则`res.type`为`text/html`，`res.charset`为`utf8`。

## Response Status

响应状态标识可以用来判断请求是否成功以及一些其他的额外信息。除此之外，还可以用superagent来构建理想的restful服务器。目前提供的标识定义如下，

```javascript
var type = status / 100 | 0;

// status / class
res.status = status;
res.statusType = type;

// basics
res.info = 1 == type;
res.ok = 2 == type;
res.clientError = 4 == type;
res.serverError = 5 == type;
res.error = 4 == type || 5 == type;

// sugar
res.accepted = 202 == status;
res.noContent = 204 == status || 1223 == status;
res.badRequest = 400 == status;
res.unauthorized = 401 == status;
res.notAcceptable = 406 == status;
res.notFound = 404 == status;
res.forbidden = 403 == status;
```

# 中断请求

要想中断请求，可以简单的调用`request.abort()`即可。

# 请求超时

可以通过`request.timeout()`来定义超时时间。当超时错误发生时，为了区别于别的错误，`err.timeout`属性被定义为超时时间（一个`ms`的值）。注意，当超时错误发生后，后续的请求都会被重定向。意思就说超时是针对所有的请求而不是单个某个请求。

# 基本授权

nodejs客户端目前提供两种基本授权的方式。

一种是通过类似下面这样的url，

```javascript
request.get('http://tobi:learnboost@local').end(callback);
```

意思就是说要求在url中指明`user:password`。

第二种方式就是通过`auth()`方法。

```javascript
request
    .get('http://local')
    .auth('tobi', 'learnboost')
    .end(callback);
```

# 跟随重定向

默认是向上跟随5个重定向，不过可以通过调用`res.redirects(n)`来设置个数，

```javascript
request
    .get('/some.png')
    .redirects(2)
    .end(callback);
```

# 管道数据

nodejs客户端允许使用一个请求流来输送数据。比如请求一个文件作为输出流，

```javascript
var request = require('superagent');
var fs = require('fs');

var stream = fs.createReadStream('path/to/my.json');
var req = request.post('/somewhere');
req.type('json');
stream.pipe(req);
```

或者将一个响应流写到文件中，

```javascript
var request = require('superagent')
var fs = require('fs');

var stream = fs.createWriteStream('path/to/my.json');
var req = request.get('/some.json');
req.pipe(stream);
```

# 复合请求

superagent用来构建复合请求非常不错，提供了低级和高级的api方法。

低级的api是使用多个部分来表现一个文件或者字段，`part()`方法返回一个新的部分，提供了跟request本身相似的api方法。

```javascript
var req = request.post('/upload');

req.part()
    .set('Content-Type', 'image/png')
    .set('Content-Disposition', 'attachment; filename="myimage.png"')
    .write('some image data')
    .write('some more image data');

req.part()
    .set('Content-Disposition', 'form-data; name="name"')
    .set('Content-Type', 'text/plain')
    .write('tobi');

req.end(callback);
```

## 附加文件

上面提及的高级api方法，可以通用`attach(name, [path], [filename])`和`field(name, value)`这两种形式来调用。添加多个附件也比较简单，只需要给附件提供自定义的文件名称，同样的基础名称也要提供。

```javascript
request
    .post('/upload')
    .attach('avatar', 'path/to/tobi.png', 'user.png')
    .attach('image', 'path/to/loki.png')
    .attach('file', 'path/to/jane.png')
    .end(callback);
```

## 字段值

跟html的字段很像，你可以调用`field(name, value)`方法来设置字段。假设你想上传一个图片的时候带上自己的名称和邮箱，那么你可以像下面写的那样，

```javascript
request
    .post('/upload')
    .field('user[name]', 'Tobi')
    .field('user[email]', 'tobi@learnboost.com')
    .attach('image', 'path/to/tobi.png')
    .end(callback);
```

# 压缩

nodejs客户端本身对响应体就做了压缩，而且做的很好。所以这块你不需要做额外的事情了。

# 缓冲响应

当想要强制缓冲`res.text`这样的响应内容时，可以调用`buffer()`方法。想取消默认的文本缓冲响应，比如`text/plain`，`text/html`这样的，可以调用`buffer(false)`方法。

一旦设置了缓冲标识`res.buffered`，那么就可以在一个回调函数里处理缓冲和没缓冲的响应。

# 跨域资源共享

`withCredentials()`方法可以激活发送原始cookie的能力。不过只有在**Access-Control-Allow-Origin**不是一个通配符(*)，并且**Access-Control-Allow-Credentials**为`true`的情况下才可以。

```javascript
request
    .get('http://localhost:4001/')
    .withCredentials()
    .end(function(err, res, next){
        assert(200 == res.status);
        assert('tobi' == res.text);
        next();
    });
```

# 异常处理

当请求出错时，superagent首先会检查回调函数的参数数量，当**err**参数提供的话，参数就是两个，如下

```javascript
request
    .post('/upload')
    .attach('image', 'path/to/tobi.png')
    .end(function(err, res){

    });
```

如果没有回调函数，或者回调函数只有一个参数的话，可以添加error事件的处理。

```javascript
request
    .post('/upload')
    .attach('image', 'path/to/tobi.png')
    .on('error', errorHandle)
    .end(function(res){
    
    });
```

注意：superagent**不认为**返回**4xx**和**5xx**的情况是错误。比如当请求返回500或者403之类的状态码时，可以通过`res.error`或者`res.status`等属性来查看。此时并不会有错误对象传递到回调函数中。当发生网络错误或者解析错误时，superagent才会认为是发生了请求错误，此时会传递一个错误对象`err`作为回调函数的第一个参数。

当产生一个4xx或者5xx的http响应，`res.error`提供了一个错误信息的对象，你可以通过检查这个来做某些事情。

```javascript
if (err && err.status === 404) {
    alert('oh no ' + res.body.message);
} else if (err) {
    // all other error types we handle generically
}
```



End! All rights reserved `@gejiawen`.




