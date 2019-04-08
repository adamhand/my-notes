# golang--多态
---
Go 通过接口来实现多态。

# 使用接口实现多态
一个类型如果定义了接口的所有方法，那它就隐式地实现了该接口。

通过一个程序我们来理解 Go 语言的多态，它会计算一个组织机构的净收益。假设这个虚构的组织所获得的收入来源于两个项目：`fixed billing` 和 `time and material`。该组织的净收益等于这两个项目的收入总和。假设货币单位是美元。因此货币只需简单地用 int 来表示。

程序如下：
```
package main

import "fmt"

/*
收益接口。包含了两个方法：calculate() 计算并返回项目的收入，
而 source() 返回项目名称。
 */
type Income interface {
	calculate() int
	source() string
}

/*
FixedBilling 项目的结构体类型
有两个字段：projectName 表示项目名称，而 biddedAmount 表示组织向该项目投标的金额。
 */
type FixedBilling struct {
	projectName string
	biddedAmount int
}

/*
TimeAndMaterial项目结构体类型
拥有三个字段名：projectName、noOfHours 和 hourlyRate。
 */
type TimeAndMaterial struct {
	projectName string
	noOfHours int
	hourlyRate int
}

/*
实现接口
 */
func (fb FixedBilling)calculate() int {
	return fb.biddedAmount
}

func (fb FixedBilling)source() string {
	return fb.projectName
}

func (tm TimeAndMaterial)calculate() int {
	return tm.hourlyRate * tm.noOfHours
}

func (tm TimeAndMaterial)source() string {
	return tm.projectName
}

/*
计算收益
根据 Income 接口的具体类型，程序会调用不同的 calculate() 和 source() 方法
于是，在 calculateNetIncome 函数中就实现了多态。
 */
func calculateIncome(ic []Income)  {
	var totalIncome int = 0
	for _, income := range ic{
		fmt.Printf("income from %s is $%d\n", income.source(), income.calculate())
		totalIncome += income.calculate()
	}
	fmt.Printf("total income is $%d\n", totalIncome)
}

func main()  {
	project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
	project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
	project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
	incomeStreams := []Income{project1, project2, project3}
	calculateIncome(incomeStreams)
}
```
上述程序的执行结果为：
```
income from Project 1 is $5000
income from Project 2 is $10000
income from Project 3 is $4000
total income is $19000
```

# 新增收益流
假设前面的组织通过广告业务，建立了一个新的收益流（`Income Stream`）。添加它非常简单，并且计算总收益也很容易，无需对 `calculateNetIncome` 函数进行任何修改。这就是多态的好处。

首先定义 `Advertisement` 类型，并在 `Advertisement` 类型中定义 `alculate()` 和 `source()` 方法。

`Advertisement` 类型有三个字段，分别是 `adName`、`CPC`（每次点击成本）和 `noOfClicks`（点击次数）。广告的总收益等于 `CPC` 和 `noOfClicks` 的乘积。
```
type Advertisement struct {
	adName string
	CPC int
	noOfClicks int
}

func (ad Advertisement)calculate() int {
	return ad.noOfClicks * ad.CPC
}

func (ad Advertisement)source() string {
	return ad.adName
}
```

稍微修改一下 `main` 函数，创建了两个广告项目，即 `bannerAd` 和 `popupAd`。`incomeStream `,把新的收益流添加进来。
```
func main() {  
    project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
    project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
    project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
    bannerAd := Advertisement{adName: "Banner Ad", CPC: 2, noOfClicks: 500}
    popupAd := Advertisement{adName: "Popup Ad", CPC: 5, noOfClicks: 750}
    incomeStreams := []Income{project1, project2, project3, bannerAd, popupAd}
    calculateNetIncome(incomeStreams)
}
```
然后程序的输出结果Wie：
```
income from Project 1 is $5000
income from Project 2 is $10000
income from Project 3 is $4000
income from Banner Ad is $1000
income from Popup Ad is $3750
total income is $23750
```