# CLI
## 设计组件
commander：定义命令  
inquirer/enquirer：定义交互  
chalk：输出颜色  
cli-table3：输出表格  
## 实现
### 定义命令
``` javascript
import { program } from 'commander';
program
    .version(getVersion()) // 可无
    .command('myCommand') // 可无command
    .description('My description')
    .option('-h, --help', "Print the generator's options and usage") // 定义选项
    .option('-i, --init [packageType]', 'Init')
    .action(() => { // 定义行为
        init();
    });
```
可以有多个program定义不同命令：program.opts<Type>() 获取用户输入的命令选项，Type属性为option的长选项
### 定义交互项
在action方法内：https://blog.csdn.net/huangpb123/article/details/121579663
``` javascript
import inquirer from 'inquirer';
const answer = await inquirer.createPromptModule.prompt<Type>({ // Type为返回值类型
    type: , // 提示的类型，input，number, confirm, list, rawlist, expand, checkbox, password, editor，默认input
    name: , // 问题在answer对象里的key
    message: , // 问题描述
    default: , // 默认值
    choices: [], // 问题选项
    filter: , // 用户回答的转换器、过滤器；为函数
    when: , // 根据之前回答判断是否显示; 函数
    validate：, // 校验
    transformer: , // 回答显示处理
    prefix: , // message前缀
    suffix: , // message后缀
    pageSize: , // 渲染行数
})
```
# UI组件
## 配置
``` javascript
// package.json内
    "files": [ // 定义组件包含文件
        "dist",
        "out"
    ],
    "main": "dist/index.js", // 组件入口
    "types": "out/src/index.d.ts", // type文件
```
## 组件库
每个UI组件定义组件tsx以及输出index.ts  

``` javscript
// index.ts
import '../../index.css';
export * from './MyUI';

// MyUI.tsx内为实际组件内容
```
src下index.ts导出所有组件
``` javscript
// index.ts
import './index.css';
export * from './path/MyUIA';
export * from './path/MyUIB';
```
## 打包
组件过多时，需要实现可引入单个组件，打包时做到去除其他未引入组件的能力  
webpack.base.js：打包配置  
webpack.all.js：打包入口./src/index.ts  
``` javascript
output: {
        path: path.join(__dirname, 'dist'),
        filename: 'index.js',
        libraryTarget: 'umd',
        library: 'common-ui',
        umdNamedDefine: true,
        globalObject: 'this',
    }
```
webpack.component.js：打包入口为各个组件的index.ts  
``` javascript
output: {
        path: path.join(__dirname, 'dist'),
        filename: '[name]/index.js',
        libraryTarget: 'umd',
        library: '[name]',
        umdNamedDefine: true,
        globalObject: 'this',
    }
```
## 使用
项目内引入babel-plugin-import，'@babel/preset-env', '@babel/preset-react'  
``` javascript
// package.json中sideEffects: false
// webpack对组件库做剪枝
optimization: {
        usedExports: true, // 表示开启使用tree shaking
    },
// rule中
{
                test: /\.(js|mjs|jsx)$/,
                include: [path.resolve(__dirname, 'src')],
                loader: 'babel-loader',
                options: {
                    babelrc: false,
                    configFile: false,
                    presets: ['@babel/preset-env', '@babel/preset-react'],
                    plugins: [
                        [
                            'import',
                            {
                                libraryName: '@cloud/my-ui',
                                libraryDirectory: 'dist',
                            },
                            '@cloud/my-ui',
                        ],
                    ],
                },
            }
```

# 函数组件

# 文件流
fs  
path  

# 执行命令
exec
spawn
