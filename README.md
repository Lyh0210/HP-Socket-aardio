# HP-Socket-bindings-for-aardio
#### [HP-Socket](https://github.com/ldcsaa/HP-Socket) is a high performance network framework.
#### [aardio](http://www.aardio.com/)  is an extremely easy-to-use dynamic language, but it is also a hybrid language that allows for rare and very convenient manipulation of static types, so it can directly call API interface functions of static languages such as C, C++, etc.

### A simple example for ssl http client
***
````javascript
import win.ui;
/*DSG{{*/
var winform = win.form(text="aardio form";right=466;bottom=144)
winform.add(
btnBaidu={cls="button";text="访问 baidu (chunk 传输)";left=17;top=63;right=207;bottom=94;z=2};
btnCnbeta={cls="button";text="访问 cnbeta ";left=24;top=107;right=202;bottom=135;z=3};
edInfo={cls="edit";text="网页源码保存在同目录的  test.html";left=252;top=14;right=457;bottom=41;edge=1;multiline=1;readonly=1;z=1}
)
/*}}*/

io.open()

// 导入 ssl http client 监听器，用于 https 的访问
import aaz.libhpsocket.ssl.listener.httpClient;

// 创建监听器
var listener = aaz.libhpsocket.ssl.listener.httpClient();

// 定义事件的触发函数
// 说明：事件里面的函数为线程函数，请按规则使用

// 准备连接事件
listener.onPrepareConnect = function(component, connId, soListen){
	io.print("[onPrepareConnect]", connId, soListen)
}

// 完成连接事件
listener.onConnect = function(component, connId){
	io.print("[onConnect]", connId)
} 

// 握手完成事件
listener.onHandShake = function(component, connId){
	io.print("[onHandShake]", connId)
}

// 开始解析事件
listener.onMessageBegin = function(component, connId){
	io.print("[onMessageBegin]", component, connId)
	component.reallocString(1)
}

// 状态行解析完成 （仅用于 HTTP 客户端）
listener.onStatusLine = function(component, connId, usStatusCode, lpszDesc){
	io.print("[onStatusLine]", component, connId, usStatusCode,  lpszDesc )
}

// 请求头事件
listener.onHeader = function(component, connId, pszName, lpszValue){
	io.print("[onHeader]", component, connId, pszName, lpszValue)
}

// 请求头完成事件
listener.onHeadersComplete = function(component, connId){
	io.print("[onHeadersComplete]", component, connId)
	
	if(component.getHeader("Transfer-Encoding") == "chunked"){
		io.print("分块传输")
	}
	else {
		io.print("一次性传输->content-length", component.contentLength)
	}
		
	io.print("------获取全部 header-----------")
	var headers = component.getAllHeaders()
	for(i=1;#headers;1){
		io.print( headers[i].name, headers[i].value )
	}
	
	io.print("------获取指定 header (Set-Cookie)-----------")
	var headers = component.getHeaders("Set-Cookie");
	for(i=1;#headers;1){
		io.print(headers[i])
	}	
	
	io.print("------获取全部 cookie-----------")
	var cookies = component.getAllCookies()
	for(i=1;#cookies;1){
		io.print( cookies[i].name, cookies[i].value )
	}	
}

// Chunked 报文头事件
listener.onChunkHeader = function(component, connId, iLength){
	io.print("[onChunkHeader]", component, connId, iLength)
}

// Chunked 报文结束事件
listener.onChunkComplete = function(component, connId){
	io.print("[onChunkComplete]", component, connId)
}

// BODY 报文事件
listener.onBody = function(component, connId, pData, len){
	io.print("[onBody]", component, connId, pData, len)
	component.appendString(pData, len)
}

// 完成解析事件
listener.onMessageComplete = function(component, connId){
	io.print("[onMessageComplete]", component, connId)
	
	var html = component.getString();
	string.save("\test.html", html)
}

// 升级协议
listener.onUpgrade = function(component, connId, enUpgradeType){
	io.print("[onUpgrade]", component, connId, enUpgradeType)
}

// 解析错误
listener.onParseError = function(component, connId, iErrorCode, lpszErrorDesc){
	io.print("[onParseError]", component, connId, iErrorCode, lpszErrorDesc)
}

// Web Socket 数据包头
listener.onWsMessageHeader = function(component, connId, bFinal, iReserved, iOperationCode, lpszMask, ullBodyLen){
	io.print("[onWsMessageHeader]", component, connId, bFinal, iReserved, iOperationCode, lpszMask, ullBodyLen)
}

// Web Socket 数据包体
listener.onWsMessageBody = function(component, connId, pData, iLength){
	io.print("[onWsMessageBody]", component, connId, pData, iLength)
}

// Web Socket 数据包体
listener.onWsMessageComplete = function(component, connId){
	io.print("[onWsMessageComplete]", component, connId)
}

// 连接关闭事件
listener.onClose = function(component, connId){
	io.print("[onClose]", component, connId)
	component.reallocString(0)
}

// 创建组件，用于访问 baidu
var componentBaidu = listener.createComponent();
// 创建组件，用于访问 cnbeta
var componentCnbeta = listener.createComponent();

// 初始化 ssl 环境，并设置验证模式，默认不验证，即第一个参数为0
// 初始化成功返回 TRUE，失败返回 FALSE，初始化失败可通过 SYS_GetLastError() 获取错误代码
// 组件停止通信（调用 Stop()）时会自动清理 SSL 环境，因此，应用程序只需调用 SetupSSLContext() 初始化组件的 SSL 环境参数，而不需要手工调用 CleanupSSLContext 函数
componentBaidu.setupSSLContext(0)

// 向服务端应用程序发起连接请求，如果连接成功则返回 TRUE 并且会先后接收到 OnPrepareConnect、OnConnect 和 OnHandshake事件
// 断开连接时，客户端应用程序将收到 OnClose 事件
componentBaidu.start("www.baidu.com",443)

componentCnbeta.setupSSLContext(0)
componentCnbeta.start("www.cnbeta.com",443)

winform.btnBaidu.oncommand = function(id,event){	
    // 客户端应用程序调用 Send() 方法向服务端应用程序发出数据
	// 自己想办法在触发 onHandShake 事件后才运行
	componentBaidu.sendGet(
		"/",
    	{
    		["Accept"] = "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9";
    		["user-agent"] = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36";
    		["accept-language"] = "zh-CN,zh;q=0.9,ja;q=0.8,zh-TW;q=0.7";  
		}
	)	
}

winform.btnCnbeta.oncommand = function(id,event){	
	// 自己想办法在触发 onHandShake 事件后才运行
	componentCnbeta.sendGet(
		"/",
    	{
    		["user-agent"] = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36";
    		["accept-language"] = "zh-CN,zh;q=0.9,ja;q=0.8,zh-TW;q=0.7";  
		}
	)
}

winform.onClose = function(hwnd,message,wParam,lParam){
    // 客户端应用程序调用 Stop() 方法让组件停止通信，如果调用成功则返回 TRUE 并收到 OnClose 事件
    componentBaidu.stop()
	componentCnbeta.stop() 
}

winform.show();
win.loopMessage();
````
