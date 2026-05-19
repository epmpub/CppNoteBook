## golang reflection

1. 大量的遍历与嵌套；
2. 递归
3. 如果有相同的类型，注意顺序，第一个和第二个类型；

```golang
package main

import (
	"fmt"
	"reflect"
)

type address struct {
	City  string
	State string
}

type Person struct {
	FName string
	LName string
	Age   int
	Addr  address
}

func foo() {

	var p Person

	v := reflect.ValueOf(&p).Elem()
	t := v.Type()

	stringIndex := 0

	// 遍历所有字段
	for i := 0; i < v.NumField(); i++ {

		field := v.Field(i)
		fieldType := t.Field(i)

		fmt.Println("Field:", fieldType.Name)

		// 根据字段类型动态赋值
		switch field.Kind() {

		case reflect.String:
			if stringIndex == 0 {
				field.SetString("Alice")
			} else {
				field.SetString("Smith")
			}
			stringIndex++

		case reflect.Int:
			field.SetInt(123)
		case reflect.Struct:
			// 处理嵌套结构体
			for j := 0; j < field.NumField(); j++ {
				nestedField := field.Field(j)
				nestedFieldType := field.Type().Field(j)

				fmt.Println("  Nested Field:", nestedFieldType.Name)

				switch nestedField.Kind() {
				case reflect.String:
					//use target struct field name to assign value
					switch nestedFieldType.Name {
					//if i don't use target struct field name to assign value, it will assign value to all string field in nested struct
					case "City":
						nestedField.SetString("New York")
					case "State":
						nestedField.SetString("NY")
					}

				case reflect.Int:
					nestedField.SetInt(456)
				}
			}
		}
	}

	fmt.Printf("%+v\n", p)
}

func main() {
	foo()
}

```



```go
package main

import (
	"fmt"
	"reflect"
)

type Address struct {
	City  string
	State string
	Zip   int
}

type Person struct {
	FName string
	LName string
	Age   int
	Addr  Address
}

func fill(v reflect.Value) {

	if v.Kind() == reflect.Ptr {
		v = v.Elem()
	}

	if v.Kind() != reflect.Struct {
		return
	}

	for i := 0; i < v.NumField(); i++ {

		field := v.Field(i)

		switch field.Kind() {

		case reflect.String:
			field.SetString("hello")

		case reflect.Int:
			field.SetInt(123)

		case reflect.Struct:
			fill(field)
		}
	}
}

func main() {

	var p Person

	fill(reflect.ValueOf(&p))

	fmt.Printf("%+v\n", p)
}
```

