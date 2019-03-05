# vue笔记

#### 输入框输入暂停请求
```
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
