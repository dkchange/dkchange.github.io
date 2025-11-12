---
title: 使用postman
date: 2020-04-30 10:37:17
tags:
categories:
---

# 基本 api 功能

https://blog.csdn.net/qq_28284093/article/details/81031670

# 使用 postman 进行测试

https://segmentfault.com/a/1190000014144322

https://community.postman.com/t/running-a-request-multiple-times-with-different-data-sets/1064

https://github.com/epoberezkin/ajv

有关官多参数，有填，不填，null 个别的回传需要是什么要做验证，万一有 4 个参数以上交互起来就是 4 就要和每个参数的不同可能相乘
https://testerhome.com/topics/11744

发送异步请求
pm.sendRequest("https://postman-echo.com/get", function (err, response) {
console.log(response.json());

# postman 使用测试案例

因為 postman import file 只能够 同一个 collection 共用 data file
所以参考
https://blog.travelex.io/automation-with-postman-collection-runner-4a28eb975abf

过滤 data 的资料是否是指定给该请求
if(data.selectItemRequest!==pm.info.requestName){
postman.setNextRequest();
}
上面这样做不会成功，因為 setNextRequest()是在请求后呼叫的，所以写在 prerequest 并没有用，请求还是会被发送
https://postman-quick-reference-guide.readthedocs.io/en/latest/cheatsheet.html

pm.test.skip("Status code is 200", () => {
pm.response.to.have.status(200);
});

(skipTest ? pm.test.skip : pm.test)('should be valid', function () {
// do something here
});
结论是目前 postman 还没办法针对个别请求设定测试资料，还不能根据资料来略过请求，為了避免麻烦，只好不使用 csv 和 json 的受测资料

https://community.meraki.com/t5/Developers-APIs/Postman-run-multiple-request-with-different-variable-in-CSV-file/td-p/70234

# 测试注意事项

追踪 bug

# offline測試
因為每間公司的環境不同，有些公司是不允許聯外網的
 settings>>scratch pad
https://community.postman.com/t/working-in-offline-mode/20174/79