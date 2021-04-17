---
title: "Go notes (OOP topics) "
date: 2020-06-08T10:00:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["golang", "go", "google","coursera","programming","examples"]
author: "mrturkmen"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Go Programming Language Notes."
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "../../images/golangposts/header.png" # image path/url
    alt: "" # alt text
    caption: "" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/tree/master/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

In this post, I would like to share the notes that I took when I was completing following Go series education on Coursera. I can recommend it for anyone who would like to get rapid introduction to Go programming. 

[Programming with Google Go](https://www.coursera.org/specializations/google-golang)
  

![Go course certificate](../../images/go_certificate.png)


I should admit that although it looks fancy, nothing can be compared to actual development and contribution to open source projects. 
The course itself is quite interesting and contains very handy exercises regarding to Go development mentality. I would recommend it for anyone who has some interest in Go development and do not know where to start. 

There will be following headers and subheaders which contains some notes on Go Programming language as bulletpoints. 

## OOP in Go 

In Go, there is no such a concept of object oriented programming as in Java, however, it is possible to imply similar approach (object oriented approach) in Go using interfaces and structs. 


### Classes 

 __There is no "Class" keyword in Go.__

-	Collection of data fields and functions that share a well-defined responsibility 
    -  	Example: Point class 
    -	Used in a geometry program
    -	Data: x coordinate,  y coordinate 
    -	Funtions: 
        -	`DistToOrigin()`, `Quadrant()`
        -	`AddXOffSet()`, `AddYOffset()`
    -	Classes are template    
    -	Contain data fields, not data 

Classes are supported with structs in Go. 

```go

type Point struct {
    X float64
    Y float64
}

```
- Structs with methods, structs and methods together allow arbitrary data and functions to be composed.

```go

func (p Point) DistToOrig() {
  t:= math.Pow(p.x,2) + math.Pow(p.y,2) 
  return math.Sqrt(t)
}

func main() {
	p1 := Point(3,4)
	fmt.Println(p1.DistToOrig())
}
```



### Objects 

 -  Instance of a class 
 -	Contains real data 
 -	Example: Point Class




### Encapsulation

-	Data can be protected from the programmer 
-	Data can be accessed only using methods
-	Maybe we do not trust the programmer to keep data consistent
-	Example: Double distance to origin 
    -	Option 1: Make method DoubleDist
    -	Option 2: Trust programmer to double X and Y directly 


Encapsulation is supported as following ; 

Create a package called `data` which has exported function which is `PrintX`, since it starts with capital letter, in Go, __when something starts with capital letter, it means that it is exported__

```go 
package data 
var x int=1
func PrintX() {
    fmt.Println(x)
}
```

Main package which is starting point of any application in Go. 

```go 

package main 
func main() {
    data.PrintX()
}

```

__Controlling access to Structs__

- Hide fields of structs by starting field name with a lower-case letter. 
- Define public methods which access hidden data 

```go 

package data 

type Point struct{
	x float64
	y float64
}

func (p *Point) InitMe(xn,xy float64) {
	p.x =xn
	p.y =xy
}

func (p *Point) Scale(v float64) {
		p.x=p.x*v
		p.y=p.y*v
}

func (p *Point) PrintMe() { 
	fmt.Println(p.x,p.y)
}

```

Note that all methods are public ! Since their initial character is capital letter. 

```go 
package main

func main() {
    var p data.Point 
    p.InitMe(3,4)
    p.Scale(2)
    p.PrintMe()
}

```

- Access to hidden fields can only be possible through public access. 

## Limitation of Method

- Receiver is passed implicitly as an argument to the method 
- Method cannot modify the data inside the receiver 
  
- Example: `OffsetX()` should increase x coordinate 

```go
package main

func main(){
	p1:=Point(3,4)
	p1.OffsetX(5)
}
```

__Large Receivers__

-  If receiver is large, lots of copying is required


```go

type Image [100] [100]int 
func main() { 
	i1 := GrabImage()
	i1.BlurImage()
}

```
__`10.000 ints` copied to `BlurImage()`  (Pitfalls)__

__Pointer Receivers__

```go
func (p *Point) OffsetX (v float64) {
	p.x=p.x+v
}
```
- Receiver can be a pointer to a type 
- Call by reference, pointer is passed to the method



### Point Receivers, Referenceing, Dereferencing 

- __No need to dereference__

```go
func (p *Point) OffsetX (v int) {
	p.x=p.x+v
}
```
- Point is referenced as `p`, not `*p`
  
- __No need to reference__

```go
package main 

func main() {
	p:=Point{3,4}
	p.OffsetX(5)
	fmt.Println(p.x)
}
```
- Do not need to reference when calling the method

__Good Programming Practices__

- All methods for type have pointer receivers or 
- All methods for a type have non-pointer receivers 
- Mixing pointer/non-pointer receivers for a type will get confusing ! 
  - Pointer receiver allows modification

## Polymorphism

- Ability for an object to have different forms depending on the context
- Example `Area()` function 
    - Rectangle area is `base * height`
    - Triangle area is `0.5*base*height`
- Identical at a high level of abstraction
- Different at a low level of abstraction 
  
## Inheritance (No Inheritance in GoLang)

- Sublcass inherits the methods/data of the superclass 
- Example: `Speaker` superclass
  - `Speak()` method, `pring "<noise> " `
  
- Subclassses `Cat` and `Dog` 
    - Also have the `Speak()` method 
- `Cat` and `Dog`  are different forms of speaker 
- Remember: Go does not have inheritance 

## Overriding 

- Subclass redefines a method inherited from the superclass 
- Example: `Speaker`, `Cat`, `Dog`
  - Spekaer `Speak()` prints "<noise>"
  - Cat  `Speak()` prints "meow"
  - Dog `Speak()` prints "woof"
  
- `Speak()` is polymorphic
  - Different implementations for each class 
  - Same signature (name, params and return)
  
## Interface 

- Set of method signatures 
  - Name, parameters, return values 
  - Implementation is NOT defined 
  
- Used to express conceptial similarity between types. 
- Example : `Shape2D` interface
- All 2D shapes must have `Area()` and `Perimeter()`

### Satisfying an Interface 

- Type satisfies an interface if type defines all methods specified in the interface. 
  - Same method signatures 
- Rectangle and Triangle types satisfy the Shape2D interface 
  - Must have `Area()` and `Perimeter()` methods 
  - Additional methods are OK. 
- Similar to inheritance with overriding. 

### Example 

```go 

type Shape2D interface {
		Area() float64
		Perimeter() float64
}
type Triangle {...}

func (t Triangle) Area() float64 {....}
func (t Triangle) Perimeter() float64 {....}

```
- Triangle type satisfies the Shape2D interface
- No need to state it explicitly

## Interface vs Concrete Types

### Concrete Types 
  -	Specify the exact representation of the data and methods  
  -	Complete method implementation is included
### Interface Types 
  -	Specifies some method signatures 
  -	Implementations are abstracted 

### Interface Values 
  -	Can be treated like other values 
    -	Assigned to variables 
    -	Passed, returned 
-	Interface values have two components 

1.	Dynamic Type : Concrete type which it is assigned to 
2.	Dynamic Value: Value of the dynamic type 

### Defining an interface type 

```go

type Speaker interface { Speak()}
type Dog struct {name string }
func (d Dog) Speak() {
	fmt.Println(d.name)
}
func main() {
	var s1 Speaker
	var d1 Dog{"Brian"}
    s1=d1
	s1.Speak()
```
- Dynamic type is `Dog` and dynamic value is d1. 

- An interface can have a nil dynamic value 

```go 
    var s1 Speaker 
    var d1 *Dog 
    s1=d1
```

- `d1` has no concrete value yet 
- `s1` has a dynamic type but no dynamic value 

### Nil Dynamic Value 

- Can still call the Speak() method of s1 
- Does not need a dynamic value to call
- Need to check inside the method 

```go 

func (d *Dog)Speak() { 
	if d==nil{
	    fmt.Println("<noise>")
    }else{
	    fmt.Println(d.name)
    }	
}

var s1 Speaker
var d1 *Dog 
s1=d1
s1.Speak() // it works, since s1 is mapped to d1 
```

### Nil Interface Value 

- Interface with nil dynamic type 
- Very different from an interface with a nill dynamic value 

__Nil dynamic value and valid dynamic type__

```go
var s1 Speaker 
var d1 *Dog 
s1=d1 
```

- Can call a method since type is known, Nil dynamic type
   
```go
var s1 Speaker  *(there is no actually method to call)
```
- Cannot call a method, runtime error 


__No dynamic type and no dynamic value then you cannot call the interface…__


## Using Interfaces

- Need a function which takes multiple types of parameter 
- Function foo() parameter
  - Type X or Type Y 
- Define interface Z 
- foo() parameter is interface Z 
- Types X and Y satisfy Z 
- Interface methods must be those needed by foo() 

### Example Interface for Shapes

__Pool in a Yard__

-	I need to put a pool in my yard 
-	Pool need to fit in my yard
    - Total area must be limited 
-	Pool needs to be fenced 
    - Total perimeters must be limited 
-	Need to determine if a pool shape satisfies criteria 
- FitInYard()  
  - Takes a shape as argument 
  - Returns true if the shape satisfies criteria
- FitInYard()
  - Many Possible shape types 
    - Rectangle, triangle, circle
- FitInYards() should take many shape types 
- Valid shape types must have
  - Area()
  - Perimeter()
- Any shape with these methods is OK.


```go 

type Shape2D interface {
	Area() float64
	Perimeter() float64
}
type Triangle {...}
func (t Triangle) Area() float64 {...}
func (t Triangle) Perimeter() float64 {...}
type Rectangle {...}
func(t Rectangle) Area() float64 {...}
func (t Rectangle) Perimeter() float64 {....}

```

- Rectangle and Triangle satisfy Shape2D interface. 

- `FitInYard()` Implementation 

```go

func FitInYard(s Shape2D) bool {
    if (s.Area() > 100 && s.Perimeter() > 100) {
		return true
    }
return false
} 

```

## Empty Interface 

- Empty interface specifies no methods
- All types satisfy the empty interface 
- Use it to have a function accept any type as a parameter 

```go 
func PrintMe(val interface{} ) {
	fmt.Println(val)
} 
```

### Type Assertions 

#### Concealing Type Differences__

- Interfaces hide the differences between types 

```go 
func fitInYard(s Shape2D)bool {
    if (s.Area() >100 && s.Perimeter()>100){
	    return true
    }
    return false
}
```
- Sometimes you need to treat different types in different ways

#### Exposing Type Differences

-	Example: Graphics program 
-	`DrawShape()` will draw any shape 

	```go 
    func DrawShape (s Shape2D) {..... }
    ```
-	Underlying API has different drawing functions for each shape 

    ```go 
    func DrawRect (r Rectangle) {....
    func DrawTriangle(t Triangle) {...
    ```
-	Concrete type of shape s must be determined 

#### Type Assertions for Disambiguation 

- Type assertions can be used to determine and extract the underlying concrete type 

```go 
func DrawShape(s Shape2D) bool {
	rect,ok :=s.(Rectangle)
	if ok {
		DrawRect(rect)
    }
    tri,ok  := s.(Triangle)
    if ok {
	    DrawRect(tri)
    }
}
```

- Type assertion extracts Rectangle from Shape2D
  - Concrete type in parentheses
- If interface contains concrete type 
  - rect == concrete type, ok == true
- If interface does not contain concrete type 
  - rect==zero, ok==false 

#### Type Switch

- Switch statement used with a type assertion

```go 
func DrawShape(s Shape2D) bool {
    
    switch:= sh:=s.(type) {
	case Rectangle: 
		DrawRect(sh)
	case Triangle: 
		DrawTri(sh) 
    }
}
```

## Error Interface

- Many Go programs return error interface objects to indicate errors

```go 
type error interface {
	Error() string
}
```
- Correct operation : `error==nil `
- Incorrect operation: `Error() print error message`
  



### Handling Errors 

```go 
f,err := os.Open("/harris/text.txt")
if err!=nil {
	fmt.Println(err)
	return 
}
```

- Check whether the error is nil 
- If it is not nil, handle it
- `fmt package` calls the `error()` method to generate string to print


Keep in mind that the topics which are mentioned on this post is just brief summary, it means that all subheaders and headers can be extended to any size, however these are just bulletpoints and overall information in [Functions, Methods, and Interfaces in Go](https://www.coursera.org/learn/golang-functions-methods) module of [specialization serie](https://www.coursera.org/specializations/google-golang).

In next post, notes which are taken from concurrency module of the specialization serie will be posted. 

Take care ! 👋🏻

