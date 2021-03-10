# Vue跨域访问



## 跨域

* 什么是跨域

    浏览器不允许当前页面的所在源去请求另一个源的数据。只要协议、端口或者域名中，有一个不相同就是跨域访问。

    通常情况下，在目前主流的前后端分离的环境下，前端项目和后端项目通常不在一台服务器下，并且所监听的端口也并不相同，就涉及到了跨域问题。



## 后端解决

SpringBoot项目可以在Controller上标注`@CrossOrigin`注解，允许其他端口访问接口，并不推荐使用



## 前端解决

如果使用Vue-cli脚手架搭建的项目，可以编写配置文件

> 注：Vue2会自动生成config.js文件，Vue3手动创建一个Vue.config.js文件

<img src="http://101.133.134.12:8079/Vue/%E8%B7%A8%E5%9F%9F-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.png" style="zoom:25%;" />

在配置文件中写入

``` js
module.exports = {
    devServer: {
        host: "localhost",
        port: 80, // 端口号
        https: false, // https:{type:Boolean}
        // open: true, //配置自动启动浏览器
        //配置跨域
        proxy: {
            '/api': {
                // 要代理的地址
                target: 'http://localhost:8080',
                ws:true,
                //允许跨域
                changeOrigin: true,
                // 如果是https接口的话，需要配置这个参数
                //重写接口
                pathRewrite: {
                    '^/api': ''
                },
            }
        },
    },
    publicPath: "./" // 解决项目打包后白屏的问题
}
```

此后项目中所有需要请求后端的接口全部写为`/api/接口地址`，例：

``` js
this.axios.get("/api/getArticle").then((res) => {
    console.log(res);
}).catch((err) => {
    console.log(err);
});
```





# Axios中添加请求头

* post请求

    ``` js
    this.axios.post('url', 参数, {
        headers: {
            '请求头': 'value',
        }
    }).then(res => {
        console.log(res);
    }).catch(err => {
        console.log(err);
    });
    ```

* get请求

    ``` js
    this.axios.get('url', {
       params: {
           参数
       },
       hearders: {
           'key': 'value',
       }
    }).then(res => {
        console.log(res);
    }).catch(err => {
        console.log(err);
    });
    ```

    