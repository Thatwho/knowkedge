# goroutine
写并发程序的时候，最佳做法是，在`main`函数返回前，清理并终止所有之前启动的`goroutine`。编写启动和终止时的状态都很清晰的程序，有助减少bug，防止资源异常。
`WaitGroup`是一个计数信号量，我们可以利用它来统计所有的`goroutine`是不是都完成了工作。
例：
```go
func Run(searchTerm string) {
    // 获取数据源列表
    feeds, err := RetrieveFeeds()
    // 如果返回错误，则输出异常，并终止程序
    if err != nil {
        log.Fatal(err)
    }

    // 创建无缓冲的通道，用于接受匹配的结果
    results := make(chan *Result)

    // wait group用于在所有数据源处理完成之前，防止主程序退出
    // WaitGroup是一个计数信号量，可以用于统计所有的go程是否都完成工作
    var waitGroup sync.WaitGroup

    // 需要等待处理的每个数据源的协程数量
    waitGroup.Add(len(feeds))

    // 为每个数据源启动一个goroutine，查找结果
    for _, feed := range feeds {
        // 获取匹配器用于查找
        matcher, exists := matchers[feed.Type]
        if !exists {
            matcher = matchers["default"]
        }

        // 启动goroutine执行搜索
        go func(matcher Matcher, feed *Feed) {
            Match(matcher, feed, searchTerm, results)
            waitGroup.Done()
        }(matcher, feed)
    }

    // 启动独立的goroutine，监控是否所有的工作都已经完成
    go func() {
        // 等待所有的任务完成
        waitGroup.Wait()

        // 关闭通道
        close(results)
    }()

    // 展示结果，并在所有结果展示完后返回
    Display(results)
}
```

如果不需要维护状态，那么可以定义一个空结构体，并实现相关接口方法，因为不需要维护状态，所以定义接口时使用类型的值作为接收者即可。e.g.
```go
// defaultMatcher 实现默认匹配器
type defaultMatcher struct{}

// Search implements the behavior for the default matcher.
func (m defaultMatcher) Search(feed *Feed, searchTerm string) ([]*Result, error) {
	return nil, nil
}
```

go 的导入路径：假如Go安装在`/usr/local/go`，环境变量`GOPATH`为`/home/myproject:/home/mylibraries`，那么编译器会按一下顺序查找`net/http`包：
```
/usr/local/go/src/pkg/net/http 
/home/myproject/src/net/http
/home/mylibraries/src/net/http
```
一旦找到一个满足import语句的包，就会停止查找。

# go的类型系统
## 内置类型
* 数值类型
* 字符串类型
* 布尔类型

## 引用类型
* slice
* map
* channel
* interface
* func

声明引用类型时创建的是header变量。header包含指向底层数据结构的指针，以及一组管理底层数据结构的字段。header是为了复制而设计的，不需要共享引用类型的值，复制header的值就是在共享底层数据结构。
如果一种了类型的值其本质是非原始类型，那么应该使用共享，即返回指针的方式传递这个值。

## 接口
接口的值是两个字长度的数据结构，第一个字长度是iTable中的指针，第二个字存放的是指向实际值的指针。e.g.
```go
type notifier interface {
    notify(){}
}

type person struct {
    name string
}

func(person) notify() {}

func main() {
    var n notifier
    n = person("lalala")
}
```
变量n是接口`notifier`值，由两个字长，第一个字长指向iTable中`person`类型的定义，这个定义包含person类型的类型信息以及定义的一组方法。第二个字长指向`person("lalala")`实例的内存位置。

编译器并不是总能获得一个值的地址，所以使用如果接口方法是通过指针实现的，那么只能把该类型的指针作为相应的接口值传递。因为传递值的时候，编译器不能获得该值的指针。

