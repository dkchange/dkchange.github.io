---
title: 使用postman
date: 2020-04-30 10:37:17
tags:
categories:
---

# 基本 API 功能

[CSDN 教學](https://blog.csdn.net/qq_28284093/article/details/81031670)

# 使用 Postman 進行測試

[SegmentFault 教學](https://segmentfault.com/a/1190000014144322)
[Postman Community 討論](https://community.postman.com/t/running-a-request-multiple-times-with-different-data-sets/1064)
[ajv GitHub](https://github.com/epoberezkin/ajv)

有關多參數，有填、不填、null 個別的回傳需要是什麼要做驗證，萬一有 4 個參數以上交互起來就是 4 就要和每個參數的不同可能相乘
[TesterHome 討論](https://testerhome.com/topics/11744)

發送非同步請求

```javascript
pm.sendRequest("https://postman-echo.com/get", function (err, response) {
  console.log(response.json())
})
```

# Postman 使用測試案例

因為 Postman import file 只能夠同一個 Collection 共用 Data File
所以參考
[Travelex.io 教學](https://blog.travelex.io/automation-with-postman-collection-runner-4a28eb975abf)

過濾 Data 的資料是否是指定給該請求

```javascript
if (data.selectItemRequest !== pm.info.requestName) {
  postman.setNextRequest()
}
```

上面這樣做不會成功，因為 setNextRequest() 是在請求後呼叫的，所以寫在 prerequest 並沒有用，請求還是會被發送
[Postman Quick Reference Guide](https://postman-quick-reference-guide.readthedocs.io/en/latest/cheatsheet.html)

```javascript
pm.test.skip("Status code is 200", () => {
  pm.response.to.have.status(200)
})

;(skipTest ? pm.test.skip : pm.test)("should be valid", function () {
  // do something here
})
```

結論是目前 Postman 還沒辦法針對個別請求設定測試資料，還不能根據資料來略過請求，為了避免麻煩，只好不使用 CSV 和 JSON 的受測資料
[Meraki Community 討論](https://community.meraki.com/t5/Developers-APIs/Postman-run-multiple-request-with-different-variable-in-CSV-file/td-p/70234)

# 測試注意事項

追蹤 Bug

# Offline 測試

因為每間公司的環境不同，有些公司是不允許聯外網的
Settings >> Scratch Pad
[Postman Community 討論](https://community.postman.com/t/working-in-offline-mode/20174/79)
