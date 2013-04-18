//缓存控制模块
define('cache', function(require, exports, module) {
	module.exports = {
		/*
		 *	设置缓存 version,value,cacheTime
		 *	@param cacheTime 毫秒
		 */
		setStore: function(key, value, version, cacheTime) {
			try {
				if (!window.localStorage) {
					return this;
				}
			} catch (e) {
				return this;
			}
			var content = "";
			if (cacheTime) {
				var d = {};
				d.version = "localCachev1";
				d.dataVersion = version;
				d.cacheTime = ((new Date()).getTime() + (cacheTime ? parseInt(cacheTime, 10) : 0));
				d.content = value;
				content = JSON.stringify(d);
			} else {
				content = value;
			}
			try {
				localStorage.setItem(key, content);
			} catch (e) {
				return this;
			}
			return this;
		},
		/*
		 *	获取缓存
		 */
		getStore: function(key, version) {
			try {
				if (!window.localStorage) {
					return "";
				}
			} catch (e) {
				return "";
			}
			var content = localStorage.getItem(key);
			if (!content) {
				return "";
			}
			if (content.indexOf("localCachev1") >= 0) {
				var d = JSON.parse(content);
				//检查数据版本是否有效
				if (d.dataVersion != version) {
					return "";
				}
				//检查cache是否有效
				if (d.cacheTime >= (new Date()).getTime()) {
					return d.content;
				} else {
					this.removeStore(key);
					return "";
				}
			} else {
				return content;
			}
		},
		/*
		 *	删除缓存
		 */
		removeStore: function(key) {
			try {
				if (!window.localStorage) {
					return this;
				}
			} catch (e) {
				return this;
			}
			localStorage.removeItem(key);
			return this;
		},
		/**
		 * 设置页面离线缓存
		 */
		setManifest: function(fileName) {
			iframe = document.createElement("iframe");
			iframe.src = "/" + fileName ? fileName : "manifest.html";
			iframe.style.display = "none";
			setTimeout(function() {
				document.body.appendChild(iframe)
			}, 100);
			return this;
		}
	};
});
//设备相关调用
define('device',function(require,exports,module){
	module.exports = {
		/**
         * 返回屏幕宽度
         * @return {*}
         */
        getWidth:function() {
            return $(window).width();
        },
        /**
         * 返回屏幕高度
         * @return {*}
         */
        getHeight:function() {
            return $(window).height();
        },
        /**
         * 返回页面高度
         * @return {Number}
         */
        getPageHeight:function() {
            return $(document.body).height();
        },
        /**
         * 返回页面卷起的高度
         * @return {*}
         */
        getScrollHeight:function() {
            return document.body.scrollTop;
        },
        /**
         * 是否PC
         * @type {RegExp}
         */
        isPc:/(WindowsNT)|(Windows NT)|(Macintosh)/i.test(navigator.userAgent)
	};
});

/*
 *	事件管理模块，
 *	模块会以事件委托的形式，根据PC和mobile客户端实现事件差异化处理，以下为注意事项:
 *	1、模块支持的事件类型如下
 *	   tap(mobile touch实现的点击事件) 、click 、input、change、 sroll
 *	2、为实现事件委托，需要在body标签下创建 id="etDiv"(id为配置) 的div标签，所有的页面内容，应该
 *	   放在此div标签内容，考虑到以下原因，因此不在document下做事件委托：
 *	   
 *	   android系统(uc qq浏览器)
 *	   a、所有元素(包括document)，一定会响应click事件，此时document上的click事件代理有效
 *
 *     iso系统(safari chrome qq 浏览器)
 *     a.可点击元素(如a标签)，一定会响应click事件，此时document上的click事件代理有效
 *     b.不可点击元素，若父节点上（不包括body）没有绑定click事件，则document上的click事件代理失效。
 *       如果父节点上（不包括body）有绑定click事件，且touchend事件没有e.preventDefault();
 *       阻止浏览器行为，则document上的click事件代理有效。
 *  3、需要做事件处理的节点，需在节点内容增加一个自定义属性，属性名为pe(名称可配置)，pe的内容格式为：
 *     事件类型:xxxx  (xxxx为自定义信息，如:tap:myTap 或者 input:myInput,tap:myTap)
 *  4、事件默认会冒泡，直到绑定全局事件的节点。中途遇到pe属性的节点都会触发响应事件。如果事件名称以_开头，
 *     如：tap._myTap ，则阻止向上冒泡并出发事件 myTap(去掉了_)
 *	   
 */
define('event', function(require, exports, module) {
	var device = require('device'),
		inf = require('info');
	var g = {
		pc: device.isPc,//客服端是否PC
		coordinates : [],//在手机端记录touchend时坐标，以便取消该坐标25PX范围内的click事件响应

		//以下为配置信息
		etAttr : "pe",//节点中，记录事件信息的自定义属性名称
		dom : $("#etDiv"),//全局事件绑定的节点
		time : 0//用于生成事件处理函数的id
	};

	/**
	 * 绑定事件
	 * @param  {[string]}   type 自定义的事件名称，必须
	 * @param  {Function} fn   处理函数  必须
	 * @param  {[domelement]}   dom  用于透传的dom ，非必须
	 * @param  {[string]}   id   函数唯一标识，非必须
	 * @return {[null]}        无返回
	 */
	exports.on = function(type, fn, dom, id){
		inf.on(type, fn, dom, id);
	}
	exports.onMsg = exports.on;

	/**
	 * 触发事件
	 * @param  {[type]} type [自定义事件名称]
	 */
	exports.emit = function(type){
		return inf.emit.apply(this,arguments);
	}
	exports.postMsg = exports.emit;

	/**
	 * 解绑事件
	 * @param  {[string]} type 自定义的事件名称，必须
	 * @param  {[string]} id  函数唯一标识，非必须。
	 *                         如果没有id，则删除type下所有处理函数
	 *                         如果有id,则删除指定处理函数
	 * @return {[object]}      调用者
	 */
	exports.off = function(type, id) {
		inf.off(type, id);
		return this;
	}
	exports.removeMsg = exports.off;

	/**
	 * 解绑事件,非建议方法，不利于后续自动化扫描
	 * @param  {[string]} type 自定义的事件名称，正则表达式
	 * @return {[object]}      调用者
	 */
	exports.offReg = function(type) {
		inf.offReg(type);
		return this;
	}
	exports.removeMsgReg = exports.offReg;

	/**
	 * 事件绑定
	 * mobile使用zepto，PC使用jquery，两个库的on函数的第二个参数(样式选择器)存在差异，
	 * 
	 * zepto:
	 * 只有最初的元素符合selector时，才会调用响应函数。如果给定selector ,
	 * 则this是符合selector的元素节点，否则为绑定事件的元素节点
	 * (click时，把样式选择器的参数去掉，因为发现某些场景会导致click函数不处理，暂时没找到原因)
	 *
	 * jquery:
	 * 在冒泡过程中，只有节点符合selector，才会调用一次响应函数，有多少个符合，
	 * 调用多少次。如果没有给定selector,则，只有冒泡到绑定事件的节点才会调用响应函数
	 * @return {[type]} [无返回]
	 */
	exports.eventBind = function() {
		var dom = g.dom;
		if (!g.pc) { //非PC客户端，绑定touch事件
			dom.on("touchstart",touchstartFn)
				.on("touchmove",touchmoveFn)
				.on("touchend",touchendFn)
				.on("click",mClickFn)
				.on("input",eventFn("input"))
				.on("change",eventFn("change"));
		}else{
			dom.on("click",eventFn("click"))
				.on("keyup",eventFn("keyup"))
				.on("change",eventFn("change"));
		}
		$(window).on("scroll", scrollEvent);
	}

	
	/**
	 * scroll事件处理函数,暂时只处理滚动到底部
	 * 滚动到底部的自定义事件名称 ： scrollBot
	 * @return {[type]} [description]
	 */

	function scrollEvent(e) {
		var ph = device.getPageHeight(),
			sh = device.getScrollHeight(),
			wh = device.getHeight();

		//滚动到底部，且有响应处理方法
		if((sh+wh)>=ph){
			exports.emit("scrollBot",e,$(e.target),arguments.callee.toString()); //广播消息
		}
	}

	/**
	 * 统一事件处理函数
	 * @param  {[string]} type [事件类型]
	 * @return {[type]} [事件处理方法]
	 */

	function eventFn(type) {
		return function(e) {
			eventBubble(e,e.target,type);
		}
	}

	/**
	 * 事件冒泡处理
	 * @param  {[object]} e    [事件对象]
	 * @param  {[dom]} t    [dom节点]
	 * @param  {[string]} type [事件类型]
	 * @return {[type]}      [description]
	 */
	function eventBubble(e,t,type){
		//冒泡到顶部，则停止
		if(t==document.body || t==g.dom[0]){
			return;
		}
		var _t = $(t),
			et = _t.attr("et") || "",
			pe = _t.attr(g.etAttr) || "";
		//PC客户端，将tap事件转为click事件 ,input改成keyup
		if(g.pc && (/tap:|input:/.test(et)||/tap:|input:/.test(pe))){
			et = et.replace(/tap:/g,"click:").replace(/input:/g,"keyup:");
			pe = pe.replace(/tap:/g,"click:").replace(/input:/g,"keyup:");
		}
		if(type && et.indexOf(type + ":") != -1){//有对应处理方法
			if(type=="tap"){
				//响应了tap事件，则取消对应坐标的click事件
				preventGhostClick(g.x,g.y);
			}			
			var reg = new RegExp(type + ":([^,]*)(?:,|$)");
			exports.emit((et.match(reg)[1]||""),e,_t,arguments.callee.toString());
			return;
		}else if(type && pe.indexOf(type + ":") != -1){
			if(type=="tap"){
				//响应了tap事件，则取消对应坐标的click事件
				preventGhostClick(g.x,g.y);
			}			
			var reg = new RegExp(type + ":([^,]*)(?:,|$)"),
				fnName = pe.match(reg)[1]||"";//自定义事件名称
			exports.emit(fnName.replace(/^_/,""),e,_t,arguments.callee.toString());
			if(fnName.indexOf("_")!=0){//非 _ 开头，则冒泡
				arguments.callee(e,t.parentNode,type);
			}
		}else{//此节点不需要处理，继续冒泡
			arguments.callee(e,t.parentNode,type);
		}
	}

	/**
	 * touch事件开始
	 * @param  {[eventElement]} e [事件对象]
	 * @return {[type]}   [description]
	 */

	function touchstartFn(e) {
		/*
		if (e.target.nodeType == 3) { //如果是文本节点
			g.startTarget = $(e.target).parent();
		} else {
			g.startTarget = $(e.target);
		}*/
		//记录点击的位置
		var st = e.changedTouches[0];
		g.x = st.clientX;
		g.y = st.clientY;
	}

	/**
	 * touchmove事件处理
	 * @param  {[eventElement]} e [事件对象]
	 * @return {[type]}   [description]
	 */
	function touchmoveFn(e){
		var st = e.changedTouches[0],
			diffY = Math.abs(g.y - st.clientY),
			diffX = Math.abs(g.x - st.clientX);
		if (diffY<= 70 && diffX>10 && diffX>diffY && $(e.target).attr("scrollx")!="1") {//横向滑动，且sliderx!=1(没有设定可以横向滑动)
			//阻止浏览器横向滑动的前进后退行为
			e.preventDefault();
            e.stopPropagation();
            return false;
		}
	}

	/**
	 * [touchend事件]
	 * @param  {[eventElement]} e [事件对象]
	 */

	function touchendFn(e) {
		var st = e.changedTouches[0];
		if ((Math.abs(g.y - st.clientY) > 5) || (Math.abs(g.x - st.clientX) > 5)) {
			return;
		}
		eventFn("tap")(e);//手指点击事件
	}

	/**
	 * 手机端，click事件处理
	 * 若touchend已经处理了，则click事件阻止触发
	 * @param  {[type]} e [description]
	 * @return {[type]}   [description]
	 */
	function mClickFn(e){
		for (var i=0,iLen=g.coordinates.length;i<iLen;i+=2) {
		    var x = g.coordinates[i],
		    	y = g.coordinates[i + 1];
		    if (Math.abs(e.clientX - x) < 25 && Math.abs(e.clientY - y) < 25) {
		    	//响应click的坐标与记录的坐标相差25px
		    	//阻止click事件响应
		      e.stopPropagation();
		      e.preventDefault();
		      return false;
		    }
		}

		//以下代码表示此坐标没有touchend处理，则响应click事件
		eventFn("click")(e);
	}

	/**
	 * 阻止默认click事件响应
	 * @param  {[int]} x [touchstar时的X坐标]
	 * @param  {[int]} y [touchstar时的Y坐标]
	 */
	function preventGhostClick(x,y){
		g.coordinates.push(x, y);
  		setTimeout(function(){
  			g.coordinates.splice(0, 2);//删除前面的坐标
  		}, 2500);
	}
});

/**
 * 消息广播模块
 */
define('info', function(require, exports, module) {
	var g = {
		__Msgs: {}
	}
	/**
	 * 监听消息
	 * @param  {[string]}   type 自定义的消息名称，必须
	 * @param  {Function} fn   处理函数  必须
	 * @param  {[domelement]}   dom  用于透传的dom ，非必须
	 * @param  {[string]}   id   函数唯一标识，非必须
	 * @return {[null]}        无返回
	 */
	exports.on = function(type, fn, dom, id) {
		var __Msgs = g.__Msgs;
		__Msgs[type] = Object.prototype.toString.call(__Msgs[type]) == "[object Array]" ? __Msgs[type] : [];
		__Msgs[type].push({
			guid: id ? id : +new Date() + "" + g.time++,
			fn: fn,
			dom: dom
		})
	}
	/**
	 * 消息广播
	 * @param  {[string]} type 自定义的消息名称，必须
	 *                         除type外的其他参数会透传给处理函数
	 * @return {[object]}      必须符合规范 
	 *                         {
	 *                         		msgBack: mixed,//任意类型，作为postMsg的返回值
	 *                         		msgGoon :boolen,//true:执行后续事件处理函数，false:不执行
	 *                         		sendMsg:string//新的广播消息的名称，空串则不广播
	 *                            }
	 *                         如果没有按照规范返回，则此值直接作为postMsg的返回值，且后续事件处理函数不会执行
	 */
	exports.emit = function(type) {
		return function(center, args, queue, reValue, guid, o) {
			//纯粹调试用
			var debug = center["*"];
			if(debug){//所有消息都执行
				for (var i = 0, j = debug.length; i < j; i++) {
					debug[i].fn.apply(this, [type].concat(args));
				}
			}

			if (queue = center[type]) {
				var backData = {
						msgBack : null,//函数返回值
						msgGoon : true,//是否处理后续函数
						sendMsg : ""//新的广播消息名称
					};
				for (var i = 0, j = queue.length; i < j; i++) {
					o = queue[i];
					reValue = o.fn.apply(o.dom, args);
					if(Object.prototype.toString.call(reValue)=="[object Object]" && typeof(reValue)!="undefined"){//object类型
						backData.msgBack=reValue.msgBack;
						backData.msgGoon=reValue.msgGoon===false?false:true;
						backData.sendMsg = reValue.sendMsg;
					}else{
						backData.msgBack=reValue;
						backData.msgGoon = false;
						backData.sendMsg = "";
					}

					if(backData.sendMsg){//需要广播新消息
						exports.postMsg.apply(this,[backData.sendMsg].concat(args));
					}
					if(backData.msgGoon===false){//阻止后续事件处理
						break;
					}
				}
				return backData.msgBack;
			}
		}(g.__Msgs, Array.prototype.slice.call(arguments, 1))
	}


	/**
	 * 从队列中移除消息
	 * @param  {[string]} type 自定义的消息名称，必须
	 * @param  {[string]} id  函数唯一标识，非必须。
	 *                         如果没有id，则删除type下所有处理函数
	 *                         如果有id,则删除指定处理函数
	 * @return {[object]}      调用者
	 */
	exports.off = function(type, id) {
		var __Msgs = g.__Msgs;
		if (!id) { //没有id,则删除事件名称下所有处理函数
			delete __Msgs[type];
		} else {
			var _o = __Msgs[type] || [];
			for (var i in _o) {
				if (_o[i].guid == id) {
					_o.splice(i--, 1);
					break;
				}
			}
		}
		return this;
	}


	/**
	 * 从队列中移除消息,非建议方法，不利于后续自动化扫描
	 * @param  {[string]} type 自定义的事件名称，正则表达式
	 * @return {[object]}      调用者
	 */
	exports.offReg = function(type) {
		var __Msgs = g.__Msgs,
			reg = new RegExp(type);
		for (var i in __Msgs) {
			if (reg.test(i)) {
				delete __Msgs[i];
			}
		}
		return this;
	}
});
//相关代码调试工具
define('tools', function(require, exports, module) {
	module.exports = {
		/*
		 *	操作计时器
		 */
		timeMark: function(tag, type) {
			var t = "_timeMark";
			window[t] || (window[t] = {});
			if (type == "start") {
				window[t][tag] = {
					start: (new Date()).getTime(),
					end: 0,
					count: 0
				};
			} else if (type == "end") {
				if (!window[t][tag]) {
					return;
				}
				window[t][tag].end = (new Date()).getTime();
				window[t][tag].count = window[t][tag].end - window[t][tag].start;
			}
		},
		/*
		 *	计时结果 如果无此计时，返回null;
		 */
		getTimeMark: function(tag) {
			var t = "_timeMark";
			if (!window[t]) return null;
			var r = window[t][tag];
			return r ? r : null;
		}
	};
});
//模板渲染模块
define('tpl', function(require, exports, module) {
	/**
	 * 模板解析器txTpl:
	 * @author: wangfz
	 * @param {String}  模板id || 原始模板text
	 * @param {Object}  数据源json
	 * @param {String}  可选 要匹配的开始选择符 '<%' 、'[%' 、'<#' ..., 默认为'<%'
	 * @param {String}  可选 要匹配的结束选择符 '%>' 、'%]' 、'#>' ..., 默认为'%>'
	 * @param {Boolean} 可选 默认为true
	 * @return {String}
	 * 注意1: 输出"\"时, 要转义,用"\\"或者实体字符"&#92";
	 *　　　  输出"开始选择符"或"结束选择符"时, 至少其中一个字符要转成实体字符。
	 *　　　  html实体对照表：http://www.f2e.org/utils/html_entities.html
	 * 注意2: 模板拼接时用单引号。
	 * 注意3: 数据源尽量不要有太多的冗余数据。
	 */
	exports.tpl = (function() {
		var cache = {};
		return function(str, data, startSelector, endSelector, isCache) {
			var fn, d = data,
				valueArr = [],
				isCache = isCache != undefined ? isCache : true;
			if (isCache && cache[str]) {
				for (var i = 0, list = cache[str].propList, len = list.length; i < len; i++) {
					valueArr.push(d[list[i]]);
				}
				fn = cache[str].parsefn;
			} else {
				var propArr = [],
					formatTpl = (function(str, startSelector, endSelector) {
						if (!startSelector) {
							var startSelector = '<%';
						}
						if (!endSelector) {
							var endSelector = '%>';
						}
						var tpl = str.indexOf(startSelector) == -1 ? document.getElementById(str).innerHTML : str;
						return tpl.replace(/\\/g, "\\\\").replace(/[\r\t\n]/g, " ").split(startSelector).join("\t").replace(new RegExp("((^|" + endSelector + ")[^\t]*)'", "g"), "$1\r").replace(new RegExp("\t=(.*?)" + endSelector, "g"), "';\n s+=$1;\n s+='").split("\t").join("';\n").split(endSelector).join("\n s+='").split("\r").join("\\'");
					})(str, startSelector, endSelector);
				for (var p in d) {
					propArr.push(p);
					valueArr.push(d[p]);
				}
				fn = new Function(propArr, " var s='';\n s+='" + formatTpl + "';\n return s");
				isCache && (cache[str] = {
					parsefn: fn,
					propList: propArr
				});
			}

			try {
				return fn.apply(null, valueArr);
			} catch (e) {
				function globalEval(strScript) {
					var ua = navigator.userAgent.toLowerCase(),
						head = document.getElementsByTagName("head")[0],
						script = document.createElement("script");
					if (ua.indexOf('gecko') > -1 && ua.indexOf('khtml') == -1) {
						window['eval'].call(window, fnStr);
						return
					}
					script.innerHTML = strScript;
					head.appendChild(script);
					head.removeChild(script);
				}

				var fnName = 'txTpl' + new Date().getTime(),
					fnStr = 'var ' + fnName + '=' + fn.toString();
				globalEval(fnStr);
				window[fnName].apply(null, valueArr);
			}
		}
	})();
		/** doT模版引擎 调用方法：frame.template(html,data),返回内容为HTML字符串
	 * 虽然引擎使用了较多eval函数，但针对彩票简单的模版解释在手机端一般只需要1-2毫秒
	 * doT.js
	 * 2011, Laura Doktorova, https://github.com/olado/doT
	 * doT.js is an open source component of http://bebedo.com
	 * Licensed under the MIT license.
	 */
	"use strict";
	var doT = {
		version: '0.2.0',
		templateSettings: {
			evaluate:    /\{\{([\s\S]+?)\}\}/g,
			interpolate: /\{\{=([\s\S]+?)\}\}/g,
			encode:      /\{\{!([\s\S]+?)\}\}/g,
			use:         /\{\{#([\s\S]+?)\}\}/g,
			define:      /\{\{##\s*([\w\.$]+)\s*(\:|=)([\s\S]+?)#\}\}/g,
			conditional: /\{\{\?(\?)?\s*([\s\S]*?)\s*\}\}/g,
			iterate:     /\{\{~\s*(?:\}\}|([\s\S]+?)\s*\:\s*([\w$]+)\s*(?:\:\s*([\w$]+))?\s*\}\})/g,
			varname: 'it',
			strip: true,
			append: true,
			selfcontained: false
		},
		template: undefined, //fn, compile template
		compile:  undefined  //fn, for express
	};
	var global = {};

	function encodeHTMLSource() {
		var encodeHTMLRules = { "&": "&#38;", "<": "&#60;", ">": "&#62;", '"': '&#34;', "'": '&#39;', "/": '&#47;' },
			matchHTML = /&(?!#?\w+;)|<|>|"|'|\//g;
		return function(code) {
			return code ? code.toString().replace(matchHTML, function(m) {return encodeHTMLRules[m] || m;}) : code;
		};
	}

	var startend = {
		append: { start: "'+(",      end: ")+'",      startencode: "'+encodeHTML(" },
		split:  { start: "';out+=(", end: ");out+='", startencode: "';out+=encodeHTML("}
	}, skip = /$^/;

	function resolveDefs(c, block, def) {
		return ((typeof block === 'string') ? block : block.toString())
		.replace(c.define || skip, function(m, code, assign, value) {
			if (code.indexOf('def.') === 0) {
				code = code.substring(4);
			}
			if (!(code in def)) {
				if (assign === ':') {
					def[code]= value;
				} else {
					eval("def['"+code+"']=" + value);
				}
			}
			return '';
		})
		.replace(c.use || skip, function(m, code) {
			var v = eval(code);
			return v ? resolveDefs(c, v, def) : v;
		});
	}

	function unescape(code) {
		return code.replace(/\\('|\\)/g, "$1").replace(/[\r\t\n]/g, ' ');
	}

	doT.template = function(tmpl, c, def) {
		c = c || doT.templateSettings;
		var cse = c.append ? startend.append : startend.split, str, needhtmlencode, sid=0, indv;

		if (c.use || c.define) {
			var olddef = global.def; global.def = def || {}; // workaround minifiers
			str = resolveDefs(c, tmpl, global.def);
			global.def = olddef;
		} else str = tmpl;

		str = ("var out='" + (c.strip ? str.replace(/(^|\r|\n)\t* +| +\t*(\r|\n|$)/g,' ')
					.replace(/\r|\n|\t|\/\*[\s\S]*?\*\//g,''): str)
			.replace(/'|\\/g, '\\$&')
			.replace(c.interpolate || skip, function(m, code) {
				return cse.start + unescape(code) + cse.end;
			})
			.replace(c.encode || skip, function(m, code) {
				needhtmlencode = true;
				return cse.startencode + unescape(code) + cse.end;
			})
			.replace(c.conditional || skip, function(m, elsecase, code) {
				return elsecase ?
					(code ? "';}else if(" + unescape(code) + "){out+='" : "';}else{out+='") :
					(code ? "';if(" + unescape(code) + "){out+='" : "';}out+='");
			})
			.replace(c.iterate || skip, function(m, iterate, vname, iname) {
				if (!iterate) return "';} } out+='";
				sid+=1; indv=iname || "i"+sid; iterate=unescape(iterate);
				return "';var arr"+sid+"="+iterate+";if(arr"+sid+"){var "+vname+","+indv+"=-1,l"+sid+"=arr"+sid+".length-1;while("+indv+"<l"+sid+"){"
					+vname+"=arr"+sid+"["+indv+"+=1];out+='";
			})
			.replace(c.evaluate || skip, function(m, code) {
				return "';" + unescape(code) + "out+='";
			})
			+ "';return out;")
			.replace(/\n/g, '\\n').replace(/\t/g, '\\t').replace(/\r/g, '\\r')
			.replace(/(\s|;|}|^|{)out\+='';/g, '$1').replace(/\+''/g, '')
			.replace(/(\s|;|}|^|{)out\+=''\+/g,'$1out+=');

		if (needhtmlencode && c.selfcontained) {
			str = "var encodeHTML=(" + encodeHTMLSource.toString() + "());" + str;
		}
		try {
			return new Function(c.varname, str);
		} catch (e) {
			if (typeof console !== 'undefined') console.log("Could not create a template function: " + str);
			throw e;
		}
	};

	doT.compile = function(tmpl, def) {
		return doT.template(tmpl, null, def);
	};

	exports.compile = doT.compile;

	/*
	*	模版调用
	*/
	exports.template = function(html,data){
		return doT.template(html)(data);
	};

});
//url相关操作
define('url',function(require,exports,module){
	module.exports = {
		/**
		 * 设置hash
		 * @param name
		 */
		setHash:function(name) {
	        location.hash = name;
	    },
	    /**
         * 获取当前url中的hash值
         * @param url
         * @return String
         */
        getHash:function(url) {
            var u = url || location.hash;
            return u ? u.replace(/.*#/,"") : "";
        },
        /*
	    *	根据hash获取对应的模块名
	    */
	    getHashmoudelName:function(){
	    	var hash=this.getHash();
			return (hash?hash.split("&")[0].split("=")[0]:"");
	    },
	    /*
	    *	从hash中获取action
	    */
	    getHashActionName:function(){
	    	var hash=this.getHash();
	    	if(hash=="")return "";
			return (hash?hash.split("&"):[])[0].split("=")[1];
	    },
	    /*
		* 从hash中获取name对应的值
	    */
	    getHashParam:function(name){
	    	var result = this.getHash().match(new RegExp("(^|&)"+ name +"=([^&]*)(&|$)"));
			return result != null ? result[2] : "";
	    },
	    /*
	    *	从URL中获取参数对应的值
	    */
	    getUrlParam:function(name,url){
            //参数：变量名，url为空则表从当前页面的url中取
            var u  = arguments[1] || window.location.search,
                reg = new RegExp("(^|&)"+ name +"=([^&]*)(&|$)"),
                r = u.substr(u.indexOf("\?")+1).match(reg);
            return r!=null?r[2]:"";
        },
        /*
        *	获取所有HASH的参数，剔除moudel.
        */
        getParams:function(){
			var param = [],
				hash = this.getHash();
	    		paramArr = hash ? hash.split("&") : [];
	        for(var i= 1,l=paramArr.length;i<l;i++) {
	            param.push(paramArr[i]);
	        }
	        return param;
		},
		decodeUrl:function(url) {
            url = decodeURIComponent(url);
            var urlObj = this.parseUrl(url), decodedParam = [];
            $.each(urlObj.params, function(key, value) {
                value = decodeURIComponent(value);
                decodedParam.push(key+"="+value);
            });
            var urlPrefix = url.split("?")[0];
            return urlPrefix+"?"+decodedParam.join("&");
        },
        parseUrl:function(url) {
            var a =  document.createElement('a');
            a.href = url;
            return {
                source: url,
                protocol: a.protocol.replace(':',''),
                host: a.hostname,
                port: a.port,
                query: a.search,
                params: (function(){
                    var ret = {},
                        seg = a.search.replace(/^\?/,'').split('&'),
                        len = seg.length, i = 0, s;
                    for (;i<len;i++) {
                        if (!seg[i]) { continue; }
                        s = seg[i].split('=');
                        ret[s[0]] = s[1];
                    }
                    return ret;
                })(),
                file: (a.pathname.match(/([^\/?#]+)$/i) || [,''])[1],
                hash: a.hash.replace('#',''),
                path: a.pathname.replace(/^([^\/])/,'/$1'),
                relative: (a.href.match(/tps?:\/\/[^\/]+(.+)/) || [,''])[1],
                segments: a.pathname.replace(/^\//,'').split('/')
            };
        }
	};
});

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