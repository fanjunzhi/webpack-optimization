# webpack优化之路
开发了几个月的webpack构建的项目，总要留(流)下点什么

![开篇](https://pic4.zhimg.com/0fc7bd004140b1707e7226f22b4bc0bb_b.jpg)
![开篇](https://pic1.zhimg.com/4cefc77e2e9e82f1f225d2e96aaa707c_b.jpg) 
 
什么按需加载什么的就不提了，需要知道按需加载不是适合于所有的场景。

## 1、别名alia
```
'react': (0, join)(__dirname, './node_modules/react/dist/react.min.js'),

resolve: {
    alias: alias,
},
```
## 2、css-loader < 0.15.0([相关链接](https://github.com/webpack/css-loader/issues/124))

```
"css-loader": "^0.14.1",
```
## 3、移除css-loader的sourcemap 
---

是不是感觉没多大效果啊  
![莫慌，抱紧我](https://pic2.zhimg.com/abe078526fd7ad08855b5af938fa3521_b.jpg)
## 4、外部引入模块
```
externals: {
	'react': 'React',
	'react-dom': 'ReactDOM',
	'reactUpdate': 'ReactUpdate',
	'antd': 'antd',
	"jquery": "jQuery",
	'AMap': 'window.AMap',
 	'moment': 'moment',
	'react-dnd': 'ReactDnD',
	'react-router': 'ReactRouter'
},
```
必须设置

```
output: {
	'libraryTarget': 'var',
},
```
然后html引入外部js 

```
	<script src="${assetsAt('react.min.js')}"></script>  
	<script src="${assetsAt('react-dom.min.js')}"></script>  
```
## 5、设置cache为true  
```
cache: true,
```
## 6、设置root([相关链接](https://github.com/Automattic/wp-calypso/pull/4128))
```
resolve: {
    root: [path.resolve('./src')],
},
```
## 7、设置babel的cacheDirectory为true(打包性能提升很明显,[相关链接](https://github.com/babel/babel-loader))
```
/*
 * babel参数
 * */
var babelQuery = {
    presets: ['es2015', 'react', 'stage-0'],
    plugins: ['transform-runtime', 'add-module-exports', 'typecheck', "transform-decorators-legacy"],
    cacheDirectory: true
};

loaders: [
	{
		test: /\.js$/,
		exclude: /node_modules/,
		loader: 'babel',
		query: babelQuery
	}, {
		test: /\.jsx$/,
		loader: 'babel',
		query: babelQuery
	}
]
```
## 8、一些loader的大小限制(限制小于多少才转为base64,让生成的文件最小)

```
loaders: [
	{
		test: /\.woff(\?v=\d+\.\d+\.\d+)?$/,
		loader: 'url?limit=10000&minetype=application/font-woff'
	}, {
		test: /\.woff2(\?v=\d+\.\d+\.\d+)?$/,
		loader: 'url?limit=10000&minetype=application/font-woff'
	}, {
		test: /\.ttf(\?v=\d+\.\d+\.\d+)?$/,
		loader: 'url?limit=10000&minetype=application/octet-stream'
	}, {
		test: /\.eot(\?v=\d+\.\d+\.\d+)?$/, 
		loader: 'file'
	}, {
		test: /\.svg(\?v=\d+\.\d+\.\d+)?$/,
		loader: 'url?limit=10000&minetype=image/svg+xml'
	}, {
		test: /\.(wav|mp3)?$/,
		loader: 'url-loader?limit=8192'
	}
]
```
## 9、noParse
如果你 确定一个模块中没有其它新的依赖 就可以配置这项，webpack 将不再扫描这个文件中的依赖。

```  
module: {
	loaders: [
            
	],
	noParse: [
		/moment-with-locales/
	] 
},
```
## 10、拷贝静态文件
把指定文件夹xia的文件复制到指定的目录

```
var CopyWebpackPlugin = require('copy-webpack-plugin');

var copyFile = {
    production: [
        {from: 'src/static/antd-0.12.15.min.css', to: 'antd.min.css'},
        {from: 'src/static/antd-0.12.15.min.js', to: 'antd.min.js'},
        {from: 'src/static/jquery-2.2.1.min.js', to: 'jquery.min.js'},
        {from: 'src/static/react-15.0.1.min.js', to: 'react.min.js'},
        {from: 'src/static/react-dom-15.0.1.min.js', to: 'react-dom.min.js'},
        {from: 'src/static/react-update-0.14.6.min.js', to: 'react-update.min.js'},
        {from: 'src/static/favicon.ico', to: 'favicon.ico'},
        {from: 'src/static/moment-2.13.0.min.js', to: 'moment.min.js'},
        {from: 'src/static/react-dnd-2.1.4.min.js', to: 'react-dnd.min.js'},
        {from: 'src/static/react-router-1.0.3.min.js', to: 'react-router.min.js'}
    ],
    development: [
        {from: 'src/static/antd-0.12.15.min.css', to: 'antd.min.css'},
        {from: 'src/static/antd-0.12.15.min.js', to: 'antd.min.js'},
        {from: 'src/static/jquery-2.2.1.min.js', to: 'jquery.min.js'},
        {from: 'src/static/react-development-15.0.1.min.js', to: 'react.min.js'},
        {from: 'src/static/react-dom-development-15.0.1.min.js', to: 'react-dom.min.js'},
        {from: 'src/static/react-update-0.14.6.min.js', to: 'react-update.min.js'},
        {from: 'src/static/favicon.ico', to: 'favicon.ico'},
        {from: 'src/static/moment-2.13.0.min.js', to: 'moment.min.js'},
        {from: 'src/static/react-dnd-2.1.4.min.js', to: 'react-dnd.min.js'},
        {from: 'src/static/react-router-1.0.3.min.js', to: 'react-router.min.js'}
    ]
};

new CopyWebpackPlugin(
	(runmod == 'devserver') ? copyFile.development : copyFile.production
),
```
## 11、设置dll([相关链接](http://engineering.invisionapp.com/post/optimizing-webpack/))
必须要配置json文件，对开发中依赖模块变化比较大的慎用。  

---  
---
---
---
---
---

[参考链接1](http://www.slideshare.net/trueter/how-to-make-your-webpack-builds-10x-faster)
## 12、水饺
![水饺](https://pic1.zhimg.com/e2d5be25d126edf5e036b16cd5440c70_b.jpg)
