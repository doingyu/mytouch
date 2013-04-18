var fs = require('fs');

var filePath='../src/';
var fileList=["cache.js","device.js","event.js","info.js","tools.js","tpl.js","url.js","mytouch.js"];
var content=[];
for(var i=0;i<fileList.length;i++){
	content.push(fs.readFileSync(filePath+fileList[i]).toString());
}
var fc=content.join("\r\n");
fs.writeFile('../release/mytouch.js', fc, function (err) { if (err) throw err; console.log('mytouch 框架打包成功，文件位置： ../release/mytouch.js!'); });