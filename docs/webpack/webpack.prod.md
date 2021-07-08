## 生产环境常用配置

```js
// 生产环境配置
const { resolve } = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

process.env.NODE_ENV = 'production';

const commonCssLoader = [
  // 将css从js中提取成单独的文件
  MiniCssExtractPlugin.loader,
  'css-loader',
  {
    loader: 'postcss-loader',
    options: {
      // ⚠️：自己尝试了，这个写法会抱错，webpack版本 5.42.0
      ident: 'postcss',
      // 需要指定 process.env.NODE_ENV, 到 package.json 里面找 browserslist 对应的
      plugins: () => [require('postcss-preset-env')()],
    },
  },
];

module.exports = {
  mode: 'production',
  entry: './src/index.js',
  output: {
    // 对应主入口的输出文件名
    filename: 'js/[name].[hash:8].bundle.js',
    // 文件输出路径
    path: resolve(__dirname, 'build'),
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [...commonCssLoader],
      },
      {
        test: /\.less$/,
        use: [...commonCssLoader, 'less-loader'],
      },
      /* 
         对同一个文件，既要执行 eslint 又要执行 兼容性处理
         eslint 必须要先执行，可以设置 enforce: 'pre'
      */
      {
        // js 语法检查，配合 package.json 中的 eslintConfig 或者 .eslintrc.js 使用
        test: /\.js$/,
        loader: 'eslint-loader',
        exclude: /node_modules/,
        // 优先执行
        enforce: 'pre',
        options: {
          fix: true,
        },
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: {
          presets: [
            [
              // @babel/preset-env的polyfill使用usage形式（不了解的可以查看官方文档），意思是以项目设置的target环境为前提，根据项目中使用到的api功能进行polyfill
              '@babel/preset-env',
              {
                useBuiltIns: 'usage',
                corejs: {
                  version: '3',
                },
                targets: {
                  chrome: '60',
                  firefox: '50',
                  safari: '40',
                  ie: '9',
                  edge: '16',
                },
              },
            ],
          ],
        },
      },
      {
        test: /\.(png|jpe?g|gif)$/,
        loader: 'url-loader',
        options: {
          limit: 8 * 1024,
          // 输出路径
          outputPath: 'imgs',
          name: '[hash:10].[ext]',
          // 配合 html-loader 处理图片src的路径问题
          esModule: false,
        },
      },
      {
        // 处理 html 里图片src路径的问题
        test: /\.html$/,
        loader: 'html-loader',
      },
      {
        // 其他文件，原样复制
        exclude: /\.(png|jpe?g|gif|js|css|less|html)$/,
        loader: 'file-loader',
        options: {
          // 输出路径
          outputPath: 'media',
        },
      },
    ],
  },
  plugins: [
    // 将css抽取成单独的文件
    new MiniCssExtractPlugin({
      filename: 'css/[name].[hash:8].css',
    }),
    // css 压缩
    new OptimizeCssAssetsWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: './src/index.html',
      minify: {
        // 移除空格
        collapseWhitespace: true,
        // 移除注释
        removeComments: true,
      },
    }),
  ],
};
```
