# golang--组合取代继承
---

Go 不支持继承，但它支持组合（Composition）。

# 通过嵌套结构体进行组合
在 Go 中，通过在结构体内嵌套结构体，可以实现组合。

组合的典型例子就是博客帖子。每一个博客的帖子(Post)都有标题、内容和作者信息(author)。“作者信息”这个结构体内嵌在“帖子”这个结构体中。

下面写一个这个例子。
```
package main

import "fmt"

type author struct {
	firstName string
	lastName  string
	bio       string
}

func (a author)fullName() string {
	return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {
	title string
	content string
	author
}

func (p post)details()  {
	fmt.Println("title: ", p.title)
	fmt.Println("content: ", p.content)
	//一旦结构体内嵌套了一个结构体字段，Go 可以使我们访问其嵌套的字段，好像这些字段属于外部结构体一样。
	fmt.Println("author: ", p.author.fullName()) //可替换为p.fullName()
	fmt.Println("bio: ", p.author.bio) //可替换为p.bio
}

func main()  {
	author1 := author{
		"Naveen",
		"Ramanathan",
		"Golang Enthusiast",
	}
	post1 := post{
		"Inheritance in Go",
		"Go supports composition instead of inheritance",
		author1,
	}
	post1.details()
}
```
改程序的执行结果为：
```
title:  Inheritance in Go
content:  Go supports composition instead of inheritance
author:  Naveen Ramanathan
bio:  Golang Enthusiast
```

# 结构体切片的嵌套
可以进一步处理这个示例，使用博客帖子的切片来创建一个网站。
```
package main

import "fmt"

type author struct {
	firstName string
	lastName  string
	bio       string
}

func (a author)fullName() string {
	return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {
	title string
	content string
	author
}

func (p post)details()  {
	fmt.Println("title: ", p.title)
	fmt.Println("content: ", p.content)
	//一旦结构体内嵌套了一个结构体字段，Go 可以使我们访问其嵌套的字段，好像这些字段属于外部结构体一样。
	fmt.Println("author: ", p.author.fullName()) //可替换为p.fullName()
	fmt.Println("bio: ", p.author.bio) //可替换为p.bio
}

type website struct {
	posts []post
}

func (w website)contents()  {
	fmt.Println("website contents: ")
	fmt.Println()
	for _, v := range w.posts{
		v.details()
		fmt.Println()
	}
}

func main()  {
	author1 := author{
		"Naveen",
		"Ramanathan",
		"Golang Enthusiast",
	}
	post1 := post{
		"Inheritance in Go",
		"Go supports composition instead of inheritance",
		author1,
	}
	post2 := post{
		"Struct instead of Classes in Go",
		"Go does not support classes but methods can be added to structs",
		author1,
	}
	post3 := post{
		"Concurrency",
		"Go is a concurrent language and not a parallel one",
		author1,
	}
	w := website{
		posts: []post{post1, post2, post3},
	}
	w.contents()
}
```
上述程序的执行结果为：
```
website contents: 

title:  Inheritance in Go
content:  Go supports composition instead of inheritance
author:  Naveen Ramanathan
bio:  Golang Enthusiast

title:  Struct instead of Classes in Go
content:  Go does not support classes but methods can be added to structs
author:  Naveen Ramanathan
bio:  Golang Enthusiast

title:  Concurrency
content:  Go is a concurrent language and not a parallel one
author:  Naveen Ramanathan
bio:  Golang Enthusiast
```

需要注意的是，**结构体不能嵌套一个匿名切片。**如果上面`website`结构体像下面的写法：
```
type website struct {
	[]post
}
```
就会报错，`syntax error: unexpected [, expecting field name or embedded type`