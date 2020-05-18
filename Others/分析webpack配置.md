> 这是我一个项目的 webpack 配置，为了了解 webpack，我对其进行了逐行分析。

---

- 入口起点(entry points)
- 输出(output)
- 模式(mode) （development，production）
- loader（webpack 只能理解 JavaScript 和 JSON 文件。**loader** 让 webpack 能够去处理其他类型的文件）
- 模块解析(resolve ) （别名，alias）
- devServer   host
- 插件(plugins)
- 配置(configuration)
- 模块(module)

```javascript
//  路径模块提供用于处理文件和目录路径的实用程序
const path = require("path");
// 模块捆绑器
const webpack = require("webpack");
// 简单创建 HTML 文件，用于服务器访问
const HtmlWebpackPlugin = require("html-webpack-plugin");
// 此插件将CSS提取到单独的文件中。它为每个包含CSS的JS文件创建一个CSS文件。
const MinicssExtractPluin = require("mini-css-extract-plugin");
// const InlineChunkHtmlPlugin = require("react-dev-utils/InlineChunkHtmlPlugin");
// 用于优化CSS大小
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin");
// 缩小你的JavaScript
const TerserPlugin = require("terser-webpack-plugin");
// 用于删除/清除构建文件夹的webpack插件
const CleanWebpackPlugin = require("clean-webpack-plugin");
// 使用交互式可缩放树形图可视化webpack输出文件的大小。
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer")
  .BundleAnalyzerPlugin;
// 配置文件
const app = require("./app.json");
const cssRegex = /\.css$/;
const sassRegex = /\.(css|scss|sass)$/;
module.exports = () => {
  /*
        在node中，有全局变量process表示的是当前的node进程
        process.env包含着关于系统环境的信息
        process.env中并不存在NODE_ENV这个东西
        NODE_ENV是用户一个自定义的变量，在webpack中它的用途是判断生产环境或开发环境的依据的。
    */
  /*
        package.json
        "scripts": {
            "dev": "cross-env NODE_ENV=development webpack-dev-server  --config ./webpack.config.js --hot --color ",
            ...
        },
        cross-env 运行跨平台设置和使用环境变量的脚本
        当我们使用 NODE_ENV = production 来设置环境变量的时候，大多数windows命令会提示将会阻塞或者异常，或者，windows不支持NODE_ENV=development的这样的设置方式，会报错
     */
  const env = process.env.NODE_ENV || "production";
  const isProd = env === "production";
  const isDev = env === "development";
  const isTest = env === "test";
  // css的loaders
  const getCssloaders = function() {
    /*
            require.resolve(request[, options])
            request 需要解析的模块路径
            options 解析模块的起点路径。 没给意味着 node_modules 层级将从这里开始查询
         */
    /*
            Array.filter(Boolean) Array 类中的 filter 方法使用目的是移除所有的 ”false“ 类型元素
            Boolean(value)其值为 0、-0、null、false、NaN、undefined、或者空字符串（""），则生成的 Boolean 对象的值为 false
            传入的参数是 DOM 对象  document.all，也会生成值为 false
         */
    const loaders = [
      isDev && { loader: require.resolve("style-loader") },
      (isProd || isTest) && {
        loader: MinicssExtractPluin.loader,
        options: { publicPath: "../" } // 这里配置生成的css 文件里的静态资源的路径，这里是因为 最终生成在styles 文件下
      },
      {
        // 将 CSS 转化成 CommonJS 模块
        // importLoaders 在CSS加载器之前应用的加载器数量 0 => no loaders (default); 1 => postcss-loader; 2 => postcss-loader, sass-loader 这里的数量指的是当前loader之后loader的数量（“之后”说的是在表达中当前loader之后）。等于1是说，当前loader之后的一个loader。如果是n就是n个loader。
        // sourceMap 源码映射
        loader: require.resolve("css-loader"),
        options: {
          importLoaders: 1,
          sourceMap: !isProd
        }
      },
      {
        // PostCSS作用：1、把 CSS 解析成 JavaScript 可以操作的 AST。2、调用插件来处理 AST 并得到结果
        // 当独立使用postcss-loader（没有css-loader）时，不要在你的CSS中使用@import，因为这会导致相当臃肿的包
        /*
                    webpack ident在使用options时需要一个标识符（）{Function}/require（复杂选项）
                    autoprefixer 处理CSS前缀问题的神器
                    flexbox: 'no-2009' 将仅为最终版本和IE版本的规范添加前缀。
                 */
        loader: "postcss-loader",
        options: {
          ident: "postcss",
          plugins: () => [
            // https://github.com/csstools/postcss-preset-env
            require("postcss-preset-env")({
              autoprefixer: {
                flexbox: "no-2009"
              },
              stage: 3
            })
          ],
          sourceMap: !isProd
        }
      },
      app.csslang === "sass" && {
        // 将 Sass 编译成 CSS
        loader: "sass-loader",
        options: {
          sourceMap: !isProd
        }
      }
    ].filter(Boolean);
    return loaders;
  };
  // 可以配置的全局常量
  const getEnvStringifed = function() {
    const defs = {};
    // 三种环境development production test
    const e = app.env[env];
    for (const key in e) {
      if (e.hasOwnProperty(key)) {
        // 因为这个插件直接执行文本替换，给定的值必须包含字符串本身内的实际引号
        // eg:'"DEV"'或者 JSON.stringify("DEV")
        defs[key] = JSON.stringify(e[key]);
      }
    }
    return defs;
  };
  // 创建 HTML 文件
  const getHtmlPlugins = function() {
    /*
            "pages": [
                {
                    "page": "index",
                    "entry": "./src/pages/zhweb/index.js",
                    "template": "./src/pages/zhweb/index.html"
                },
                ...
            ],
         */
    return app.pages.map(page => {
      return new HtmlWebpackPlugin(
        /*
                    Object.assign(target,...sources)
                    将所有可枚举属性的值从一个或多个源对象复制到目标对象 不能深拷贝
                    inject: true javascript资源都将放置在body元素的底部。
                    filename 输出文件的文件名称(html文件目录是相对于webpackConfig.output.path路径而言的，不是相对于当前项目目录结构的 eg:现在是./build)
                    chunks 不配置此项默认会将entry中所有的thunk注入到模板中
                    minify 使用html压缩
                 */
        Object.assign(
          {},
          {
            inject: true,
            filename: `${page.page}.html`,
            template: page.template,
            chunks: ["runtime", "vendor", "commons", page.page]
          },
          isProd
            ? {
                minify: {
                  removeComments: true,
                  collapseWhitespace: true,
                  removeRedundantAttributes: true,
                  useShortDoctype: true,
                  removeEmptyAttributes: true,
                  removeStyleLinkTypeAttributes: true,
                  keepClosingSlash: true,
                  minifyJS: true,
                  minifyCSS: true,
                  minifyURLs: true
                }
              }
            : undefined
        )
      );
    });
  };
  // 入口
  const getEntrys = function() {
    return app.pages.reduce((a, b) => {
      a[b.page] = b.entry;
      return a;
    }, {});
  };
  return {
    // ref:https://webpack.docschina.org/concepts/mode/
    // 默认有三种环境 production  生产环境 development  开发环境  test  测试环境
    mode: isDev ? "development" : "production",
    // 入口文件，如果是多页 添加对应的chunkName 和入口文件
    entry: getEntrys(),
    /*
            filename 入口起点 此选项决定了每个输出 bundle 的名称
            chunkhash 使用基于每个 chunk 内容的 hash
            chunkFilename 此选项决定了非入口(non-entry) chunk 文件的名称
         */
    output: {
      // 代码生成到那个文件夹，如果是开发模式，代码在 内存里没有真实文件生成
      // 将一系列路径或路径段解析为绝对路径。
      path: !isDev ? path.resolve(__dirname, "./build") : undefined,
      // 入口文件对应的最终生成的文件的名称
      // 开发环境无需hash 值
      // 如果是多页 filename 即使开发环境也需要name去区分不同的chunk 否则 后面的会覆盖前面的
      filename: !isDev
        ? "scripts/[name].[chunkhash:8].js"
        : "js/[name].bundle.js",
      chunkFilename: !isDev
        ? "scripts/[name].[chunkhash:8].js"
        : "js/[name].bundle.js",
      // 资源托管到 CDN 时
      // 按需加载(on-demand-load)或加载外部资源(external resources)（如图片、文件等）
      // 相对于 HTML 页面（目录相同）
      publicPath: ""
    },
    // 测试环境打包对应的sourcemap  便于定位错误
    // 在大多数情况下,最佳选择是 cheap-module-eval-source-map。
    devtool: isDev
      ? "cheap-module-eval-source-map"
      : isTest
      ? "source-map"
      : false,
    // 配置路径别称，这里用@代表 src 目录
    // 对应的在js 中使用时直接当做src 即可 在 sass 内部使用时  需要使用 ~@
    // 如果需要增加别的别称 参考如下即可
    resolve: Object.assign(
      {},
      {
        alias: {
          "@": path.resolve(__dirname, "./src")
        }
      }
    ),
    module: {
      // 加载各类资源的规则
      rules: [
        {
          /*
                        test 匹配特定条件。一般是提供一个正则表达式或正则表达式的数组
                        enforce 指定 loader 种类。
                        use  每个入口(entry)指定使用一个 loader
                        include 匹配特定条件
                     */
          test: /\.(js|mjs|jsx)$/,
          enforce: "pre",
          use: [
            {
              options: {
                formatter: require.resolve("react-dev-utils/eslintFormatter"),
                eslintPath: require.resolve("eslint")
              },
              loader: require.resolve("eslint-loader")
            }
          ],
          include: path.resolve(__dirname, "./src")
        },
        {
          // 只使用第一个匹配的规则
          oneOf: [
            // css 资源处理
            {
              test: app.csslang === "sass" ? sassRegex : cssRegex,
              // loader 是从后往前执行的
              // 在开发模式下，css会通过js 插入到style节点中，在生产环境下，会单独打包出css 文件
              use: getCssloaders()
            },
            // 处理js 资源
            {
              test: app.isReactApp ? /\.(js|jsx)$/ : /\.js$/,
              // 不处理node_modules 中的文件
              // exclude: /node_modules/,
              // 只处理src 下自己的代码文件
              include: path.resolve(__dirname, "./src"),
              use: [
                {
                  loader: require.resolve("babel-loader"),
                  options: {
                    // 开启缓存，提高构件速度
                    cacheDirectory: true
                    // 使用babelrc 单独配置，具体配置看.babelrc 文件
                  }
                }
              ]
            },
            // 处理图片资源
            {
              test: /\.(png|jpg|jpeg|gif|bmp)$/,
              use: {
                loader: require.resolve("url-loader"),
                options: {
                  // url-loader大小低于limit的图片将被生成base64 编码的图片直接插入到css 文件中
                  limit: 10000,
                  name: isDev
                    ? "assets/[name].[ext]"
                    : "assets/[name].[hash:8].[ext]"
                }
              }
            },
            // file-loader 生成的文件的文件名就是文件内容的 MD5 哈希值并会保留所引用资源的原始扩展名
            {
              loader: require.resolve("file-loader"),
              exclude: [/\.(js|mjs|jsx|ts|tsx)$/, /\.html$/, /\.json$/],
              options: {
                name: isDev
                  ? "assets/[name].[ext]"
                  : "assets/[name].[hash:8].[ext]"
              }
            }
          ]
        }
      ].filter(Boolean)
    },
    plugins: [
      // 页面 每个页面(html)都需要配置一个HtmlWebpackPlugin ...扩展运算符
      ...getHtmlPlugins(),
      // doesnt work
      // isProd && new InlineChunkHtmlPlugin(HtmlWebpackPlugin, [/runtime/]),
      // 允许在编译时(compile time)配置的全局常量(eg:开发模式和生产模式的构建允许不同的行为)
      // 每个传进 DefinePlugin 的键值都是一个标志符或者多个用 . 连接起来的标志符(Webpack中设置环境变量为何要用JSON.stringify赋值)
      // 如果这个值是一个字符串，它会被当作一个代码片段来使用
      new webpack.DefinePlugin(getEnvStringifed()),
      /*
                从 bundle 中排除某些模块
                防止在 import 或 require 调用时，生成以下正则表达式匹配的模块
                moment.js  会引入体积很大的locale包
                v5版本写法
                new webpack.IgnorePlugin({
                    resourceRegExp: /^\.\/locale$/,
                    contextRegExp: /moment$/
                })
                resourceRegExp：匹配(test)资源请求路径的正则表达式。
                contextRegExp：（可选）匹配(test)资源上下文（目录）的正则表达式。
            */
      new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/),
      !isDev &&
        new MinicssExtractPluin({
          // Options similar to the same options in webpackOptions.output
          // both options are optional
          filename: "styles/[name].[contenthash:8].css",
          chunkFilename: "styles/[name].[contenthash:8].css"
        }),
      /*
                assetNameRegExp 一个正则表达式，指示应优化资源的名称
                cssProcessor 用于优化CSS的CSS处理器
                cssProcessorPluginOptions 传递给cssProcessor的插件选项
                canPrint：一个布尔值，指示插件是否可以向控制台打印消息
             */
      /*
                之前直接使用 minimize: true 在匹配到css后直接压缩
                遇到的问题：
                项目是用了autoprefix自动添加前缀，这样压缩，会导致添加的前缀丢失
                new OptimizeCSSAssetsPlugin({
                    assetNameRegExp: /\.css\.*(?!.*map)/g,  //注意不要写成 /\.css$/g
                    cssProcessor: require('cssnano'),
                    cssProcessorOptions: {
                        discardComments: { removeAll: true },
                        // 避免 cssnano 重新计算 z-index
                        safe: true,
                        // cssnano 集成了autoprefixer的功能
                        // 会使用到autoprefixer进行无关前缀的清理
                        // 关闭autoprefixer功能
                        // 使用postcss的autoprefixer功能
                        autoprefixer: false
                    },
                    canPrint: true
                }),
             */
      !isDev &&
        new OptimizeCSSAssetsPlugin({
          assetNameRegExp: /\.css$/g,
          cssProcessor: require("cssnano"),
          cssProcessorPluginOptions: {
            preset: ["default", { discardComments: { removeAll: true } }]
          },
          canPrint: true
        }),
      // 清除打包文件
      !isDev &&
        new CleanWebpackPlugin([path.join(__dirname, "./build")], {
          allowExternal: true
        }),
      // 打包性能
      !isDev && new BundleAnalyzerPlugin()
    ].filter(Boolean),
    /*
            使用 optimization.splitChunks 为页面间共享的应用程序代码创建 bundle
            minimize 告知 webpack 使用 TerserPlugin 压缩 bundle
            minimizer 允许你通过提供一个或多个定制过的 TerserPlugin 实例，覆盖默认压缩工具(minimizer)。
         */
    optimization: !isDev
      ? {
          minimize: true,
          minimizer: [
            new TerserPlugin({
              terserOptions: {
                warnings: false,
                compress: {
                  comparisons: false
                },
                parse: {},
                mangle: true,
                output: {
                  comments: false,
                  ascii_only: true
                }
              },
              parallel: true,
              cache: true,
              sourceMap: true
            })
          ],
          nodeEnv: "production",
          sideEffects: true,
          concatenateModules: true,
          // vendor中文翻译为厂商（第三方）
          // optimization.splitChunks 选项将 vendor 和 app(应用程序) 模块分开
          splitChunks: {
            chunks: "all",
            minSize: 30000,
            minChunks: 1,
            maxAsyncRequests: 5,
            maxInitialRequests: 3,
            name: true,
            cacheGroups: {
              vendor: {
                test: /[\\/]node_modules[\\/]/,
                chunks: "all",
                name: "vendor"
              },
              commons: {
                chunks: "all",
                minChunks: 2,
                reuseExistingChunk: true,
                enforce: true,
                name: "commons"
              }
            }
          },
          runtimeChunk: {
            name: "runtime"
          }
        }
      : {},
    /*
            devServer 能够用于快速开发应用程序
            host 指定使用一个 host。默认是 localhost。如果你希望服务器外部可访问 为'0.0.0.0'
            overlay 当出现编译器错误或警告时，在浏览器中显示全屏覆盖层。默认禁用
            disableHostCheck为true 此选项绕过主机检查。不建议这样做
            openPage 指定打开浏览器时的导航页面。
            useLocalIp 此选项允许浏览器使用本地 IP 打开。
         */
    devServer: isDev
      ? {
          host: "0.0.0.0", // 使用这个host 表示可以再除本机外的设备上访问
          overlay: true,
          disableHostCheck: true,
          open: true, // 浏览器自动打开,
          openPage: app.pages[0].page + ".html",
          useLocalIp: true,
          proxy: {
            "/proxy_api": {
              target: "http://*********/api",
              pathRewrite: { "^/proxy_api": "" }
            }
          }
        }
      : undefined
  };
};
```
