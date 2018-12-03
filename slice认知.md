# slice 认知

## array
Go中的数组是一个重要概念，但是一般使用的非常少，很重要的一点就是长度也是数组类型的一部分
```
	var buffer [256]byte
	var buffer [512]byte
```

是完全不一样的类型，也就是说如果一个函数接受[256]byte，那么传入[512]byte就会报错

## slice 结构
先从如何初始化一个slice开始
```
	s1 := make([]int, 0, 1)
	fmt.Println(len(s1), cap(s1)) // 0 1
	
	s2 := []int{1, 2, 4}
	fmt.Println(len(s2), cap(s2)) // 3 3
	
	var s3 []int
	s3 = make([]int, 0, 1)
```
slice的概念中会涉及到len和cap
* len 表示当前slice实际元素个数
* cap 表示当前slice容量

来看看slice的结构描述
```$golang
	type sliceHeader struct {
        Length        int
        Capacity      int
        ZerothElement *byte // slice的头指针
    }
```


## slice 传参
需要注意三个概念
* 第一个是copy传值，传入函数的值是一个sliceHeader的拷贝
* 第二个sliceHeader的拷贝指向的是底层的同一份数组，所以可以修改底部数组的值
* 第三个是因为是copy传值，所以本sliceHeader本身的修改，不会影响原来的sliceHeader

## slice 扩容
### 官方扩容
```$golang
	s = append(s, item) //item是一个元素
	
	s = append(s, s2) //s2是同类型的slice
```

### 理解下实现
```$golang
	// Append appends the elements to the slice.
    // Efficient version.
    func Append(slice []int, elements ...int) []int {
        n := len(slice)
        total := len(slice) + len(elements)
        if total > cap(slice) {
            // Reallocate. Grow to 1.5 times the new size, so we can still grow.
            newSize := total*3/2 + 1 // 这里的+1 防止total为0的情况
            newSlice := make([]int, total, newSize)
            copy(newSlice, slice)
            slice = newSlice
        }
        slice = slice[:total]
        copy(slice[n:], elements) //整体拷贝进去
        return slice
    }
```

## 总结
slice是golang中非常重要的概念