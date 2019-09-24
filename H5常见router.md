# H5常见router

## 页面Router

主要通过操作页面来改变页面路由。

window.location.href = “www.baidu.com”

history.back();

## Hash Router

主要通过改变页面路由hash值，来改变页面路由。

window.location.href=''#test1"

window.location.href=''#test2"

window.onhashchange= function(){console.log("current hash:",window.location.hash)}

## H5 Router

history.pushState('name','title','/path');//往栈里面推路由

history.pushState('test','title','#test');

history.pushState('test','title','/user/login');

window.onpopstate = function(e){

    console.log('h5 router change',e.state);
    
    console.log(window.location.href);//全路径
    
    console.log(window.location.pathname);//绝对路由
    
    console.log(window.location.hash);//hash值
    
    console.log(window.location.search);//search ?之后的参数

}//路由监听

history.pushState('test','title','/user/test');

history.replaceState('name',null,'/path')//替换当前页面路由

## React Router

<Route></Route>//路由规则

<Switch></Switch>//路由选项 自动匹配，匹配到第一个就不再继续匹配

<Link></Link>//跳转导航

<Redirect>//自动跳转
