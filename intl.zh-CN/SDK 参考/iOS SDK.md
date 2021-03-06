# iOS SDK {#reference_qyv_vh4_12b .reference}

阿里云日志服务 SDK 基于[概览](../../../../../intl.zh-CN/API 参考/概览.md)实现，目前提供写入日志的功能。

GitHub 地址：[https://github.com/aliyun/aliyun-log-ios-sdk](https://github.com/aliyun/aliyun-log-ios-sdk?spm=5176.doc43145.2.3.AKDn3Z)

## Swift {#section_rbp_sgj_sy .section}

```
 /*
    通过EndPoint、accessKeyID、accessKeySecret 构建日志服务客户端
    @endPoint: 服务访问入口，参见 https://www.alibabacloud.com/help/doc-detail/29008.htm
 */
let myClient = try! LOGClient(endPoint: "",
                              accessKeyID: "",
                              accessKeySecret: "",
                              projectName:"")
/* 创建logGroup */
let logGroup = try! LogGroup(topic: "mTopic",source: "mSource")
   /* 存入一条log */
    let log1 = Log()
          try! log1.PutContent("K11", value: "V11")
         try! log1.PutContent("K12", value: "V12")
         try! log1.PutContent("K13", value: "V13")
    logGroup.PutLog(log1)
   /* 存入一条log */
    let log2 = Log()
          try! log2.PutContent("K21", value: "V21")
         try! log2.PutContent("K22", value: "V22")
         try! log2.PutContent("K23", value: "V23")
    logGroup.PutLog(log2)
 /* 发送 log */
 myClient.PostLog(logGroup,logStoreName: ""){ response, error in
        // handle response however you want
        if error?.domain == NSURLErrorDomain && error?.code == NSURLErrorTimedOut {
            print("timed out") // note, `response` is likely `nil` if it timed out
        }
    }
```

## Objective-C {#section_qkw_tgj_sy .section}

参见 Github：[https://github.com/lujiajing1126/AliyunLogObjc](https://github.com/lujiajing1126/AliyunLogObjc)

