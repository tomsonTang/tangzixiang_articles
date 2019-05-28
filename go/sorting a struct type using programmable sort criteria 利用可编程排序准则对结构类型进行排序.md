# sorting a struct type using programmable sort criteria 利用可编程排序准则对结构类型进行排序

在 go 源码中看到一个设计模式

代码位置： [`go/src/sort/example_keys_test.go`](https://github.com/golang/go/blob/master/src/sort/example_keys_test.go)
  
```go
// Copyright 2013 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

package sort_test

import (
	"fmt"
	"sort"
)

// A couple of type definitions to make the units clear.
type earthMass float64
type au float64

// A Planet defines the properties of a solar system object.
type Planet struct {
	name     string
	mass     earthMass
	distance au
}

// By is the type of a "less" function that defines the ordering of its Planet arguments.
type By func(p1, p2 *Planet) bool

// Sort is a method on the function type, By, that sorts the argument slice according to the function.
func (by By) Sort(planets []Planet) {
	ps := &planetSorter{
		planets: planets,
		by:      by, // The Sort method's receiver is the function (closure) that defines the sort order.
	}
	sort.Sort(ps)
}

// planetSorter joins a By function and a slice of Planets to be sorted.
type planetSorter struct {
	planets []Planet
	by      func(p1, p2 *Planet) bool // Closure used in the Less method.
}

// Len is part of sort.Interface.
func (s *planetSorter) Len() int {
	return len(s.planets)
}

// Swap is part of sort.Interface.
func (s *planetSorter) Swap(i, j int) {
	s.planets[i], s.planets[j] = s.planets[j], s.planets[i]
}

// Less is part of sort.Interface. It is implemented by calling the "by" closure in the sorter.
func (s *planetSorter) Less(i, j int) bool {
	return s.by(&s.planets[i], &s.planets[j])
}

var planets = []Planet{
	{"Mercury", 0.055, 0.4},
	{"Venus", 0.815, 0.7},
	{"Earth", 1.0, 1.0},
	{"Mars", 0.107, 1.5},
}

// ExampleSortKeys demonstrates a technique for sorting a struct type using programmable sort criteria.
func Example_sortKeys() {
	// Closures that order the Planet structure.
	name := func(p1, p2 *Planet) bool {
		return p1.name < p2.name
	}
	mass := func(p1, p2 *Planet) bool {
		return p1.mass < p2.mass
	}
	distance := func(p1, p2 *Planet) bool {
		return p1.distance < p2.distance
	}
	decreasingDistance := func(p1, p2 *Planet) bool {
		return distance(p2, p1)
	}

	// Sort the planets by the various criteria.
	By(name).Sort(planets)
	fmt.Println("By name:", planets)

	By(mass).Sort(planets)
	fmt.Println("By mass:", planets)

	By(distance).Sort(planets)
	fmt.Println("By distance:", planets)

	By(decreasingDistance).Sort(planets)
	fmt.Println("By decreasing distance:", planets)

	// Output: By name: [{Earth 1 1} {Mars 0.107 1.5} {Mercury 0.055 0.4} {Venus 0.815 0.7}]
	// By mass: [{Mercury 0.055 0.4} {Mars 0.107 1.5} {Venus 0.815 0.7} {Earth 1 1}]
	// By distance: [{Mercury 0.055 0.4} {Venus 0.815 0.7} {Earth 1 1} {Mars 0.107 1.5}]
	// By decreasing distance: [{Mars 0.107 1.5} {Earth 1 1} {Venus 0.815 0.7} {Mercury 0.055 0.4}]
}

```
1. 传统的排序我们是通过实现 `sort` 接口，在其 `Less` 方法中对需要排序的内容进行顺序判断
2. 上述例子能够灵活的使用结构体的任意字段作为排序规则依据(name/mass/distance) ⬇
3. 使用组合的方式将待排序内容 `planets` 及自定义排序功能 `by` 进行组合 得到一个新的结构体 `planetSorter`
4. 在 `planetSorter` 上实现 `sort` 接口后， 便可直接调用 `sort.Sort(ps)` 对进行排序，排序的实际规则在 `func (s *planetSorter) Less(i, j int) bool` 内实现，在 `Less` 方法中调用了自己的 `by`
5. `planetSorter` 是一个内部使用的结构体，外部无需了解
6. 最后得到的是一种设计模式：(功能).(对象) 和以往的设计模式不同，以往都是面向对象的设计模式即 (对象).(功能)
7. 这种设计模式的好处是一个功能可以多处复用在不同对象上
8. 这种模式首先对功能定义进行抽象，然后为这个功能定义接收将作用此功能的对象的方法 这里是 `func (by By) Sort(planets []Planet) `
9. 这里的功能定义可以是一个函数，譬如上述的 `By`
