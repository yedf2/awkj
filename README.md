awkj --node版本的awk
====

*   使用js，拥有强大的编程能力，无需记忆awk特有语法
*   高效运行，在一个百万行日志处理的测试中，用时仅比awk多30%

###安装

```sh
  sudo npm -g install awkj  #需要node6+ 环境
```

###使用

```sh
  echo -e "1  3\n2 5" | awkj 'console.log($2)'
```

*  可使用的预定义变量有：FS NR $n (含义与awk相同) G（用于全局数据的存储）
*  END 含义与awk相同

```sh
  echo -e "1  3\n2 5" | awkj 'if(NR==1) {G.sum=0;} G.sum+=parseFloat($2); END console.log(G.sum/NR)'
```

###实现

核心代码如下：
```javascript
  let G={NR:0}; // 全局数据
  let [body, end] = cont.split('END'); //用户输入的代码分为两部分，前面的body每行执行，END后面的代码结束时执行
  eachLine(line=>{
    G.NR ++; // G.NR为行号
    let fs = line.split(sep); // 把行切分为字段
    let obj = {FS:fs.length, $0:line, G, NR:G.NR}; // 构建实参
    fs.forEach((e,i)=>obj['$'+(i+1)] = e); //构建$1...$n
    //函数内部需要解构obj，并且解构$1...$n，最多解构到$99
    G.func = G.func || eval(`obj=>{ 
            let {${Object.keys(obj).join(',')}, ${new Array(100-fs.length).join('0').split('').map((e,k)=>`$${k+1+fs.length}`).join(',')}} = obj;
            ${body}
        }`);
    G.func(obj); //根据用户输入构建函数并调用
  }, ()=>{ //文件结束的回调
    end && eval(`obj=>{let {NR,G}=obj;${end}}`)({NR:G.NR,G}); //构建尾部函数并调用
  });
```
