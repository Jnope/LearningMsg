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

# 函数组件

# 文件流
fs  
path  

# 执行命令
exec
spawn
