---
title: go mod 使用
date: 2019-08-10 08:29:52
tags: Go
---

Golang 1.11 版本引入的 go mod，之前一直在使用go get方式管理包


#### 本地开发环境情况说明
```
➜ /Users/jack/go/gomod/admin git:(master)>go env
GOARCH="amd64"
GOBIN="/Users/jack/go/bin"
GOCACHE="/Users/jack/Library/Caches/go-build"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/jack/go"
GOPROXY=""
GORACE=""
GOROOT="/usr/local/go"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
GCCGO="gccgo"
CC="clang"
CXX="clang++"
CGO_ENABLED="1"
GOMOD="/Users/jack/go/gomod/admin/go.mod"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/np/ks8717cn2_q91yhkw6gz08bm0000gn/T/go-build544492236=/tmp/go-build -gno-record-gcc-switches -fno-common"
```
之前项目都在 ~/go/src/里面，所有的包也是src目录里面

---

#### 现有项目迁移

1. 把之前的工程 src/admin 目录拷贝到$GOPATH/src之外）
2. 在工程目录下执行 go mod init admin 该命令会创建一个go.mod文件
3. 然后在该目录下执行 go build ，就可以了。你将看到：

```
    ➜ /Users/jack/go/gomod/admin git:(dev)>make run
    go build -o admin -v 
    go: finding github.com/go-sql-driver/mysql v1.4.1
    go: finding github.com/astaxie/beego v1.12.0
    go: finding github.com/pkg/errors v0.8.1
    go: finding github.com/shiena/ansicolor v0.0.0-20151119151921-a422bbe96644
    go: finding github.com/cloudflare/golz4 v0.0.0-20150217214814-ef862a3cdc58
    go: finding github.com/Knetic/govaluate v3.0.0+incompatible
    go: finding github.com/bradfitz/gomemcache v0.0.0-20180710155616-bc664df96737
    go: finding github.com/OwnLocal/goes v1.0.0
    go: finding github.com/go-redis/redis v6.14.2+incompatible
    go: finding github.com/casbin/casbin v1.7.0
    go: finding github.com/couchbase/go-couchbase v0.0.0-20181122212707-3e9b6e1258bb
    go: finding github.com/beego/goyaml2 v0.0.0-20130207012346-5545475820dd
    go: finding github.com/mattn/go-sqlite3 v1.10.0
    go: finding github.com/beego/x2j v0.0.0-20131220205130-a0352aadc542
    go: finding github.com/siddontang/go v0.0.0-20180604090527-bdc77568d726
    go: finding github.com/wendal/errors v0.0.0-20130201093226-f66c77a7882b
    Fetching https://golang.org/x/net?go-get=1
    go: finding github.com/syndtr/goleveldb v0.0.0-20181127023241-353a9fca669c
    go: finding github.com/couchbase/goutils v0.0.0-20180530154633-e865a1461c8a
    go: finding github.com/elazarl/go-bindata-assetfs v1.0.0
    go: finding github.com/siddontang/ledisdb v0.0.0-20181029004158-becf5f38d373
    Parsing meta tags from https://golang.org/x/net?go-get=1 (status code 200)
    get "golang.org/x/net": found meta tag get.metaImport{Prefix:"golang.org/x/net", VCS:"git", RepoRoot:"https://go.googlesource.com/net"} at https://golang.org/x/net?go-get=1
    go: finding golang.org/x/net v0.0.0-20181114220301-adae6a3d119a
    go: finding github.com/gogo/protobuf v1.1.1
    go: finding github.com/lib/pq v1.0.0
    go: finding github.com/gomodule/redigo v2.0.0+incompatible
    go: finding github.com/pelletier/go-toml v1.2.0
    go: finding github.com/ssdb/gossdb v0.0.0-20180723034631-88f6b59b84ec
    go: finding github.com/couchbase/gomemcached v0.0.0-20181122193126-5125a94a666c
    go: finding github.com/edsrzf/mmap-go v0.0.0-20170320065105-0bce6a688712
    go: finding github.com/cupcake/rdb v0.0.0-20161107195141-43ba34106c76
    Fetching https://golang.org/x/crypto?go-get=1
    Parsing meta tags from https://golang.org/x/crypto?go-get=1 (status code 200)
    get "golang.org/x/crypto": found meta tag get.metaImport{Prefix:"golang.org/x/crypto", VCS:"git", RepoRoot:"https://go.googlesource.com/crypto"} at https://golang.org/x/crypto?go-get=1
    go: finding golang.org/x/crypto v0.0.0-20181127143415-eb0de9b17e85
    go: finding github.com/golang/snappy v0.0.0-20180518054509-2e65f85255db
    go: finding github.com/pkg/errors v0.8.0
    Fetching https://gopkg.in/yaml.v2?go-get=1
    go: finding github.com/siddontang/rdb v0.0.0-20150307021120-fc89ed2e418d
    Parsing meta tags from https://gopkg.in/yaml.v2?go-get=1 (status code 200)
    get "gopkg.in/yaml.v2": found meta tag get.metaImport{Prefix:"gopkg.in/yaml.v2", VCS:"git", RepoRoot:"https://gopkg.in/yaml.v2"} at https://gopkg.in/yaml.v2?go-get=1
    go: finding gopkg.in/yaml.v2 v2.2.1
    Fetching https://gopkg.in/check.v1?go-get=1
    Parsing meta tags from https://gopkg.in/check.v1?go-get=1 (status code 200)
    get "gopkg.in/check.v1": found meta tag get.metaImport{Prefix:"gopkg.in/check.v1", VCS:"git", RepoRoot:"https://gopkg.in/check.v1"} at https://gopkg.in/check.v1?go-get=1
    go: finding gopkg.in/check.v1 v0.0.0-20161208181325-20d25e280405
    go: downloading github.com/astaxie/beego v1.12.0
    go: downloading github.com/pkg/errors v0.8.1
    go: downloading github.com/go-sql-driver/mysql v1.4.1
    go: downloading github.com/gomodule/redigo v2.0.0+incompatible
    go: downloading github.com/shiena/ansicolor v0.0.0-20151119151921-a422bbe96644
    go: downloading golang.org/x/crypto v0.0.0-20181127143415-eb0de9b17e85
    go: downloading gopkg.in/yaml.v2 v2.2.1
    ./admin
    2019/08/06 11:41:20.004 [I] [router.go:270]  /Users/jack/go/src/admin/controllers no changed
    2019/08/06 11:41:20.005 [I] [router.go:270]  /Users/jack/go/src/admin/controllers no changed
    2019/08/06 11:41:22.070 [I] [asm_amd64.s:1333]  http server Running on http://:80
    2019/08/06 11:41:22.070 [I] [asm_amd64.s:1333]  Admin server Running on :8088
 ```
    
    可以看到会自动去下载相对应的包
    
    这个时候看项目目录里面自动创建两个文件 go.mod go.sum
    
```
    ➜ /Users/jack/go/gomod/admin git:(dev) ✗>cat go.mod go.sum 
    module admin
    
    require (
    	github.com/astaxie/beego v1.12.0
    	github.com/go-sql-driver/mysql v1.4.1
    	github.com/pkg/errors v0.8.1
    	github.com/shiena/ansicolor v0.0.0-20151119151921-a422bbe96644 // indirect
    )
    github.com/Knetic/govaluate v3.0.0+incompatible/go.mod h1:r7JcOSlj0wfOMncg0iLm8Leh48TZaKVeNIfJntJ2wa0=
    github.com/OwnLocal/goes v1.0.0/go.mod h1:8rIFjBGTue3lCU0wplczcUgt9Gxgrkkrw7etMIcn8TM=
    github.com/astaxie/beego v1.12.0 h1:MRhVoeeye5N+Flul5PoVfD9CslfdoH+xqC/xvSQ5u2Y=
    github.com/astaxie/beego v1.12.0/go.mod h1:fysx+LZNZKnvh4GED/xND7jWtjCR6HzydR2Hh2Im57o=
    github.com/beego/goyaml2 v0.0.0-20130207012346-5545475820dd/go.mod h1:1b+Y/CofkYwXMUU0OhQqGvsY2Bvgr4j6jfT699wyZKQ=
    github.com/beego/x2j v0.0.0-20131220205130-a0352aadc542/go.mod h1:kSeGC/p1AbBiEp5kat81+DSQrZenVBZXklMLaELspWU=
    github.com/bradfitz/gomemcache v0.0.0-20180710155616-bc664df96737/go.mod h1:PmM6Mmwb0LSuEubjR8N7PtNe1KxZLtOUHtbeikc5h60=
    github.com/casbin/casbin v1.7.0/go.mod h1:c67qKN6Oum3UF5Q1+BByfFxkwKvhwW57ITjqwtzR1KE=
    github.com/cloudflare/golz4 v0.0.0-20150217214814-ef862a3cdc58/go.mod h1:EOBUe0h4xcZ5GoxqC5SDxFQ8gwyZPKQoEzownBlhI80=
    github.com/couchbase/go-couchbase v0.0.0-20181122212707-3e9b6e1258bb/go.mod h1:TWI8EKQMs5u5jLKW/tsb9VwauIrMIxQG1r5fMsswK5U=
    github.com/couchbase/gomemcached v0.0.0-20181122193126-5125a94a666c/go.mod h1:srVSlQLB8iXBVXHgnqemxUXqN6FCvClgCMPCsjBDR7c=
    github.com/couchbase/goutils v0.0.0-20180530154633-e865a1461c8a/go.mod h1:BQwMFlJzDjFDG3DJUdU0KORxn88UlsOULuxLExMh3Hs=
    github.com/cupcake/rdb v0.0.0-20161107195141-43ba34106c76/go.mod h1:vYwsqCOLxGiisLwp9rITslkFNpZD5rz43tf41QFkTWY=
    github.com/edsrzf/mmap-go v0.0.0-20170320065105-0bce6a688712/go.mod h1:YO35OhQPt3KJa3ryjFM5Bs14WD66h8eGKpfaBNrHW5M=
    github.com/elazarl/go-bindata-assetfs v1.0.0/go.mod h1:v+YaWX3bdea5J/mo8dSETolEo7R71Vk1u8bnjau5yw4=
    github.com/go-redis/redis v6.14.2+incompatible/go.mod h1:NAIEuMOZ/fxfXJIrKDQDz8wamY7mA7PouImQ2Jvg6kA=
    github.com/go-sql-driver/mysql v1.4.1 h1:g24URVg0OFbNUTx9qqY1IRZ9D9z3iPyi5zKhQZpNwpA=
    github.com/go-sql-driver/mysql v1.4.1/go.mod h1:zAC/RDZ24gD3HViQzih4MyKcchzm+sOG5ZlKdlhCg5w=
    github.com/gogo/protobuf v1.1.1/go.mod h1:r8qH/GZQm5c6nD/R0oafs1akxWv10x8SbQlK7atdtwQ=
    github.com/golang/snappy v0.0.0-20180518054509-2e65f85255db/go.mod h1:/XxbfmMg8lxefKM7IXC3fBNl/7bRcc72aCRzEWrmP2Q=
    github.com/gomodule/redigo v2.0.0+incompatible h1:K/R+8tc58AaqLkqG2Ol3Qk+DR/TlNuhuh457pBFPtt0=
    github.com/gomodule/redigo v2.0.0+incompatible/go.mod h1:B4C85qUVwatsJoIUNIfCRsp7qO0iAmpGFZ4EELWSbC4=
    github.com/lib/pq v1.0.0/go.mod h1:5WUZQaWbwv1U+lTReE5YruASi9Al49XbQIvNi/34Woo=
    github.com/mattn/go-sqlite3 v1.10.0/go.mod h1:FPy6KqzDD04eiIsT53CuJW3U88zkxoIYsOqkbpncsNc=
    github.com/pelletier/go-toml v1.2.0/go.mod h1:5z9KED0ma1S8pY6P1sdut58dfprrGBbd/94hg7ilaic=
    github.com/pkg/errors v0.8.0/go.mod h1:bwawxfHBFNV+L2hUp1rHADufV3IMtnDRdf1r5NINEl0=
    github.com/pkg/errors v0.8.1 h1:iURUrRGxPUNPdy5/HRSm+Yj6okJ6UtLINN0Q9M4+h3I=
    github.com/pkg/errors v0.8.1/go.mod h1:bwawxfHBFNV+L2hUp1rHADufV3IMtnDRdf1r5NINEl0=
    github.com/shiena/ansicolor v0.0.0-20151119151921-a422bbe96644 h1:X+yvsM2yrEktyI+b2qND5gpH8YhURn0k8OCaeRnkINo=
    github.com/shiena/ansicolor v0.0.0-20151119151921-a422bbe96644/go.mod h1:nkxAfR/5quYxwPZhyDxgasBMnRtBZd0FCEpawpjMUFg=
    github.com/siddontang/go v0.0.0-20180604090527-bdc77568d726/go.mod h1:3yhqj7WBBfRhbBlzyOC3gUxftwsU0u8gqevxwIHQpMw=
    github.com/siddontang/ledisdb v0.0.0-20181029004158-becf5f38d373/go.mod h1:mF1DpOSOUiJRMR+FDqaqu3EBqrybQtrDDszLUZ6oxPg=
    github.com/siddontang/rdb v0.0.0-20150307021120-fc89ed2e418d/go.mod h1:AMEsy7v5z92TR1JKMkLLoaOQk++LVnOKL3ScbJ8GNGA=
    github.com/ssdb/gossdb v0.0.0-20180723034631-88f6b59b84ec/go.mod h1:QBvMkMya+gXctz3kmljlUCu/yB3GZ6oee+dUozsezQE=
    github.com/syndtr/goleveldb v0.0.0-20181127023241-353a9fca669c/go.mod h1:Z4AUp2Km+PwemOoO/VB5AOx9XSsIItzFjoJlOSiYmn0=
    github.com/wendal/errors v0.0.0-20130201093226-f66c77a7882b/go.mod h1:Q12BUT7DqIlHRmgv3RskH+UCM/4eqVMgI0EMmlSpAXc=
    golang.org/x/crypto v0.0.0-20181127143415-eb0de9b17e85 h1:et7+NAX3lLIk5qUCTA9QelBjGE/NkhzYw/mhnr0s7nI=
    golang.org/x/crypto v0.0.0-20181127143415-eb0de9b17e85/go.mod h1:6SG95UA2DQfeDnfUPMdvaQW0Q7yPrPDi9nlGo2tz2b4=
    golang.org/x/net v0.0.0-20181114220301-adae6a3d119a/go.mod h1:mL1N/T3taQHkDXs73rZJwtUhF3w3ftmwwsq0BUmARs4=
    gopkg.in/check.v1 v0.0.0-20161208181325-20d25e280405/go.mod h1:Co6ibVJAznAaIkqp8huTwlJQCZ016jof/cbN4VW5Yz0=
    gopkg.in/yaml.v2 v2.2.1 h1:mUhvW9EsL+naU5Q3cakzfE91YhliOondGd6ZrsDBHQE=
    gopkg.in/yaml.v2 v2.2.1/go.mod h1:hI93XBmqTisBFMUTm0b8Fm+jr3Dg1NNxqwp+5A1VGuI=
```

    下载下来的包放在 $GOPATH/pkg/mod 目录里面
    
```
    ➜ /Users/jack/go/pkg/mod >tree -L 2 $GOPATH/pkg/mod
    /Users/jack/go/pkg/mod
    ├── cache
    │   ├── download
    │   └── vcs
    ├── github.com
    │   ├── astaxie
    │   ├── go-sql-driver
    │   ├── gomodule
    │   ├── pkg
    │   └── shiena
    ├── golang.org
    │   └── x
    └── gopkg.in
        └── yaml.v2@v2.2.1
    
    13 directories, 0 files
```

    
学习参考

[using-go-modules](https://blog.golang.org/using-go-modules)

[Golang官方包依赖管理工具 go mod 简明教程](https://ieevee.com/tech/2018/08/28/go-modules.html)