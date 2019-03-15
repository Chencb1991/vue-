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
 
