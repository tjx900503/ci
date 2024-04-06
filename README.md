# 慈善家

使用方法详见脚本内注释，不再提供任何反馈渠道，慈善事业随缘更新~

主要接口验参调用函数模块，脚本内容全部加密无任何调用个人接口，我很忙没空研究如何偷你ck，喜欢🐕叫的别用谢谢

Tips：仓库内全部都是工具本没有常规本不需要默认设置定时任务，部分情况下运行脚本会报错例如不设置环境变量、变量无效等

## 拉库

- ### Arcadia 面板（推荐）

  ```bash
  arcadia repo 慈善家 https://gitlab.com/SuperManito/cishanjia.git main --updateTaskList true --autoDisable false --whiteList '^jd_'
  ```
  详见官方文档 [arcadia.cool](https://arcadia.cool)

- ### 其它

  建议使用 `git` 拉取本仓库

## 需要安装的依赖库

建议使用 Node.js® 20 LTS，最低需要 Node.js® 16

```bash
npm install -g axios got@11 https-proxy-agent ds crypto-js jsdom console-table-printer
```

## 功能配置

- ### 网络全局代理

  > 近期新增基于 Axios 的自研请求框架，全面支持请求代理自动化，脚本正在逐步适配中  
  > 暂不提供具体支持的脚本清单，建议以修改日期判断是否支持，如果你看不懂这部分内容那你就别用了~

  - #### 简述

    `JD_COMMON_REQUEST_` 前缀的为全局变量即生效于所有脚本，`<entryScriptName>_` 前缀的为独特变量即生效于特定脚本

    新请求框架代理配置引入了一个基于入口脚本文件名的独特变量配置体系，并且基于脚本的独特变量会优先于全局变量生效

    例如脚本文件名为 `a_example_test.js`，那么对应的独特静态代理变量名称为 `a_example_test_http_proxy`

  - #### 代理白名单

    > 不通过代理的hostname列表，多个用英文逗号分割，支持*通配符
    ```bash
    export JD_COMMON_REQUEST_NO_PROXY=""
    export <entryScriptName>_no_proxy=""
    ```
    > 例：`127.0.0.1,172.17.0.1,localhost,*.local`，多个用英文逗号分割  
    > 代理白名单是针对的实际业务请求而不是脚本名称

  - #### 静态代理

    ```bash
    export JD_COMMON_REQUEST_HTTP_PROXY=""
    export <entryScriptName>_http_proxy=""
    ```
    > 支持带有认证的地址格式，例如 `http://username:password@hostname:port`

  - #### 动态代理

    ```bash
    # 获取动态代理的接口地址
    export JD_COMMON_REQUEST_HTTP_DYNAMIC_PROXY_API=""
    export <entryScriptName>_http_dynamic_proxy_api=""
    # 指定每个动态代理地址的最多使用次数（整数），默认为1即每个代理仅使用1次，设置为0时表示不限次数
    export JD_COMMON_REQUEST_HTTP_DYNAMIC_PROXY_USE_LIMIT=""
    export <entryScriptName>_http_dynamic_proxy_use_limit=""
    # 指定每个动态代理地址的有效时长（单位毫秒），默认为10000即10秒，设置为0时表示永久有效
    export JD_COMMON_REQUEST_HTTP_DYNAMIC_PROXY_TIME_LIMIT=""
    export <entryScriptName>_http_dynamic_proxy_time_limit=""
    # 当获取动态代理失败时是否使用原生网络环境继续发起请求，填入 'true/false'，默认否
    export JD_COMMON_REQUEST_HTTP_DYNAMIC_PROXY_FETCH_FAIL_CONTINUE=""
    export <entryScriptName>_http_dynamic_proxy_fetch_fail_continue=""
    ```
    > 为了避免不必要的浪费建议将接口每次响应的代理地址数量设置为1个，另外建议将接口响应格式设置为单行文本的 `ip:port` 格式，同时也支持 `json` 格式不过仅适配了部分代理商  
    > 使用静态代理时会忽略动态代理配置

  - #### 额外提供的静态代理方式（过时的方法）

    > 此功能基于 `getToken` 模块实现，与上面描述的新请求框架无关  
    > 目前不建议使用此过时的功能，当上面提到的网络全局代理功能普及完毕后会移除该功能  
    > 若盲目使用该方式配置代理则可能会导致与上方的代理功能叠加，请针对特定脚本使用  
    > 该代理功能实现可能会导致脚本运行结束滞后，即脚本内容运行完毕后需要等待一些时间才会结束Node进程

    ```bash
    ## 启用代理
    export JD_ISV_GLOBAL_PROXY="true"
    ## 代理组件库相关控制变量
    # 定义 HTTP 代理地址（必填）
    export GLOBAL_AGENT_HTTP_PROXY="" # 例：http://127.0.0.1:8080
    # 过滤不需要代理的地址（必填）
    export GLOBAL_AGENT_NO_PROXY='127.0.0.1,172.17.0.1,*.telegram.org,oapi.dingtalk.com' # 用英文逗号分割多个地址，这里特别注意要把用到的内网ip过滤掉
    ```
    全局代理适用于本仓库绝大多数脚本，更多配置方法详见 [gajus/global-agent](https://github.com/gajus/global-agent)  
    需要额外安装代理依赖库才能使用 `npm install -g global-agent`
    > 如果你正在使用 Arcadia 面板则无需重复安装此代理依赖库，并且可以通过命令选项 `--agent` 在任意脚本上便捷的实现全局代理功能，具体详见配置文件和文档


- ### 自定义获取 `Token` 相关配置

  > `Token` 是关联账号的重要信息，它的有效期为30分钟左右因此不用每次都用新的，默认缓存在本地文件中，缓存时间为29分钟，同时也支持使用 `Redis` 数据库进行缓存以实现跨设备共用

  - #### 获取 `Token` 局部代理

    目前受限于官方接口策略，同一IP段请求多个账号后会频繁响应 `403`，因此可能需要配合代理使用，代理配置形式与上面提到的网络全局代理一致

    - ##### 静态代理

      ```bash
      export JD_ISV_TOKEN_HTTP_PROXY=""
      ```

    - ##### 动态代理

      ```bash
      # 获取动态代理的接口地址
      export JD_ISV_TOKEN_HTTP_DYNAMIC_PROXY_API=""
      # 指定每个动态代理地址的最多使用次数（整数），默认为1即每个代理仅使用1次，设置为0时表示不限次数
      export JD_ISV_TOKEN_HTTP_DYNAMIC_PROXY_USE_LIMIT=""
      # 指定每个动态代理地址的有效时长（单位毫秒），默认为10000即10秒，设置为0时表示永久有效
      export JD_ISV_TOKEN_HTTP_DYNAMIC_PROXY_TIME_LIMIT=""
      # 当获取动态代理失败时是否使用原生网络环境继续发起请求，填入 'true/false'，默认否
      export JD_ISV_TOKEN_HTTP_DYNAMIC_PROXY_FETCH_FAIL_CONTINUE=""
      ```
      > 为了避免不必要的浪费建议将接口每次响应的代理地址数量设置为1个，另外建议将接口响应格式设置为单行文本的 `ip:port` 格式，同时也支持 `json` 格式不过仅适配了部分代理商  
      > 使用静态代理时会忽略动态代理配置

  - #### 控制缓存时长

    ```bash
    export JD_ISV_TOKEN_CACHE_EXPIRE_MINUTES="" # 整数，默认不填为29分钟
    ```

  - #### 自定义本地缓存文件路径

    ```bash
    export JD_ISV_TOKEN_CUSTOM_CACHE="" # 绝对路径，建议以 token.json 命名
    ```
    > 此文件默认存储在仓库 `function/cache` 目录下，如果你需要多仓库使用建议定义此变量

  - #### 多场景互联（基于 `Redis` 数据库）

    > 通过数据库实现跨设备共用一套 `Token` 缓存，受限于脚本内容结构该功能可能没那么好用
    ```bash
    export JD_ISV_TOKEN_REDIS_CACHE_URL="" # 数据库地址，例：redis://password@127.0.0.1:6379/0
    export JD_ISV_TOKEN_REDIS_CACHE_KEY="" # 自定义提取或提交的键名规则，详见下方说明
    export JD_ISV_TOKEN_REDIS_CACHE_SUBMIT="" # 是否向数据库提交新的缓存token（true/false），默认是
    ```
    > 需要额外安装依赖库才能使用 `npm install -g redis`，默认从键名为用户名的字符串对象中提取键值，用户名是解码后的  
    > 如果你想自定义键名格式则需要将用户名位置设为 `<pt_pin>` 例如：`isv_token:<pt_pin>`，否则将自动在末尾追加

- ### 账号全局屏蔽

  > 此功能基于 `getToken` 模块实现

  ```bash
  export JD_ISV_TOKEN_LZKJ_PIN_FILTER=""
  export JD_ISV_TOKEN_LZKJ_NEW_PIN_FILTER=""
  export JD_ISV_TOKEN_CJHY_PIN_FILTER=""
  ```
  > 填入用户名，多个用@分割

- ### 自动登记实物奖品收货地址

  ```bash
  export WX_ADDRESS="" # 变量格式：收件人@手机号@省份@城市@区县@详细地址@6位行政区划代码@邮编，需按照顺序依次填写，多个用管道符分开（6位行政区划代码自己查地图，也可用身份证号前六位）
  export WX_ADDRESS_BLOCK="" # 黑名单关键词，多个关键词用@分开
  ```
  此变量是通用的，不过部分脚本具有与此功能相同的独特变量，届时将优先使用独特变量

- ### 自定义获取 APP 签名验参

  > 本仓库绝大部分脚本需要使用签名，默认通过请求 [杂货铺公益API](http://api.nolanstore.cc) 在线获取签名不自定义签名也能正常使用脚本

  - #### 请求 API 获取

    ```bash
    export JD_SIGN_API="" # 接口地址，例：http://127.0.0.1:3000/api/getSign
    export JD_SIGN_API_BODY_FIELD="" # body参数字段名，默认 'body'
    export JD_SIGN_API_FUNCTIONID_FIELD="" # functionId参数字段名，默认 'fn'
    export JD_SIGN_API_METHOD="" # 请求方法，默认 'POST'，自定义仅支持 'GET'
    export JD_SIGN_API_CONTENT_TYPE="" # 请求头 'Content-Type'，默认 'application/json; charset=utf-8'，支持 'application/x-www-form-urlencoded' 格式
    ```
    JSON响应格式解析的字段目前仅支持 `body` `convertUrl` `convertUrlNew`

  - #### 本地自定义脚本生成

    如果存在本地签名生成脚本则会优先加载本地签名，具体规范如下：
    - 1. 需要将脚本命名为 genSign.js 并存储在与 getSign 脚本同一目录下
    - 2. 调用函数名为 genSign 并且需要 export 导出
    - 3. 函数固定两个传参，分别是 functionId（函数id） 和 bodyParams（body参数对象）
    - 4. 函数需要返回含有 body、st、sign、sv 等关键字段的url参数形式的签名字符串

  不管通过何种途径获取签名，最终需要的签名形式为url参数格式且至少包含 `body` `st` `sv` `sign` 字段

- ### 自定义账号消息推送通知

  > 此功能基于 `sendJDNotify` 模块实现

  > 只对定义了推送通知开关独特环境变量的部分脚本有效，且默认均为不推送通知  
  > 账号消息的默认格式为 `【XX账号<账号序号>】<用户名>：<消息内容1>，<消息内容2>`

  - #### 过滤关键词

    ```bash
    export JD_NOTIFY_FILTER_KEYWORDS="" # 例："空气"，多个用@分割
    ```

  - #### 消息内容分隔符

    ```bash
    export JD_NOTIFY_DELIMITER="" # 例："、"，此分隔符用于分隔多条账号消息
    ```

  - #### 设置替换用户名为昵称

    ```bash
    export JD_NOTIFY_NICKNAMES="" # 例："userpin_α@哥哥,userpin_β@弟弟"，多个昵称配置用英文逗号分割，用户名和昵称用@分割
    ```

  - #### 是否显示用户名

    ```bash
    export JD_NOTIFY_SHOW_USERNAME="" # 例："false"，true/false，默认显示
    ```

  - #### 设置推送通知的用户名是否脱敏

    ```bash
    export JD_NOTIFY_USERNAME_DESENSITIZATION="" # 例："true"，true/false，默认不脱敏，根据用户名长度动态将部分字符用*替换
    ```

  - #### 设置消息前缀格式

    ```bash
    export JD_NOTIFY_PREFIX_FORMATA="" # 例："[账号%]"，%代表账号序号
    ```

  - #### 设置自动合并消息中用数字开头表示数量的内容

    ```bash
    export JD_NOTIFY_AUTO_MERGE_TYPE="" # 例："积分 🎟️"，多个规则用@分割，正则匹配
    ```

__未经授权请勿搬运，脚本仅供用于学习交流请勿推广，切勿用于商业用途！__
