# golang--Map
---

# 基本语法
map 是在 Go 中将值（value）与键（key）关联的内置类型。通过相应的键可以获取到值。

通过向 make 函数传入键和值的类型，可以创建 map。make(map[type of key]type of value) 是创建 map 的语法。
```
personSalary := make(map[string]int)
```
上面的代码创建了一个名为 personSalary 的 map，其中键是 string 类型，而值是 int 类型。**map 必须使用 make 函数初始化。**



# 给 map 添加元素
给 map 添加新元素的语法和数组相同。
```
package main

import (
    "fmt"
)

func main() {
    personSalary := make(map[string]int)
    personSalary["steve"] = 12000
    personSalary["jamie"] = 15000
    personSalary["mike"] = 9000
    fmt.Println("personSalary map contents:", personSalary)
}
```

也可以在声明的时候初始化 map。
```
package main

import (  
    "fmt"
)

func main() {  
    personSalary := map[string]int {
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("personSalary map contents:", personSalary)
}
```

**键不一定只能是 string 类型。所有可比较的类型，如 boolean，interger，float，complex，string 等，都可以作为键。**

# 获取 map 中的元素
获取 map 元素的语法是 map[key] 。
```
package main

import (
    "fmt"
)

func main() {
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    employee := "jamie"
    fmt.Println("Salary of", employee, "is", personSalary[employee])
}
```
**如果获取一个不存在的元素，map 会返回该元素类型的零值。**如果想知道 map 中到底是不是存在这个 key，该怎么做：
```
value, ok := map[key]
```
如果 ok 是 true，表示 key 存在，key 对应的值就是 value ，反之表示 key 不存在。
```
package main

import "fmt"

func main() {
	personSalary := map[string]int{
		"bob":1000,
		"alice":1200,
	}

	personSalary["mike"] = 1300
	newEmp := "joe"
	value, ok := personSalary[newEmp]
	if ok == true{
		fmt.Println("salary of ", newEmp, "is ", value)
	}else {
		fmt.Println(newEmp," is not found")
	}
}
```
结果如下：
```
joe  is not found
```

# 遍历map
遍历 map 中所有的元素需要用 for range 循环。
```
package main

import (
    "fmt"
)

func main() {
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("All items of a map")
    for key, value := range personSalary {
        fmt.Printf("personSalary[%s] = %d\n", key, value)
    }

}
```

上面程序输出：
```
All items of a map
personSalary[mike] = 9000
personSalary[steve] = 12000
personSalary[jamie] = 15000
```
**有一点很重要，当使用 for range 遍历 map 时，不保证每次执行程序获取的元素顺序相同。**
# 删除 map 中的元素
删除 map 中 key 的语法是 delete(map, key)。这个函数没有返回值。
```
package main

import (  
    "fmt"
)

func main() {  
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("map before deletion", personSalary)
    delete(personSalary, "steve")
    fmt.Println("map after deletion", personSalary)

}
```
执行结果为:
```
map before deletion map[steve:12000 jamie:15000 mike:9000]
map after deletion map[mike:9000 jamie:15000]
```

# 获取 map 的长度
获取 map 的长度使用 len 函数。
```
package main

import (
    "fmt"
)

func main() {
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("length is", len(personSalary))

}
```

上述程序中的 len(personSalary) 函数获取了 map 的长度。程序输出 length is 3。

# Map 是引用类型
和 slices 类似，map 也是引用类型。当 map 被赋值为一个新变量的时候，它们指向同一个内部数据结构。因此，改变其中一个变量，就会影响到另一变量。
```
package main

import (
    "fmt"
)

func main() {
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("Original person salary", personSalary)
    newPersonSalary := personSalary
    newPersonSalary["mike"] = 18000
    fmt.Println("Person salary changed", personSalary)

}
```
程序输出：
```
Original person salary map[steve:12000 jamie:15000 mike:9000]
Person salary changed map[steve:12000 jamie:15000 mike:18000]
```
**当 map 作为函数参数传递时也会发生同样的情况。函数中对 map 的任何修改，对于外部的调用都是可见的。**

# Map 的相等性
map 之间不能使用 == 操作符判断，== 只能用来检查 map 是否为 nil。
```
package main

func main() {
    map1 := map[string]int{
        "one": 1,
        "two": 2,
    }

    map2 := map1

    if map1 == map2 {
    }
}
```

上面程序抛出编译错误 `invalid operation: map1 == map2 (map can only be compared to nil)`。

判断两个 map 是否相等的方法是遍历比较两个 map 中的每个元素。