define('mytouch', function(require, exports, module) {
	//业务框架的moudel名及初始化action的名字，业务框架是指业务逻辑进行全局操作时的公共服务，一定会被调用
	var frameMoudle = "frame";
	var frameAction = "init";
	//首页模块的moudel名及默认action名字，首页主模块是指主框架初始化完成后首先调用的默认业务模块的名字
	var defaultMoudle = "main";
	var defaultAction = "index";
	/**
	 * 以上配置内容可以自己在代码外额外定义两个变量进行重写
	 * mytouch_frame="frame/init";
	 * mytouch_main="main/index";
	 */
	//读取个性化配置
	if(window["mytouch_frame"]){
		frameMoudle=mytouch_frame.split("/")[0];
		frameAction=mytouch_frame.split("/")[1];
	}
	if(window["mytouch_main"]){
		defaultMoudle=mytouch_main.split("/")[0];
		defaultAction=mytouch_main.split("/")[1];
	}

	//mytouch框架的初始化入口
	var url = require("url");
	var info = require("info");

	var lastMoudle = ""; //记录上一次访问的moudle
	var lastAction = ""; //记录上一次访问的Action
	exports.checkDestroy = checkDestroy;
	//初始化业务框架模块，如果没有主模块的话则保持变量frameMoudle、frameAction为空
	initFrameMoudle();
	//HASH改变事件
	$(window).on("hashchange", function(e) {
		hashRoute();
	});
	//初始化hash路由模块，hash规则：#后以key1=value1&key2=value2的格式表示，第一个key表示要调用的moudel，第一个value表示要调用的action
	hashRoute();

	//初始化业务框架模块 once

	function initFrameMoudle() {
		if (!frameMoudle || !frameAction) {
			return false;
		}
		require.async(frameMoudle, function(m) {
			if (m && typeof m[frameAction] == 'function') {
				m[frameAction]();
			} else {
				return false
			}
		});
	}
	//初始化hash路由模块

	function hashRoute() {
		//没有设置默认hash的时候使用默认的moudel、action
		var moudel = (url.getHashmoudelName() || defaultMoudle);
		var action = (url.getHashActionName() || defaultAction);

		//广播新的模块被载入
		info.emit("new_moudel_action", moudel, action);
		//异步加载要执行的moudel
		require.async(moudel, function(m) {
			if (m && typeof m[action] == 'function') {
				//调用上一个action的析构方法（可选）
				checkDestroy();
				//先判断目标action是否有构造方法，有则执行，如果构造方法返回false，则不执行目标action
				if (m[action + "_construct"]) {
					(m[action + "_construct"]() !== false) && (m[action]());
				} else {
					m[action]();
				}
				//记录action场景
				lastMoudle = m;
				lastAction = action;
				//广播moudle执行完成的事件
				info.emit("moudel_run_success", moudel, action);
			} else {
				//广播moudle加载错误的事件
				info.emit("moudle_load_error", moudel);
				//没有加载到module，什么都不做
				return false;
			}

		});
	}
	//执行析构方法,既然进入析构逻辑，就说明这个moudle和action已经被加载进来了，这里就不再做模块加载的动作

	function checkDestroy() {
		//调用上一个action的析构方法
		if (lastMoudle && typeof(lastMoudle[lastAction + "_destroy"]) == "function") {
			return lastMoudle[lastAction + "_destroy"]();
		} else {
			return "";
		}
	}

});


//框架在载入后自动启动入口init.
seajs.use('mytouch');