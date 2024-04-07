#### 1. Golang Map元素取址问题： cannot assign to struct field XXXX in map

```go
// 问题描述 ：go 中对 map 类型中的 struct 赋值报错
	type Entity struct {
		Value string
	}

	entityMap := make(map[string]Entity, 0)
	entityMap["cat"] = Entity{Value: "This is a cat"}
	entityMap["cat"].Value= "This is a another cat"  // 报错行
	fmt.Println("value ",entityMap["cat"].Value)
  
  // 编译报错 ： cannot assign to struct field entityMap[“cat”].Value in map
  // 原因是 map 元素是无法取址的，也就说可以得到 a[“tao”], 但是无法对其进行修改。
  
 // 解决办法：使用指针的map
entityMap := make(map[string]*Entity, 2)
```

