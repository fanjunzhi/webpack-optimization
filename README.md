# 关于webpack2的优化([公司项目webpack迁移到webpack2的记录](https://github.com/fanjunzhi/webpack-optimization/blob/master/webpack-to-webpack2.md))
## 1、配置babel让它在编译转化es6代码时不把import export转换为cmd的module.export
```
loader: 'babel-loader',
options: {
	presets: [['es2015', {modules: false}]]
}
```
## 2、移除plugins中的add-module-exports
## 3、import尽量具体到某个模块
```
使用
import map from "lodash-es/map";
而不是
import {map} from "lodash-es";
```
## 4、用export const a代替exports.a
```
使用
export const a = "A_VAL_ES6";
export const b = "B_VAL_ES6";
代替
exports.a = "A_VAL_COMMONJS";
exports.b = "B_VAL_COMMONJS";
```
```
entry.js

import {a as a_es6, b as b_es6} from "./lib.js";
import {a as a_commonjs, b as b_commonjs} from "./lib_commonjs.js";

console.log(`Hello world: ${a_es6}`);
console.log(`Hello world: ${a_commonjs}`);

lib.js
export const a = "A_VAL_ES6";
export const b = "B_VAL_ES6";

lib_commonjs.js
exports.a = "A_VAL_COMMONJS";
exports.b = "B_VAL_COMMONJS";
```
build production result:

![build-production-result.png](https://github.com/fanjunzhi/webpack-optimization/blob/master/build-production-result.png)

1、2、3的相关知识链接 [Why Webpack 2's Tree Shaking is not as effective as you think](https://advancedweb.hu/2017/02/07/treeshaking/?utm_source=javascriptweekly&utm_medium=email)

## 5、使用[webpack-uglify-parallel](https://github.com/tradingview/webpack-uglify-parallel)代替webpack自带的UglifyJsPlugin（多核压缩代码，提升n（发布机的核数 － 1）压缩速度）-重点推荐

```
webpackConfig.plugins.some(function(plugin, i) {
        if (plugin instanceof webpack.optimize.UglifyJsPlugin) {
            webpackConfig.plugins.splice(i, 1);
            return true;
        }
    });
    
    const os = require('os');

    const options = {
        workers: os.cpus().length,
        compress: {
            warnings: true,
            drop_console: true,
            pure_funcs: ['console.log'],
        },
        //mangle: {
        //    except: ['$super', '$', 'exports', 'require']
        //},
        mangle: false,
        output: {
            comments: false,
            ascii_only: false,
        },
        sourceMap: false,
    };

    const UglifyJsParallelPlugin = require('webpack-uglify-parallel');
    webpackConfig.plugins.push(
        new UglifyJsParallelPlugin(options)
    );
```

# webpack优化之路
开发了几个月的webpack构建的项目，总要留(流)下点什么

![开篇](https://pic1.zhimg.com/4cefc77e2e9e82f1f225d2e96aaa707c_b.jpg)

什么按需加载、提取出common什么的就不提了，需要知道按需加载不是适合于所有的场景。

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
把指定文件夹下的文件复制到指定的目录

```
var CopyWebpackPlugin = require('copy-webpack-plugin');

var copyFile = {
    production: [
        {from: 'src/static/antd-0.12.15.min.css', to: 'antd.min.css'},
        {from: 'src/static/antd-0.12.15.min.js', to: 'antd.min.js'},
    ],
    development: [
        {from: 'src/static/antd-0.12.15.min.css', to: 'antd.min.css'},
        {from: 'src/static/antd-0.12.15.min.js', to: 'antd.min.js'},
    ]
};

new CopyWebpackPlugin(
	(runmod == 'devserver') ? copyFile.development : copyFile.production
),
```
## 11、设置dll([相关链接](http://engineering.invisionapp.com/post/optimizing-webpack/))
原理就是将特定的模块在项目构建前构建好，然后通过页面引入。

## 12、使用happypack([相关链接](https://github.com/amireh/happypack))  
让loader多进程去处理文件。  

```
var HappyPack = require('happypack');

loader:
	test: /\.js$/,
	exclude: /node_modules/,
	loader: 'babel',
	query: babelQuery,
	happy:   { id: 'babelJs' }
plugins:
	new HappyPack({
		id: 'babelJs' ,
		threads: 4
	}),
```

## 13、设置react-optimize (针对React [相关链接](https://github.com/thejameskyle/babel-react-optimize))
A Babel preset and plugins for optimizing React code.

## 14、减少dev开发模式减少日志信息输出的时间[相关链接](https://github.com/webpack/webpack/issues/1191)
```
{
  devServer: {
    stats: 'errors-only',
  },
}
```
## 15、自动处理不兼容的css前缀[postcss](https://github.com/postcss/postcss-loader)
## 16、devtool的选择[webpack devtool](https://webpack.github.io/docs/configuration.html#devtool)
## 17、区分开发环境、测试环境和生产环境
```
开发环境不做任何处理
测试环境(移除console)
let stripStr = '?strip[]=thisjustaplacehoderfunction';

if (process.env.NODE_ENV === 'test') {
    stripStr = '?strip[]=console.log';
    webpackConfig.module.loaders.push({
        test: /\.js$/,
        exclude: /node_modules/,
        loader: `strip-loader${stripStr}`,
    });
    webpackConfig.module.loaders.push({
        test: /\.jsx$/,
        exclude: /node_modules/,
        loader: `strip-loader${stripStr}`,
    });
}

生产环境(移除console并压缩混淆)
if (process.env.NODE_ENV === 'production') {
    webpackConfig.plugins.push(
        new webpack.optimize.UglifyJsPlugin({
            compress: {
                warnings: true,
                drop_console: true,
                pure_funcs: ['console.log'],
            },
            mangle: {
                except: ['$super', '$', 'exports', 'require'],
            },
            output: {
                comments: false,
            },
            sourceMap: false,
        })
    );
}
```
## 18、Webpack 的静态资源持久缓存(服务端支持资源加版本号的就不用考虑使用这个)

使用 webpack 开启静态资源的持久缓存：

1. 使用 [chunkhash] 为每个文件增加一个内容相关的缓存清道夫；

2. 使用编译统计在 HTML 中获取资源时取得文件名；

3. 生成 JSON 格式的模块清单文件，并在 HTML 页面加载资源之前内联进去；

4. 保证包含启动代码的入口块不会对于同样的依赖生成不同的哈希值；

5. 开始收益！

[来不及解释了，还是直接给链接吧，点我](http://www.zcfy.cc/article/long-term-caching-of-static-assets-with-webpack-1204.html)

---

## 参考链接
1. [http://www.slideshare.net/trueter/how-to-make-your-webpack-builds-10x-faster](http://www.slideshare.net/trueter/how-to-make-your-webpack-builds-10x-faster)  
2. [https://github.com/erikras/react-redux-universal-hot-example/issues/616](https://github.com/erikras/react-redux-universal-hot-example/issues/616)

## 水饺
![水饺](https://pic1.zhimg.com/e2d5be25d126edf5e036b16cd5440c70_b.jpg)
