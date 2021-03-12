#### paladin

##### 项目简介

paladin 是一个config SDK客户端，包括了file、mock几个抽象功能，方便使用本地文件或者sven配置中心，并且集成了对象自动reload功能。  

example.toml

[spu]
	addr = "127.0.0.1:3306"
	dsn = "{user}:{password}@tcp(127.0.0.1:3306)/{database}?timeout=1s&readTimeout=1s&writeTimeout=1s&parseTime=true&loc=Local&charset=utf8mb4,utf8"
	readDSN = ["{user}:{password}@tcp(127.0.0.2:3306)/{database}?timeout=1s&readTimeout=1s&writeTimeout=1s&parseTime=true&loc=Local&charset=utf8mb4,utf8","{user}:{password}@tcp(127.0.0.3:3306)/{database}?timeout=1s&readTimeout=1s&writeTimeout=1s&parseTime=true&loc=Local&charset=utf8,utf8mb4"]
	

文件位置./cfg/
实例代码


package main
 
import (
    "fmt"
    "flag"
    "github.com/netaxcess/conf/paladin"
    
)

type Mysql struct {
    Addr        string
    Dsn         string
    ReadDSN     string
}

func main() {
    flag.Parse()
	if err := paladin.Init(); err != nil {
		panic(err)
	}
    
    var dc struct {
			Spu *Mysql
	}

    paladin.Get("mysql.toml").UnmarshalTOML(&dc)
    fmt.Println(dc.Spu)
}

local files:
```
demo -conf=/data/conf/app/msm-servie.toml
// or dir
demo -conf=/data/conf/app/

```
example:
```
type exampleConf struct {
	Bool   bool
	Int    int64
	Float  float64
	String string
}

func (e *exampleConf) Set(text string) error {
	var ec exampleConf
	if err := toml.Unmarshal([]byte(text), &ec); err != nil {
		return err
	}
	*e = ec
	return nil
}

func ExampleClient() {
	if err := paladin.Init(); err != nil {
		panic(err)
	}
	var (
		ec   exampleConf
		eo   exampleConf
		m    paladin.TOML
		strs []string
	)
	// config unmarshal
	if err := paladin.Get("example.toml").UnmarshalTOML(&ec); err != nil {
		panic(err)
	}
	// config setter
	if err := paladin.Watch("example.toml", &ec); err != nil {
        panic(err)
    }
	// paladin map
	if err := paladin.Watch("example.toml", &m); err != nil {
        panic(err)
    }
	s, err := m.Value("key").String()
	b, err := m.Value("key").Bool()
	i, err := m.Value("key").Int64()
	f, err := m.Value("key").Float64()
	// value slice
	err = m.Value("strings").Slice(&strs)
	// watch key
	for event := range paladin.WatchEvent(context.TODO(), "key") {
		fmt.Println(event)
	}
}
```

##### 编译环境

- **请只用 Golang v1.12.x 以上版本编译执行**

##### 依赖包
