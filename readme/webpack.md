1.项目的搭建
ykit怎么搞，webpack怎么搞
ykit怎么把指定资源输出到指定位置，就是dsp如何实现ykit重构
webpack指定入口，出口，指定loader，指定插件，别名，本质上是传给webpack一个配置对象
打包命令 webpack webpack.config.js


假设目录结构为：
myapp
    --src
        --pages
            --index.js
        --utils
    --webpack.config.js
    --package.json


// webpack.config.js
const path = require('path');
module.exports = {
    entry: {
        index: './src/pages/index.js',
        vendor: ['react', 'react-dom', 'antd']
    },
    output: {
        path: path.resolve(DIR, 'prd'),                           // 文件打包后放到哪里
        publicPath: '//qunar.com/myapp/prd/',                     // 访问这个路径，将会返回打包文件
        filename: '[name]@[chunkhash].js'                         // 文件名,[...]是个占位符，name说明同entry的key保持一致
    },
    resolve:{
        extensions: ['.js', '.css', '.scss'],

        // 配置引用别名
        alias: {
            '$pages': path.join(__dirname, './src/pages),
            ...
        }
    },
    module: {

        // 如何编译转换各类型文件
        loaders: [
            {
                test: /\.(ttf|woff)$/,
                exclude: /node_modules/,
                loader: 'file-loader'
            }
        ]
    },

    // 插件来做一些其他事情，比如压缩打包，抽离css等
    plugins: []
};

// 如何将.css文件抽离出来extract-text-webpack-plugin