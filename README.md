# vue笔记

## vue_cli3 vue.config.js
```
module.exports ={
	baseUrl: process.env.NODE_ENV === "production" ? "./" : "/", //需方放置的服务器下级目录
	outputDir: "../www",//打包目录
	assetsDir: "./", //静态路径
	indexPath: "../www/index.html"//指定生成的 index.html 的输出路径,
	chainWebpack: config => {
    	config.module
      		.rule('images')
        	.use('url-loader')
         	 .loader('url-loader')
          	.tap(options => Object.assign(options, { limit: 1000240 }))
  	},//base64位图片生成限制大小
	devServer:{
		host: "http://47.98.155.157",//服务器测试环境
		proxy:{
		 '/api': {
      //           //target: 'http://xxx',
      //           target: 'http://xxx/portal_app',
      //           changeOrigin: true,
      //           ws: true,
      //           pathRewrite: {
      //             //'^/api': ''
      //             '^/api': '/'
      //           }
      //       }
		}
	}
}
```

### vue 路由拦截
```
const router = new Router({})
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requireAuth)){ // 判断该路由是否需要登录权限
       if (sessionStorage.getItem('loginUserId')) { // 判断当前的token是否存在
        next();
       }
       else {
          next({
          path: '/login',
          query: {redirect: to.fullPath} // 将跳转的路由path作为参数，登录成功后跳转到该路由
          })
       }
     }
     else {
       next();
     }

})
export default router
```

### vue简单搜索
```
<!DOCTYPE html>
<html>
<head>
	<title></title>
	<!-- 生产环境版本，优化了尺寸和速度 -->
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
<style type="text/css">
	ul{
		list-style: none;
		
	}
	ul li{
		width:40%;
		float: left;
	}
</style>
</head>
<body>
	<div id="app">
		<input v-model="key" type="text">
		<ul v-for="item in search">
			<li>{{item.names}}</li>
			<li>{{item.tel}}</li>
		</ul>
	</div>
</body>
<script>
	var vue = new Vue({
		el:'#app',
		data(){
			return{
				key:'',
				bodys:'',
				search:[]
			}
		},
		watch:{
			key:function(newValue,oldValue) {
				var that = this;
				
				if(newValue!=oldValue){
					var arrs=[];
					for(var i=0;i<that.bodys.length;i++){
						if(that.bodys[i].tel.indexOf(newValue)==0){
							arrs.push(that.bodys[i]);
							that.search=arrs;
							console.log(that.search);
						}
					}		
				}	
			}
		},
		mounted(){
			this.init();
			this.search=this.bodys
		},
		methods:{
			init(){
				this.bodys=[{'names':'熊**','tel':'15243623738'},
					{'names':'惠**','tel':'18569498484'},
					{'names':'老**','tel':'1396339597'},
					{'names':'电信安装','tel':'13348617628'},
					{'names':'左**','tel':'18390807665'},
					{'names':'周**','tel':'1828019409'}]
			}
		}
	})
</script>
</html>
```

### vue地址栏参数获取
```
console.log(this.$route.query)
```

### 路由传参
```
/loanreview/:id  //router.js路径

<router-link  :to="'/loanreview/'+items.Id"></router-link>
```
### 跳转参数
```
meta: {  
            aid:'',          
            requireAuth: true,  // 添加该字段，表示进入这个路由是需要登录的        
            }
	    
this.$router.push({name:'authercard',params:{aid:this.authercore.PeopleIdcardLiveState}})
console.log(this.$route.params.aid);	//authercard.vue    
```

### vue创建自定义js文件
```
export default {
  install (Vue, options) {
  	// 头部封装
    Vue.prototype.createHeader = function (loginUserId, requestType) {
        var header = new Object;
        header.requestType = requestType, 
        header.loginUserId = loginUserId,
        header.macNo = "XX-XX-XX-XX-XX-XX", 
        header.appVersionCode = "1",
        header.appLang = "1", 
        header.appChannel = "1",
        header.appDeviceType = "3",
        header.businessAppId="1"
        return header;
    },
  //服务器接口
    Vue.prototype.creathosturl = function () {
        var hosturl = new Object;
        hosturl.host=""
        //hosturl.host=""
        return hosturl;
    }
  }
 }
 
 
//main.js引入：
import utils from './vue.header'
Vue.use(utils);
```


#### 输入框输入暂停请求
```
 @keyup.native="onkeyup($event)" 

lastTimeStamp: 0,//标记当前事件函数的时间戳


onkeyup(event){
            if(event.keyCode != 13){//除回车键外
                //标记当前事件函数的时间戳
                this.lastTimeStamp = event.timeStamp;
                setTimeout(() => {
                    //1s后比较二者是否还相同（因为只要还有事件触发，lastTimeStamp就会被改写，不再是当前事件函数的时间戳）
                    if(this.lastTimeStamp == event.timeStamp){
                        this.calute()
                    }
                }, 800);
            }

        }
 ```
 
#### axios设置请求超时
```
import axios from 'axios'
axios.defaults.timeout = 6000
axios.interceptors.request.use(    
    config => {
        // 每次发送请求之前判断是否存在token，如果存在，则统一在http请求的header都加上token，不用每次请求都手动添加了
        // 即使本地存在token，也有可能token是过期的，所以在响应拦截器中要对返回状态进行判断
        // const token = store.state.token;        
        // token && (config.headers.Authorization = token);        
        //console.log(config);
        return config;    
    },    
    error => {     
    	console.log(error);   
        return Promise.error(error);    
    })
//响应请求
axios.interceptors.response.use(
	 response => {        
        if (response.status === 200) {   
        	console.log(response);         
            return Promise.resolve(response);        
        } else {            
        	console.log(response);    
            return Promise.reject(response);        
        }    
    },
     error => {
     	Toast('网络请求超时');
     	router.go(-1);//返回
     	 // router.replace({                        
       //                  path: '/',                        
       //                  query: { redirect: router.currentRoute.fullPath } 
       //              });
    	return Promise.reject(error)
     }
	)
 
 Vue.prototype.$axios = axios
 
 ```
 

```
inits(){
      this.$axios.get('/api', {
　　    params: { type: 'QHCC',
          sty: 'QHSYCC',
          stat: '3',
          fd: '2019-04-22',
          mkt: '069001008',
          code: 'ma1909',
          sc: 'MA'}
        })
      .then(function (response) {
         console.log(response.data.slice(1).substring(0,response.data.length-2));
         let datass = response.data.slice(1).substring(0,response.data.length-2);
         console.log(JSON.parse(datass))

      })
      .catch(function (error) {
      console.log(error);
      });
    },
    ```
    
    ```
    init(){
      this.$axios.get('/api', {
　　    params: { type: 'QHCC',
          sty: 'QHSYCC',
          stat: '3',
          fd: '2019-04-22',
          mkt: '069001008',
          code: 'ma1909',
          sc: 'MA'}
        })
      .then(function (response) {
         //console.log(response.data.slice(1).substring(0,response.data.length-2));
         let datass = response.data.slice(1).substring(0,response.data.length-2);
         console.log(JSON.parse(datass));
         let datasmore = JSON.parse(datass)[0].净多头龙虎榜;
         let datasless = JSON.parse(datass)[0].净空头龙虎榜;
         let datasempty = JSON.parse(datass)[0].空头增仓龙虎榜;
         let datasemptyL = JSON.parse(datass)[0].空头减仓龙虎榜;
         let datasover = JSON.parse(datass)[0].多头增仓龙虎榜;
         let datasoverL = JSON.parse(datass)[0].多头减仓龙虎榜;
         /////////////////////////////////////////////
         let datasmorecol = [];//净多头龙虎榜
         let dataslesscol = [];//净空头龙虎榜
         let arrempty = []; //空头增仓龙虎榜
         let arremptyL = []; //空头减仓龙虎榜
         let arreover = []; //多头增仓龙虎榜
         let arreoverL = []; //多头减仓龙虎榜
         /////////////////////////////////////////////
         for(var i=0;i<datasmore.length;i++){
            let datasmorearr = datasmore[i].split(",");
             datasmorecol.push({'id':datasmorearr[0],'name':datasmorearr[1]})     
         };
         for(var i=0;i<datasless.length;i++){
            let dataslessarr = datasless[i].split(",");
             dataslesscol.push({'id':dataslessarr[0],'name':dataslessarr[1]})     
         };

         for(var i=0;i<datasempty.length;i++){
            let datasemptyarr = datasempty[i].split(",");
             arrempty.push({'id':datasemptyarr[0],'name':datasemptyarr[1]})     
         };

         for(var i=0;i<datasemptyL.length;i++){
            let datasemptyarrL = datasemptyL[i].split(",");
             arremptyL.push({'id':datasemptyarrL[0],'name':datasemptyarrL[1]})     
         };

         for(var i=0;i<datasover.length;i++){
            let datasoverarr = datasover[i].split(",");
             arreover.push({'id':datasoverarr[0],'name':datasoverarr[1]})     
         };

         for(var i=0;i<datasoverL.length;i++){
            let datasoverarrL = datasoverL[i].split(",");
             arreoverL.push({'id':datasoverarrL[0],'name':datasoverarrL[1]})     
         };

         console.log(datasmorecol); 
         console.log(dataslesscol); 
         console.log(arrempty); 
         console.log(arreover); 
         console.log(arremptyL); 
         console.log(arreoverL); 
      })
      .catch(function (error) {
      console.log(error);
      });
      },
     ```
     
     
     
    #### 监听路由动态变化，改变浏览器倒退tab值变化  
     
     ```
   
  watch: {
    $route: {
      handler: function(val, oldVal){
        //console.log(val.name);
        if(val.name=='mains'){
          sessionStorage.setItem('actives',this.tabs[0]);

        }
        if(val.name=='loan'){
          sessionStorage.setItem('actives',this.tabs[1]);

        }
        if(val.name=='user'){
          sessionStorage.setItem('actives',this.tabs[2]);
          //this.init()
        }
      },
      // 深度观察监听
      deep: true
    }
  }
  ```
  
  
  #### vue上传项目到服务器后解决https无法访问的问题
  
  ```
  //app.js文件

//1.引入express模块
const express = require('express');
const http = require('http');
const https = require('https');

const note = require('./router/api')
const mongoose = require("mongoose");

const bodyParser = require("body-parser")


//2.创建app对象
const app = express()
//定义简单路由
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

app.use('/api',note);

var httpServer = http.createServer(app);
var httpsServer = https.createServer(app);

httpServer.listen(3003, function() {
    console.log('HTTP Server is running on: http://localhost:%s', 3003);
});
httpsServer.listen(4430, function() {
    console.log('HTTPS Server is running on: https://localhost:%s', 4430);
});
config-index.js

proxyTable: {
         '/api': {
          //target: 'http://121.39.66.757/3003',
          target: 'http://localhost:3003',
          changeOrigin: true,
           pathRewrite: {
                    '^/api': '/api',//重写,
                }
        }
    },
    ```
    
    
    #### vue创建自定义组件
    
    ```
 1.组件目录创建loadin文件夹，文件夹内创建Loading.vue(略)和index.js
index.js安装组件导出组件：

const LoadingComponent = require('./Loading.vue')
const loading = {
  install: function(Vue) {
    Vue.component('loading', LoadingComponent)
  }
}
module.exports = loading
2.全局main.js引入
import Loading from './components/Loading'
Vue.use(Loading);
3.组件引入页面
<loading v-if="loading"></loading>
4.当然也可在个别组件单独引入该组件。
```
    
    
#### nginx配置vue项目部署访问刷新页面出现404问题

```
nginx服务器,路由子页面刷新404/,首页无问题，因为除了首页index.html其他路由页面不存在。

解决思路
只需要当服务器发现客户端发来的url时就重定向（服务器重定向）到默认的index.html文件内容并返回给客户端，这样就实现了它的自身的路由功能。

配置方法
server {
         listen 80;

         server_name testwx.wangshibo.com;
         root /mnt/app/xm_web;
         index index.html;
        # access_log /var/log/testwx.log main;

        #处理vue-router路径Start
        #如果找不到路径则跳转到@router变量中寻找,找到了就默认进入index.html
        location / {
             try_files $uri $uri/ /index.html last;
             index index.html;
         }
        #处理vue-router路径End

}
==try_files $uri $uri/ /index.html last;==

配置完成重新启动服务器，刷新即可。

```


#### win10系统安装mongodb非关系型数据库基本操作

```
下载地址
https://www.mongodb.com/download-center/community
下载到电脑放置到C盘解压到（创建mongodb文件夹）mongodb文件夹下:如下
C:\mongodb
解压后复制bin目录地址添加到系统电脑系统变量中：
电脑-属性-高级系统设置-环境变量-高级-系统变量 -path-添加

C:\mongodb\bin
mongodb目录创建data/db 数据路径文件
C:\mongodb\data\db
mongodb目录创建log/mongodb.log 日记路径文件
C:\mongodb\log\mongodb.log
进入bin目录执行
mongod --dbpath C:\mongodb\data\db --logpath C:\mongodb\log\mongodb.log --27017
进入data目录，依次执行mongod,mongo启动
完毕。

```

#### centos 系统服务器安装mysql-server

```
# wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm

# rpm -ivh mysql-community-release-el7-5.noarch.rpm

# yum install mysql-community-server
安装成功后重启mysql服务：

# service mysqld restart
初次安装mysql，root账户没有密码。

# mysql -u root


设置密码：

mysql> set password for 'root'@'localhost' =password('password');
Query OK, 0 rows affected (0.00 sec)
mysql>

```


#### centos系统服务器安装nginx步骤

```
阿里云centos服务器

首先需要安装gcc

yum install gcc-c++
PCRE pcre-devel 安装

yum install -y pcre pcre-devel
zlib 安装

yum install -y zlib zlib-devel
OpenSSL 安装
yum install -y openssl openssl-devel
从官网下载nginx,使用tar命令解压

tar -zxvf nginx-1.10.1.tar.gz
解压以后执行

./configure && make && make install
启动、停止nginx

cd /usr/local/nginx/sbin/
./nginx
./nginx -s stop
./nginx -s quit
./nginx -s reload

```


#### centos系统服务器安装php7.2

```
阿里云服务器系统安装PHP

首先获取rpm：
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
然后可以利用 sudo yum list php*查看目前都有php的什么版本了，可以发现从4-7.2的版本都有，7.2版本名为72w，因此安装该版本即可：

sudo yum -y install php72w
但安装完毕后，输入php -v发现并没有该命令，因为php72w只是安装了php最小的库，一些应用还未安装，因此安装一些拓展包即可：

yum -y install php72w-cli php72w-common php72w-devel php72w-mysql
然后输入php -v

```


#### centos服务器安装forever让nodejs服务在后台永久运行

```
当用户断开客户链接时，nodejs的应用也随之停止。而使用forever就可以让nodejs的应用像服务一样在后台继续运行
全局安装
npm install forever -g   #安装
启动应用
forever start app.js     #启动应用
其他指令
forever stop app.js      #关闭应用
forever restart app.js   #重启应用
forever stopall          #关闭所有应用
forever restartall       #重启所有应用
forever list             #显示所有运行的应用

```

#### nginx代理node3000端口

```
server {
  listen 80;
  server_name nodevue.zhangwenzong.cn;
  access_log  /home/zhangwenzong/pb.haolinks.com_nginx.log combined;
  index index.html index.htm index.php;
  #include /usr/local/nginx/conf/rewrite/none.conf;
  root /data/wwwroot/postback;

  location / {
      proxy_set_header X-Real-IP  $remote_addr;
      proxy_pass http://127.0.0.1:3000/;
    }
  location /users/ {
      proxy_pass http://127.0.0.1:3000/users;
    }
  location /cart/ {
      proxy_pass http://127.0.0.1:3000/cart;
    }
  location /goods/* {
      proxy_pass http://127.0.0.1:3000/goods;
    }
}
```




