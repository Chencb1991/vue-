# vue笔记

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
 
