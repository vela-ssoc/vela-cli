# vela-cli
> vela快速生成项目接结构的应用程序

## 常用命令 
```bash
    # 初始化环境
    vela-cli.exe init 
    
    # 创建功能模块
    vela-cli.exe create crawler 
    # 同步模块 
    go mod tidy
  
    # 运行 或者 调试 
    go run main.go
```

- 测试功能
> 需要在管理后台下发配置模块 调用相关函数
```lua
    print(vela.crawler.banner) -- 输出:crawler succeed
```

- 生成项目架构
```text
├── go.mod 
├── main.go
├── vela-cli.exe
└── vela-crawler
    └── lua.go
```

- 文件说明

main.go 主函数模块
```go
package main

import (
crawler "github.com/vela-ssoc/vela-crawler"
    kit "github.com/vela-ssoc/vela-kit"
    "github.com/vela-ssoc/vela-kit/vela"
)

func main() {
    deploy := kit.New("vela", 
		kit.All(),  //是否注入默认所有模块
		kit.Use(func(xEnv vela.Environment) {

        //add module
        crawler.WithEnv(xEnv)
    }))

    deploy.Debug(kit.Hide{
        Lan:      []string{"ws://127.0.0.1"}, //调试入口
        Hostname: "broker.vela-ssoc.com", //调试管理后台主机名
        Edition:  "2.2.0",
        Protect:  true,
    })
}

```

go.mod 主要记录默认一些 中项目路径和项目三方导入信息
```mod
# go.mod

module github.com/vela-ssoc

go 1.19

replace (
    github.com/CycloneDX/cyclonedx-go v0.7.0 => github.com/CycloneDX/cyclonedx-go v0.6.0
    github.com/spdx/tools-golang v0.4.0 => github.com/spdx/tools-golang v0.3.0
)

require github.com/vela-ssoc/vela-kit v1.2.4

```

vela-crawler 生成模块路径 主要包含结构化入口信息 主要包含lua.go 将功能注入到lua虚拟机中
```go
package crawler

import (
    "github.com/vela-ssoc/vela-kit/lua"
    "github.com/vela-ssoc/vela-kit/vela"
)

var xEnv vela.Environment

func crawlerL(L *lua.LState) int {
    body := lua.Format(L, 0)
    L.Push(lua.S2L(body))
    return 1
}

func indexL(L *lua.LState, key string) lua.LValue {
    switch key {
    case "banner":
        return lua.S2L("crawler succeed")
    }
    return lua.LNil
}

func WithEnv(env vela.Environment) {
    xEnv = env
    xEnv.Set("crawler", lua.NewExport("lua.crawler.export", lua.WithIndex(indexL), lua.WithFunc(crawlerL)))
}
```
