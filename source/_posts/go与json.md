title: Go 处理 JSON 教程
author: Joe Tong
tags:
  - GO
categories:  
  - IT
date: 2019-11-07 14:19:53 
---
[参考](https://bingohuang.com/go-json/ "参考")
Go 处理 JSON 教程 - 如何创建和解析 JSON 数据

本文介绍在 Go 语言中，如何创建和解析 JSON 数据。

## 一、创建 JSON
使用 Go 标准库中的 encoding/json，可以很方便从 struct、map、slice 等数据结构体来创建 JSON 数据。

1、从 struct 创建
通过 json.Marshal，可以很方便的将一个 struct 转化为 []byte 类型的 JSON 数据。

```
package main

import (  
    "encoding/json"
    "fmt"
)

type Person struct {  
    Name   string
    Age    int
    Emails []string
}

func main() {  
    bingo := Person{
        Name:   "Bingo Huang",
        Age:    30,
        Emails: []string{"go@bingohuang.com", "me@bingohuang.com"},
    }
    json_bytes, err := json.Marshal(bingo)
    if err != nil {
        panic(err)
    }
    fmt.Printf("%s",json_bytes)
}
```

输出：
`{"Name":"Bingo Huang","Age":30,"Emails":[go@bingohuang.com", "me@bingohuang.com"]}`


**注意**： 
1. 结构体中的字段名，需要大写开头，否则不会被输出
2. 结构体中可以嵌入其他结构体
3. json.Marshal 函数返回一个 []byte 类型的 JSON 数据和一个 error，别忘了处理该 error
4. 返回的 []byte 类型 JSON 数据，如果你想当成字符串处理，需要做强制转换：string(json_bytes)


2、定义字段名称
如果你希望输出的 JSON 字段，不一定要大写字母开头，甚至想自定义 JSON 字段名，可以打上 json tag。


```
type Person struct {  
    Name   string   `json:"name"`
    Age    int      `json:"age"`
    Emails []string `json:"emails"`
}
```

再次输出：
`{"name":"Bingo Huang","age":30,"emails":["go@bingohuang.com","me@bingohuang.com"]}`

3、忽略空字段
如果你希望某些字段在数据为空时能能被自动忽略，只需加上 omitempty 标签。

```
package main

import (  
    "encoding/json"
    "fmt"
)

type Person struct {  
    Name   string      `json:"name,omitempty"`
    Age    int         `json:"age,omitempty"`
    Emails []string    `json:"emails,omitempty"`
}

func main() {  
    bingo := Person{
        Name: "Bingo Huang",
    }
    json_bytes, _ := json.Marshal(bingo)
    fmt.Printf("%s", json_bytes)
}
```

输出：

`{"name":"Bingo Huang"}`


4、跳过某字段
如果你不想让某个字段（无论该字段数据是否为空）输出到 JSON 中，可使用 - 来跳过。

```
package main

import (  
    "encoding/json"
    "fmt"
)

type Person struct {  
    Name   string      `json:"name,omitempty"`
    Age    int         `json:"-"`
    Emails []string    `json:"emails,omitempty"`
}

func main() {  
    bingo := Person{
        Name: "Bingo Huang",
        Age:  30,
    }
    json_bytes, _ := json.Marshal(bingo)
    fmt.Printf("%s", json_bytes)
}
```


输出：

`{"name":"Bingo Huang"}`

5、从 map 和 slice 创建
从 map 和 slice 来创建 JSON，也很容易，直接上示例

```
package main

import (  
    "encoding/json"
    "fmt"
)

func main() {  
    bingo := map[string]interface{}{
        "name": "Bingo Huang",
        "age":  30,
    }
    json_bytes, _ := json.Marshal(bingo)
    fmt.Printf("%s", json_bytes)
}
```

输出：

`{"age":30,"name":"Bingo Huang"}`


```
package main

import (  
    "encoding/json"
    "fmt"
)

func main() {  
    emails := []string{"go@bingohuang.com", "me@bingohuang.com"}
    json_bytes, _ := json.Marshal(emails)
}
```

输出：
`["go@bingohuang.com","me@bingohuang.com"]`

## 二、解析 JSON
创建 JSON 是如此方便，那反过来看看 GO 如何解析 JSON。

1、解析到 struct
相反，可以通过 json.Unmarshal 来将一个 []byte 类型的 JSON 数据解析到 struct 中。

```
package main

import (  
    "encoding/json"
    "fmt"
)

type Person struct {  
    Name   string   `json:"name"`
    Age    int      `json:"age"`
    Emails []string `json:"emails"`
}

func main() {  
    json_bytes := []byte(`
        {
            "Name":"Bingo Huang",
            "Age":30,
            "Emails":["go@bingohuang.com","me@bingohuang.com"]
        }
    `)
    bingo := Person{}
    err := json.Unmarshal(json_bytes, &amp;bingo)
    if err != nil {
        panic(err)
    }
    fmt.Println(bingo.Name, bingo.Age, bingo.Emails)
}
```

这里我们将 json_bytes 和 Person 的指针传递给 json.Unmarshal，注意要传递结构体的指针，因为解析器需要给结构体写入数据。

如果 JSON 数据中不包含结构体的某些字段，转换成 struct 时字段会被忽略，反过来如果结构体中某些字段不需要输出 JSON，也会被忽略，如下：

```
package main

import (  
    "encoding/json"
    "fmt"
)

type Person struct {  
    Name    string   `json:"name"`
    Age     int      `json:"age"`
    Emails  []string `json:"emails"`
    Address string
}

func main() {  
    json_bytes := []byte(`
        {
            "Name":"Bingo Huang",
            "Age":30
        }
    `)
    var bingo Person
    err := json.Unmarshal(json_bytes, &amp;bingo)
    if err != nil {
        panic(err)
    }
    fmt.Println(bingo.Emails)
    fmt.Println(bingo.Address)
}
```
这里 Emails 和 Address 字段都被忽略了，输出都为空。

2、解析到 map 和 slice
通过 json.Unmarshal 将 JSON 数据解析到 map 和 slice 也十分方便，这里以 map 为例。


```
package main

import (  
    "encoding/json"
    "fmt"
)

func main() {  
    json_bytes := []byte(`
        {
            "Name":"Bingo Huang",
            "Age":30,
            "Emails":["go@bingohuang.com","me@bingohuang.com"]
        }
    `)
    var bingo map[string]interface{}
    err := json.Unmarshal(json_bytes, &amp;bingo)
    if err != nil {
        panic(err)
    }
    fmt.Println(bingo["Name"], bingo["Age"], bingo["Emails"], bingo["Score"])
}
```

## 三、JSON 流处理
json 包还提供了另外两个函数 NewEncoder 和 NewDecoder，提供 Encoder 和 Decoder 类型，可支持 io.Reader 和 io.Writer 接口做流处理。

1、创建 JSON 到文件中
我们可以将一个 JSON 文件转化到结构体中。

```
package main

import (  
    "encoding/json"
    "fmt"
    "os"
)

func main() {  
    fileReader, _ := os.Open("bingo.json")
    var bingo map[string]interface{}
    json.NewDecoder(fileReader).Decode(&amp;bingo)
    fmt.Println(bingo)
}
```
bingo.json 内容如下：

```
{
  "Name":"Bingo Huang",
  "Age":30,
  "Emails":["go@bingohuang.com","me@bingohuang.com"]
}
```
`map[Name:Bingo Huang Age:30 Emails:[go@bingohuang.com me@bingohuang.com]]  `

2、从文件中解析 JSON
同样，可以反过来，将对象转化为 JSON 格式，并写入文件中。


```
package main

import (  
    "encoding/json"
    "os"
)

type Person struct {  
    Name   string
    Age    int
    Emails []string
}

func main() {  
    bingo := Person{
        Name:   "Bingo Huang",
        Age:    30,
        Emails: []string{"go@bingohuang.com","me@bingohuang.com"},
    }
    fileWriter, _ := os.Create("output.json")
    json.NewEncoder(fileWriter).Encode(bingo)
}
```

`{"Name":"Bingo Huang","Age":30,"Emails":["go@bingohuang.com","me@bingohuang.com"]}`



