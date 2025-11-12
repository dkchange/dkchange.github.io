---
title: 學習go
date: 2021-02-08 13:19:58
tags:
categories:
---

# 前言

[資料來源](https://www.youtube.com/watch?v=OSPNUKoN81o&list=PLq9Ra239pNZC0MgMN4j6ZiPHv_c0UPnBX)

新一代的程式語言，由 google 開發，目的在於解決現有語言的困境
例如
python 簡單但太慢
java 太複雜度太高
c/c++ 複雜 編譯慢(編譯慢對雲端佈署來說，是個相當大的缺點)

# 特性

- 強型別靜態型別(99％的變數都會在邊一時被宣告)
- 優良的社群
- 簡單
- 編譯快
- 垃圾回收
- 內建併發
- 編譯成一個二進位 bin(不依賴其他的 dll 檔之類的)

# 開發環境設定

- windows 去載安裝檔
- linux
  預設目錄/usr/local/go
  設定環境變數 vim .bashrc

```conosle
//設定goroot目前發現會有版本上的問題，所以能避免盡量避免
export GOROOT= /usr/local/go
export PATH= $PATH:$GOROOT/bin
//用來放置第三方套件的地方，弟一個gopath設定用來放套件，之後的就依規定裝在go工作目錄中
export GOPATH = /home/stanley/golib
//讓第三方插件的二進位檔也可執行
export PATH= $PATH:$GOPATH/bin
```

> source ~/.bashrc

重新導入環境變數

> go get github.com/stanley/gocode

就會發現第三方插件被安裝到了 golib
但是有些插件我們想要分全域用和專案用
此時就要在設置另一個 GOPATH

```console
export GOPATH = $GOPATH:/home/stanley/code
```

那如何得到 go 的工作目錄呢？有 **src** 資料夾即可

- src go 自己會去尋找這個資料架
- bin 編譯過得二進位檔
- pkg 編譯過得第三方二進位檔

# 執行

run 指令

> go run xxx.go

或者 go build 之後直接執行 binary 檔

> ./xxx

# 變數宣告

- 依照宣告的位置與大寫小寫分三總作用域

  1. function 外面宣告(outside of function)

  - 規則：

    - [x] 可以用正常的宣告方式宣告，且可以做區塊宣告

      ```go
      var (
      name string = "stanley"
      age int = 30
      )
      ```

    - [ ] package 級別不能讓 go 自己判斷變數型態，所以不能使用
      ```go
      name := "stanley" age int = 30
      ```

  - **命名規則**
    - 第一個字大寫屬於 export 出去的變數 **global** 級別
    - 第一個字小寫屬於 package 的變數 **package** 級別

  2. function 級別可用 **block** 級別

  - 規則
    - [x]
      ```go
      var name string = "stanley"
      name:="stanley"
      ```

  3. 沒使用的變數會導致編譯錯誤
  4. 重複宣告變數只存在於 shadowing(function 外的變數，在 function 內可以再宣告一次)
  5. 如果變數沒給值，他的值會是該資料型態的 0 值

     ```go
     var n bool
     fmt.Printf("%v,%T",n,n)
     // false,bool
     ```

  6. go 對資料型態的轉換非常嚴格，例如不同位元的數字相加，go 是不會自動幫你轉換的，你必須自己轉，

     ```go
     s:= "this is a string"
     b:= []byte(s)
     fmt.Printf("%v,%T",b,b)
     ```

# 常數宣告

- 通常宣告在 package level
- immutable , but can be shadowed
- 編譯期間值就必須確定
- 命名規則就和一般變數一樣，大寫字母開頭會被 export
- 讓 go 自己判斷常數型態不像變數需要加冒號
  ```go
    const a = 44
    var b int16 = 27
    // 這邊不會出現型別不同不能相加的錯誤
    fmt.Printf("%v,%T",a+b,a+b)
  ```
- iota

  - 範例

    ```go
      // const block 的宣告方式
      const(
        a = iota
        b = iota
        c = iota
      )
      func main(){
        fmt.Printf("%v\n",a)
        fmt.Printf("%v\n",b)
        fmt.Printf("%v\n",c)
      }
    ```

    和下列結果相同

    ```go
      // const block 的宣告方式,可以讓編譯器幫常數自動累加
      const(
        a = iota
        b
        c
      )
      func main(){
        fmt.Printf("%v\n",a)
        fmt.Printf("%v\n",b)
        fmt.Printf("%v\n",c)
      }
    ```

  - 要注意 iota 從零開始，而其他變數例如 int 的 zero value 是 0，預防方式如下

    ```go
      // 預設第一個元素代表error
      const (
        errorList = iota // 或者 _ = iota，compiler 知道_代表不重要的變數
        catList
        dogList
        snakeList
      )
      func main(){
        var list = int
        // 這樣即使list沒設值也不會有相等的問題
        fmt.Prinft("%v\n",list == catList)
      }
    ```

  - iota 的第一個元素，可以是有規則的公式

    ```go
      // 場景1： 檔案大小
      const (
        KB = 1 << (10*iota)
        MB
        GB
        TB
        PB
        EB
        ZB
        YB
      )
      func main(){
        fileSize := 4000000000;
        fmt.Printf("%2fGB",fileSize/GB) //3.73GB
      }
    ```

    ```go
      // 場景1： 角色管理
      const (
        canCreate = 1 << iota
        canRead
        canUpdate
        canDelete
      )
      func main(){
        var roles byte = canCreate | canRead | canDelete;
        fmt.Printf("%b\n",roles) //1011
        fmt.Printf("Is this role able to create? %v", canCreate & roles == canCreate) //true
      }
    ```

# 命名習慣

- 變數名的長短，反映變數的生命，例如 for 迴圈裡面通常用變數 i
- camelCase 當變數命名有 HTTP 這懂縮寫時，請保持 HTTP 都是大寫 useHttp(X) useHTTP(o)
- 不要用底線\_這些來命名

# 資料型態

- boolean

  - ture or false
  - not an alias for other types e.g 0 或 1 這種 int 型態
  - 預設 zero value 是 false

- numberic

  - intergers
    - signed intergers
      - int (:=預設)不管環境為何，至少會是 32 位元，根據系統不同有可能會是 64 或 128
        ```go
            var n int8;
            // -128~127
        ```
    - unsigned intergers
      - unit16 只有正整數(弟一個字母 u 帶表 unsigned)
        ```go
            var n uint8;
            // 0~255
        ```
    - 預設 zero value is 0
  - floating point numbers
    - float64(:=預設)
    - 預設 zero value is 0
  - complex numbers 用於科學計算

    - complex64(:=預設)
    - 預設 zero value is 0

    ```go
        var n complex128 = 1+2i;
        fmt.Printf("%v,%T",real(n),real(n))
        // 0~255
        fmt.Printf("%v,%T",imag(n),imag(n))
        // -128~127
    ```

  - % mod 預算符只適用於 int

- text

  - string
    - (:=預設) UTF8
    - 用 **+** 來連結字串
    - 可被轉換成[]byte
    - 取字母 取出來的會是 ascii code
      ```go
      s:= "hello"
      fmt.Println(string(s[1])) //e
      ```
  - rune
    - (:=預設) UTF32，注意 UTF32 的字母不一定是 32bits 的長度
    - alias for int32
    - ReadRune 常用來處理上面的 alias

- collection

  - Arrays

    - 陣列直接取**連續**記憶體來劃分，所以存取陣列的效率是很快的
    - 索引從 0 開始
    - 陣列元素的型態都需要一致
    - 固定長度

    ```go
        grades := [3]int{60,70,80};//這裡相當於宣告長度2次
        //所以可以改成
        //grades := [...]int{60,70,80};
        fmt.Printf("Grades: %v",grades)
        // len(grades) 可知陣列長度
    ```

    - 陣列是**傳值**呼叫(這點和 js 不同)

    ```go
      a:= [...]int{1,2,3}
      b:= a
      b[1] = 0
      fmt.Printf(a) // [1 2 3]
      fmt.Printf(b) // [1 0 3]
    ```

    ```go
      // 運用指標來解決陣列複製的資源浪費
      a:= [...]int{1,2,3}
      b:= &a
      b[1] = 0
      fmt.Printf(a) // [1 0 3]
      fmt.Printf(b) // [1 0 3]
    ```

  - Slices

    - 所有 array 有的功能 slice 都有
    - 好處是可以先不用固定長度
    - 陣列是**傳址**呼叫(和 Array 不同)

    ```go
      a:= []int{1,2,3} //和array的差別是中括號裡什麼都不用填
      fmt.Printf(a) // [1 0 3]
    ```

    ```go
      a:= []int{1,2,3,4,5,6} //和array的差別是中括號裡什麼都不用填
      b:= a[:] // 用：來代表切割範圍
      c:= a[3:5]
      fmt.Printf(a) // [1 2 3 4 5 6]
      fmt.Printf(b) // [1 2 3 4 5 6]
      fmt.Printf(c) // [4 5 6]
    ```

    ```go
      a:= make([]int,3,100) //這個function的優點是，如果你要append元素，一開始沒指定slice容量的話，go 會先填滿這個slice，如果容量不夠則會新增一個slice長度的兩倍的slice進行擴充，當容量一大會耗效能，所以能預先知道容量大小填進第三個參數是最好的
      fmt.Printf(a) // [0 0 0]
      // len(a) cap(a)  //長度3容量100，slice 的容量不必和長度相同
    ```

  - Maps

    - 宣告方式
      ```go
        statePopulations := map[string]int{
          "New York":9000425
          "Florida":5905421
          "California":2954248
        }
        // delete(statePopulations,"New York") 刪除元素
      ```
      ```go
        statePopulations := make(map[string]int)
        statePopulations := map[string]int{
          "New York":9000425
          "Florida":5905421
          "California":2954248
        }
      ```
    - 元素的順序並不固定(和 js 一樣)
    - 元素的類型要一樣
    - 如果元素不存在 map，回傳預設值為 0

      ```go
        statePopulations := make(map[string]int)
        statePopulations := map[string]int{
          "New York":9000425
          "Florida":5905421
          "California":2954248
        }
        _,ok := statePopulations["Ohio"] //_是即時這變數沒用到，go也會忽略，ok是習慣命名
        fmt.Println(ok) // 0,false 藉由false可知此元素不存在map中

      ```

    - **傳址**呼叫

  - Struct
  - 宣告

  ```go
    // 大寫的宣告可讓外面看見此結構
    type Doctor struct{
      number int // 小寫的宣告，讓人雖然看到結構，但看不到欄位的名字
      actorName string
      companions [] string
    }
    func main(){
      aDoctor := Doctor{
        number:3,
        actorName:"Jon Petwee"
        companions: []string{
            "Lisa ke",
            "Mansa cheng"
            "donna liu"
        }
      }
    }
  ```

  - positional declaration

  ```go
    func main(){
      // 不建議使用這種實例化的方式，因為一旦有人改了struct宣告結構，會報錯或是改變順序，會錯亂
      aDoctor := Doctor{
        3,
        "Jon Petwee"
        {
            "Lisa ke",
            "Mansa cheng"
            "donna liu"
        }
      }
    }
  ```

  - 用 **.**來存取元素

  ```go
    fmt.Println(aDoctor.actorName)
  ```

  - 匿名 struct

  ```go
    func main(){
      // 通常用在生命週期短的變數  例如你可能要格式化api吐的資料結構
      aDoctor := struct{name string}{name:"John Pertwee"}
      fmt.Println(aDoctor)
    }
  ```

  - **傳值**呼叫，所以要宣告賦值時要注意 struct 是否很大，或者用& pointer
  - embedding go 提供一種叫做 embedded 的語法，來達成組合，和傳統的繼承不同的是 Bird has Animal characteristic 是 has 關係而不是 is 關係，那這種方法不適合在模組化的時候使用，模組化應該要用 interface，這通常用於該資料型態要借助某個功能時
    ```go
      type Animal struct{
        Name string
        Origin string
      }
      type Bird struct {
        Animal
        SpeedKPH float32
        CanFly bool
      }
      func main(){
        b := Bird{}
        b.Name = "Emu"
        b.Origin = "Australia"
        b.SpeedKPH = 48
        b.CanFly = false
        fmt.Println(b.Name)//Emu
      }
    ```
    或是
    ```go
      func main(){
        b := Bird{
          Animal{
            Name :"Emu"
            Origin :"Australia"
          }
          SpeedKPH :48
          CanFly :true
        }
        fmt.Println(b.Name)//Emu
      }
    ```
  - tag 通常用在驗證

    ```go
      import(
        "fmt"
        "reflect"  //為了拿到struct的欄位，必須要用到反射
      )
      type Animal struct{
        Name string `require max:"100"`
        Origin string
      }

      func main(){
        t:=reflect.TypeOf(Animail{})
        field,_ :=t.FieldByName("Name")
        fmt.Println(field.Tag)
      }

    ```

# 流程控制 control flow

- if
  - 一定要有大括號
  ```go
    // pop only existing in if block
    if pop,ok:=statePopulations["Florida"];ok {
      fmt.Printlin(pop)
    }
  ```
- Switch case

  - switching with no tag (非固定值的意思)
    ```go
    func main(){
      i:=10
      switch{
        case i<= 10:
          fmt.Println("less than or equal to ten")
        case i<= 20:
          fmt.Println("less than or equal to twenty")
        default:
          fmt.Println("greater than twenty")
      }
      //less than or equal to ten
    }
    ```
  - 如果要其他語言 falling through 的用法但可以

    ```go
    func main(){
      switch 3{
        case 1,5,10:
          fmt.Println("one")
        case 2,4,6:
          fmt.Println("two")
        default:
          fmt.Println("not one or two")
      }
    }
    ```

  - 或是 fallthrough ，但要小心 fallthrough 不管邏輯都會往下掉
    ```go
    func main(){
      i:=10
      switch{
        case i<= 10:
          fmt.Println("less than or equal to ten")
          fallthrough
        case i<= 20:
          fmt.Println("less than or equal to twenty")
        default:
          fmt.Println("greater than twenty")
      }
      //less than or equal to ten
      //less than or equal to twenty
    }
    ```
  - type switch
    ```go
      var i interface{} = [3]int{}
      switch i.(type) {
        case int :
          fmt.Println("i is an int")
        case float64:
          fmt.Println("i is a float64")
        case string:
          fmt.Println("i is string")
        case [3]int:
          fmt.Println("i is [3]int")
          break //breaking out early
          fmt.Println("other stuff")
        default:
          fmt.Println("i is another type")
      }
      // i is [3]int
    ```

- Looping

  - 迴圈的三個流程，個別只能是一個句子

  ```go
    // 在go i++是句子不是運算子
    for i, j:=0,0; i<5 ;i,j = i+1,j+1{
      fmt.Println(i,j)
    }
  ```

  - loop block
    ```go
      // 宣告在外面
      i := 0;
      for ; i<5 ;i++{
        fmt.Println(i)
      }
      // 才能存取到i
      fmt.Println(i)
    ```
  - 分號也可以省略
    ```go
      i := 0;
      for i<5{
        fmt.Println(i)
        i++
      }
    ```
  - range
    ```go
    s := []int{1,2,3};
    for k,v := range s{
      fmt.Println(k,v)
    }
    ```

- Defer

  - 目的：延遲執行直到回傳之前

  ```go
  func main(){
    fmt.Println("start")
    defer fmt.Println("middle")
    fmt.Println("end")
  }
  ```

  - LIFO(last in first out) 因為通常用在 resource

  ```go
  func main(){
    res,err := http.Get("http://www.google.com/robots.txt")
    if err != nil{
      log.Fatal(err)
    }
    defer res.Body.Close() //因為你可能一直需要用res 這樣就不怕忘記寫這行
    robots,err := ioutil.ReadAll(res.Body)
    if err != nil{
      log.Fatal(err)
    }
    fmt.Println(robots)
  }
  ```

  - defer 會拿到的值是 defer 觸發時當下的值 而不是最後呼叫該 function 的值

  ```go
    a:="start"
    defer fmt.Println(a)
    a = "end"
    // print out start
  ```

- Panic
- 在 go 沒有 exception，因為在很多語言的 exception 在 go 裡面都算正常，例如要打開一個不存在的文件，在 go 是 return error value 而不是 exception

```go
  func main() {
    a,b := 1,0
    ans := a/b
    fmt.Println(ans)
  }
```

- painc 生命週期在執行玩 defer 之後

```go
  func main() {
    fmt.Println("start"))
    defer fmt.Println("this was deffered")
    panic("something bad happend")
    fmt.Println("end")
  }
```

- Recover
  - 用來恢復程式運作的
  - 一定在 defer function 內，因為 panic 之後只會執行 defer

# pointer 指標

- 把他想作是變數的一個附加欄位？裡面放的值一定是記憶體位址
- creating pointer

```go
  func main(){
    var a int =42
    var b *int = &a //一個數字的指標，指向a位址 用&位址運算符拿出位址
    fmt.Println(a,b) // 42 0x1040a124
    fmt.Println(a,b) //這裡的*作用為取出 42 42
  }
```

- pointer arithmetic
  ```go
      func main(){
      a := [3]int{1,2,3}
      b := &a[0]
      c := &a[1]
      fmt.Printf("%v %p %p\n",a,b,c) // [ 1 2 3 ] 0x1040a124 0x1040a128  可知b 和 c元素之間差4個bytes
    }
  ```
  在其他的語言你可能會這樣做
  ```go
      func main(){
      a := [3]int{1,2,3}
      b := &a[0]
      c := &a[1]-4 //go不能這樣做記憶體的運算，因為這樣會讓程式變複雜，如果你真的要這麼做，官方有unsafe package
      fmt.Printf("%v %p %p\n",a,b,c) //error
    }
  ```
- new function
  ```go
    func main(){
      var ms *myStruct
      fmt.Println(ms) // 初始值<nil>
      // ms = &myStruct{foo:42} //&{42} 初始化一個變數指向物件的指標
      ms = new(myStruct) //用new的話沒辦法像上面一樣初始值，所以他的zero value是0 //&{0}
      (*ms).foo = 42 //()是因為.操作符的優先權高，避免取到ms.foo的pointer
      fmt.Println((*ms).foo)// 42
    }
  ```
  然而 go 比其他語言方便的是
  ```go
    func main(){
      var ms *myStruct
      ms = new(myStruct)
      ms.foo = 42
      fmt.Println(ms.foo) //可以省略星號和括號，因為go知道你要取值
    }
  ```

# function

- 最簡單的程式基本構成

```go
  package main
  import(
    "fmt"
  )
  func main(){ //一定要有一個entry point
    fmt.Println("hello world")
  }

```

- 參數 type 一樣可以一起擺在後面

```go
  func greeting(greeting string,name string){
    fmt.Println(greeting,name)
  }
```

可以寫成這樣的語法糖

```go
  func greeting(greeting,name string){
    fmt.Println(greeting,name)
  }
```

- 傳址呼叫

  ```go
  func main(){
    name:="Stacey"
    greeting(&name)
    fmt.Println(name)//Ted
  }
  func greeting(name *string){
    fmt.Println(*name) //Stacey
    *name = "Ted"
    fmt.Println(*name) //Ted
  }
  ```

- 最後一組同類型參數可使用 slice

  ```go
  func main(){
    sum("the sum is ",1,2,3,4,5)
  }
  func sum(msg string,values ...int){
    result := 0
    for _,v := range values{
      result +=v
    }
    fmt.Println(msg,result)
  }
  ```

- 回傳參數要宣告
  ```go
  func sum() int{
    return 12;
  }
  ```
- 回傳指標變數
  ```go
  func sum() *int{
    result := 0
    // 在很多語言並不允許這樣做，因為function執行完的時候，釋放變數，此時你回傳的記憶體位址並不能確定值是正確不變得，但在go 如果他發現你回傳記憶體位址，他會將此變數提昇到share memory ，也就是我們說的heap裡面
    return &result
  }
  ```
- 回傳參數名宣告
  ```go
  //這種方式不常用，因為如果function太長，你還要回來看他回傳什麼，可讀性沒比較號
  func sum() (result int){
    result = 12
    return;
  }
  ```
- 處理例外
  - 回傳例外(通常不這樣做，因為 panic 最好是等到程式執行不下去才用)
    ```go
    func main(){
      d:=divide(5.0,0.0)
      fmt.Println(d)
    }
    func divide(a,b float64) (float64){
      if b==0.0{
        panic("cant provide 0 as second value")
      }
      return a/b
    }
    ```
  - 常用的方式，回傳錯誤值
    ```go
    func main(){
      d,err :=divide(5.0,0.0)
      if err != nil{
        fmt.Println(err)
        return
      }
      fmt.Println(d)
    }
    func divide(a,b float64) (float64, error){
      if b==0.0 {
      return 0.0,fmt.Errorf("cant divide by 0")
      }
      return a/b,nil
    }
    ```
- 匿名函式
  ```go
  func main(){
    // 常用在非同步執行時,scope的區隔
    for i:=0;i<5;i++{
      func(i int){
        fmt.Println(i)
      }(i)
    }
  }
  ```
- 宣告函數類型的變數
  ```go
    f:=func(){
      fmt.Println("hello")
    }
  ```
  ```go
    var f func() : =func(){
      fmt.Println("hello")
    }
  ```
  ```go
    var divide func (float64,float64) (float64,error)
    divide = func(a,b float64) (float64,error){
      return a/b
    }
  ```
- 方法(可以依附在任何的資料型態上面，有點像 js)
  ```go
    func main(){
      g:=greeter{
        name:="stanley"
      }
      g.greet() //stanley
    }
    type greeter stuct{
      name string
    }
    func (g greeter) greet(){
      fmt.Println("hello",g.name)
    }
  ```

# Interfaces

- 命名習慣 通常是動詞加上 er
- struct 裡面會放資料

  ```go
  func main(){
    //宣告 Writer 類型的變數
    var w Writer = ConsoleWriter{}//達成多型
    w.Write([]byte) (int ,error)
  }
  type Writer interface {
    Write([]byte) (int,error)
  }
  type ConsoleWriter struct{}

  // 這作法很強大，因為你不需要在設計接端就抽出個interface，因為你隨時都可以加個方法進一個資料型態，只要你可以加方法進去，就代表隨時都可以加個interface
  // 將Write依附在cw，讓ConsoleWriter有write這個方法
  func (cw ConsoleWriter) Write(data []byte) (int,error){
    n,err := fmt,Println(string(data))
    return n,err
  }
  ```

- embeding interface(組合 interface)

  ```go
    import{
      "fmt"
      "bytes"
    }
    func main(){
      var wc WriterCloser = NewBufferWritterCloser()
      wc.Write([]byte("Hello my name is stanley"))
      wc.Close()
      bwc := wc.(*BufferedWriterCloser) //type conversion 用這語法可轉型態
      fmt.Println(bwc)
    }
    type Writer interface {
      Write([]byte) (int,error)
    }
    type Closer interface {
      Close() error //return error
    }
    type WriteCloser interface {
      Writer
      Closer
    }
    type BufferedWriteClose struct {
      buffer *bytes.Buffer
    }
    func (bwc *BufferedWriterCloser) Write(data []byte) (int,error){
      n,err := bwc.buffer.Write(data)
      if err != nil{
        return 0,err
      }
      v:=make([]byte,8)
      for bwc.buffer.Len()>8{
        _,err := bwc.buffer.Read(v)
        if err != nil{
          return 0,err
        }
        _,err=fmt.Println(string(v))
        if err != nil{
          return 0,err
        }
      }
      return n,nil
    }
    func (bwc *BufferedWriteCloser) Close() error {
      for bwc.buffer.Len()>0{
        data:=bwc.buffer.Next(8)
        _,err := fmt.Println(string(data))
        if err !=nil {
          return
        }
      }
      return nil
    }

    func NewBufferWriterCloser() *BufferWriterCloser{
      return &BufferedWriterCloser{
        buffer:bytes.NewBuffer([]bytes{})
      }
    }

  ```

- 類型轉換

  - empty interface 沒有方法的 interface，所以的類型都實作這個界面，有點像是 any？

  ```go
    // 然而實際上這界面沒有方法，所以你要不就是利用反射，要不就是要做類型轉換，才能知道你用的是什物件，才能運用這個物件
    var myObj interface{} = NewBufferedWriterCloser()
    if wc,ok := myObj.(WriterCloser);ok{
      wc.Write([]byte("Hello my name is stanley"))
      wc.Close()
    }

  ```

  - type switches 在 switch 的章節有講到，需要用到 empty interface

  ```go
    var i interface{} = 0
    switch i.(type){
      case int :
        fmt.Println("i is an int")
      case float64:
        fmt.Println("i is a float64")
      case string:
        fmt.Println("i is string")
      case [3]int:
        fmt.Println("i is [3]int")
        break //breaking out early
        fmt.Println("other stuff")
      default:
        fmt.Println("i is another type")
    }

  ```

- 實例值和指標

  ```go
    import{
      "fmt"
      "bytes"
    }
    func main(){
      var wc WriterCloser = &myWritterCloser() // 加上& 其method就可以是value reciver 或 pointer reciver，但如我們所說，操作pointer 的時候要小心改到資料
      fmt.Println(wc)
    }
    type Writer interface {
      Write([]byte) (int,error)
    }
    type Closer interface {
      Close() error //return error
    }
    type WriteCloser interface {
      Writer
      Closer
    }
    type myWriteClose struct { }

    func (mwc *myWriteClose) Write(data []byte) (int,error){
      return n,nil
    }
    func (mwc *myWriteClose) Close() error {
      return nil
    }

  ```

- 最佳實踐 //todo 以下筆記需要實作後才能領悟，所以目前的筆記有可能是理解錯誤的
  - 使用多而小的 interface
  - 不要將會被使用類型的界面匯出，讓使用者自訂界面，讓他們選想要的方法
  - 將會被套件使用到的類型界面匯出 因為 go 可延遲界面的實作，所以讓使用者實例你的界面，拿到你需要的類型
  - 製作接收使用者自訂界面的 function

# 併發

在多數的語言裡面，開線程是相當耗費資源的，但對 go 來講並不會，你甚至可以看到有上千個線程的進程在跑，這在別種語言幾乎是不可能的，因為別種語言的線程是建立在 os 之上，每個線程幾乎都要佔 1mb 的堆，然而 go 則是利用現有的線程，看哪個有空用哪個，所以省資源

- go routine

```go
 func main(){
   var msg = "hello"
   go func (){
     fmt.Println(msg)
   }()
   msg = "goodbye"
   time.Sleep(100*time.Millisecond)// 因為go routine會來不急執行，所以加這句
 }
 // goodbye  你可以看出，即便線程是在另一個堆，closure可以仍取到外面的變數，但是這樣有個缺點，this is what we call race condition，你可以發現go routine在sleep之後執行
 // go run -race src/main.go 可以檢查是否有race condition
```

```go
 func main(){
   var msg = "hello"
   go func (msg string){
     fmt.Println(msg)
   }(msg)
   msg = "goodbye"
   time.Sleep(100*time.Millisecond)
 }
 // hello  在這裡我們複製了一份，利用傳值的方式，做解偶
```

- waitgroup
  - 同步兩個線程
    ```go
    var wg = sync.WaitGroup{}
    func main(){
      var msg = "hello"
      wg.Add(1)
      go func (msg string){
        fmt.Println(msg)
        wg.Done() //會自動減1，讓go知道，沒有其他線程
      }(msg)
      msg = "goodbye"
      wg.Wait()
    }
    // hello  在這裡我們複製了一份，利用傳值的方式，做解偶
    ```
- mutex

  - 同步多個線程

    ```go
    func main(){
      runtime.GOMAXPROC(100)
      fmt.Printf("Threads:%v\n",runtime.GOMAXPROCS(-1))//印出上一個設定線程的數量，在預設上，go會預設以電腦cup核心數目相同的線程
    }
    ```

    ```go
      import(
        "fmt"
        "sync"
        "runtime"
      )
      var wg = sync.WaitGroup{}
      var counter = 0
      var m = sync.RWMutex{}
      func main(){
        runtime.GOMAXPROCS(100)
        for i :=0;i<10;i++{
          wg,Add(2)
          m.RLock()
          go sayHello()
          m.Lock()
          go increment()
        }
        wg.Wait()
      }
      func sayHello(){
        fmt.Printf("Hello #%v\n",counter)
        m.RUnlock()
        wg.Done()
      }
      func increment(){
        counter++
        m.Unlock()
        wg.Done()
      }
    // 以上所有的作法都讓go routine的好處消失，變成單線程的程式，這樣倒不如直接不用go routine，一般來說多線程也都是在處理不同的事，然而萬一真的有共享的資料，mutex就是用在這時候
    ```

# Channel

- 可以解決上述資料共用的混亂

```go
  var wg = sync.WaitGroup{}
  func main(){
    ch:=make(chan int) // 初始化channel所能接收或發出的資料型態
    wg.Add(2)
    go func(ch <-chan int){ //reciving data
      i:=<- ch
      fmt.Println(i)
      wg.Done()
    }(ch)
    go func(ch chan<- int){ // sendgin data
      ch<-42
      wg.Done()
    }(ch)
    wg.Wait()

  }
```

```go
  var wg = sync.WaitGroup{}
  func main(){
    ch:=make(chan int)
    for j:=0;j<5;j++{
      wg.Add(2)
      go func(){
        i:=<- ch
        fmt.Println(i)
        wg.Done()
      }()
      go func(){
        ch<-42
        wg.Done()
      }()
    }
    wg.Wait()

  }
```

- close channel

  ```go
    var wg = sync.WaitGroup{}
    func main(){
      ch:=make(chan int,50) // 初始化channel所能接收或發出的資料型態
      wg.Add(2)
      go func(ch <-chan int){ //reciving data
        for {
          if i,ok :=<-ch;ok{
            fmt.Println(i)
          }else{
            break
          }
        }
        wg.Done()
      }(ch)
      go func(ch chan<- int){ // sendgin data
        ch<-42
        ch<-27
        close(ch)
        wg.Done()
      }(ch)
      wg.Wait()

    }
  ```

- select

  ```go
    const {
      logInfo = "INFO"
      logWarning = "WARNING"
      logError = "ERROR"
    }
    type logEntry struct{
      time time.Time
      severity string
      message string
    }
    var logCh = make(chan logEntry,50)
    func main(){
      go logger()
      defer func(){
        close(logCh)
      }()
      logCh <- logEntry{
        time.Now(),
        logInfo,
        "App is starting"
      }
      logCh <- logEntry{
        time.Now(),
        logInfo,
        "App is shutting down"
      }
      time.Sleep(100*time.Millisecond)
    }
    func logger(){
      for entry := range logCh{
        fmt.Printf("%v - [%v]%v\n",entry.time.Format("2006-01-02",entry.severity,entry.message))
      }
    }
  ```

  ```go
    const {
      logInfo = "INFO"
      logWarning = "WARNING"
      logError = "ERROR"
    }
    type logEntry struct{
      time time.Time
      severity string
      message string
    }
    var logCh = make(chan logEntry,50)
    var doneCh = make(chan struct{})
    func main(){
      go logger()
      logCh <- logEntry{
        time.Now(),
        logInfo,
        "App is starting"
      }
      logCh <- logEntry{
        time.Now(),
        logInfo,
        "App is shutting down"
      }
      time.Sleep(100*time.Millisecond)
      doneCh <- struct{}{}
    }
    func logger(){
      for {
        select{ // 整個段落會阻塞，直到我們收到需要的channel
          case entry := <-logCh:
            fmt.Printf("%v - [%v]%v\n",entry.time.Format("2006-01-02",entry.severity,entry.message))
          case <-doneCh:
            break;
        }
      }
    }

  ```
