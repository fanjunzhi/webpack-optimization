# 公司项目webpack迁移到webpack2的记录
## 1、babel -> babel-loader
## 2、不支持OccurenceOrderPlugin DedupePlugin
## 3、resolve，resolveLoader中不支持modulesDirectories
## 4、resolve中没root
## 5、resolve extensions中不允许出现’’
## 6、Error: Chunk.entry was removed. Use hasRuntime()
解决方案：

```
install extract-text-webpack-plugin@^2.0.0-beta --save-dev
```
## 7、
```
new ExtractTextPlugin('[name].css', {
            disable: false,
            allChunks: true,
        }),

```
改为

```
new ExtractTextPlugin(
            {
                  filename: '[name].css',
                  disable: false,
                  allChunks: true,
            }
        }),
```
## 8、TypeError: extractedChunk.isInitial is not a function
解决方案：
webpack 2.2.0 -> 2.1.0-beta.22
## 9、
```
new webpack.optimize.CommonsChunkPlugin('vendor', 'vendor.bundle.js')
```
改为

```
new webpack.optimize.CommonsChunkPlugin({ name: 'vendor', filename: 'vendor.bundle.js' })
```