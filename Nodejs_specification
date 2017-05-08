#Node.js 代码规范

### 避免“回调地狱”

- 使用 async 模块
- 使用 promise

### 尽量用 Non-Blocking 代码，避免使用 Blocking 代码

- Non-Blocking

	```
	fs.readFile('temp.json', (err, data)=>{
		console.log(data);
	});
	```
	
- Blocking

	```
	var data = fs.readFileSync('temp.json');
	console.log(data);
	```
	
### 回调函数一定要处理error

- 错误

```
callback => {
	request('http://localhost:8080/list', (err, list) => {
		/*
		错误：
		如果request方法抛出了err, 那么list就是undefind，下面代码会抛错：
		Uncaught TypeError: list.map is not a function
		*/
		var result = result.map(l=>l.id);
		callback(err, result);
	});
}
```

- 正确

```
callback => {
	request('http://localhost:8080/list', (err, list) => {
		if(err){
			return callback(err);
		}
	
		var result = result.map(l=>l.id);
		callback(err, result);
	});	
}
```

### 给接口方法代码加注释：描述该方法实现什么功能和参数列表

```
/**
 * 保存退款信息
 * @param id
 * @param channel
 * @param userPhone
 * @param appCode
 * @param price
 * @param serialNum
 * @param type
 * @param reason
 */
 router.post('/save', (req, res, next) => {
 	...
 });
```

### 参数个数如果有很多，尽量把参数放到一个对象中；参数中如果包含回调函数，回调函数总是放到最后

- 错误习惯

```
function queryData = (city, level, lowPrice, name, callback) => {...};
```

- 正确习惯

```
/**
 * 查询
 * @param city
 * @param level
 * @param lowPrice
 * @param name
*/
function queryData = (queryCondition, callback) => {...};
```

### 避免重复调用回调函数

- 错误

```
(params, callback) => {
	callFunc(params, (err, result) => {
		if(err){
			callback(err);
		}
		
		...
		callback(null, result);
	});
}
```

- 正确

```
(params, callback) => {
	callFunc(params, (err, result) => {
		if(err){
			return callback(err);
		}
		
		...
		callback(null, result);
	});
}
```

or

```
(params, callback) => {
	callFunc(params, (err, result) => {
		if(err) {
			callback(err);
		} else {
			...
			callback(null, result);
		}
	});
}
```

#### 用`0`和`1`代替`true`和`false`

`javascript`语言的`Boolean`类型的值是`true`和`false`，其他语言比如`Objective-C`,`python`的`Boolean`类型的值是`True`和`False`.所以涉及到跨平台交互的时候，尽量用`0`和`1`代替`Boolean`

### Express 规范：

#### 1. 路由分模块

- 错误习惯

```
app.get('/v1/users/:id', (req, res, next) => {...});
app.post('/v1/users/:id/update', (req, res, next) => {...});
app.get('/v2/users/:id', (req, res, next) => {...});
```

- 正确习惯

```
app.use('/v1', require('./routers/v1'));
app.use('/v2', require('./routers/v2'));
```

#### 2. 模糊匹配的路由一定要放到后面

- 错误

```
router.get('/users/:id', (req, res) => {...});
router.get('/users/add', (req, res) => {...})
```

- 正确

```
router.get('/users/add', (req, res) => {...})
router.get('/users/:id', (req, res) => {...});
```

#### 3. 如果路由的回调方法使用了next参数，一定要避免定义匹配规则重复的路由

- 错误

```
router.get('/users/add', (req, res, next) => {
	callFunc(req.body, (err, result)=>{
		...
		next(err);
	});
});

router.get('/users/:id', (req, res, next) => {...});
```

#### 4. 使用中间件做预处理

- 错误习惯

```
router.get('/valid', (req, res) => {
	req.query.appCode = req.get('appCode');
	handler.valid(req.query, (err, result) => {...});
});

router.get('/order/save', (req, res) => {
	req.query.appCode = req.get('appCode');
	handler.saveOrder(req.query, (err, result) => {...});
});
```

- 正确习惯

```
router.get('/', (req, res, next) => {
	req.query.appCode = req.get('appCode');
	next();
});

router.get('/valid', (req, res) => {
	handler.valid(req.query, (err, result) => {...});
});

router.get('/order/save', (req, res) => {
	handler.saveOrder(req.query, (err, result) => {...});
});
```
