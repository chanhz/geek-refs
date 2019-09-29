# go 设计模式
## 创建模式

### Builder Pattern
将一个复杂对象的构建与它的表示分离, 使得同样的构建过程可以创建不同的表示.
通俗来讲，例如，创造一辆小轿车，需要 4 个轮子、1个方向盘、4 个座位、汽油发动机；创造一辆摩托车，需要 2 个轮子、0 个方向盘，1 排座位、汽油马达。现在有一种强大的构造器，既可以构造小轿车，又能构造摩托车，此时，同样的构建过程，通过不通的构造模型和参数，可以构造出不同表示的对象。


### Factory Method
使一个类的实例化延迟到其子类, 定义一个用于创建对象的接口, 让子类决定将哪一个类实例化。

例如，购物时扣款方式可以选择现金余额或信用卡额度支付，付款时，先定义一个扣款接口，再实现现金支付和信用卡支付的函数，实现该支付的接口。

### Object Pool 对象/线程池
根据需求将预测的对象保存到channel中， 用于对象的生成成本大于维持成本。
将任务使用一个对象来描述，并将其塞到一个队列中，消费端从队列中消费，处理任务


### singleton 单例
单例模式是最简单的设计模式之一, 保证一个类仅有一个实例, 并提供一个全局的访问接口

Golang 中使用 sync.Once 原语来 Do(初始化)，保证初始化只进行一次。



### 生成器模式 Generator
生成器模式可以允许使用者在生成要使用的下一个值时与生成器并行运行

和 Python 的 `yield` 格外相似，  
```go
func Count(start, end int) <-chan int {
	ch := make(chan int)

	go func(ch chan int) {
		for i := start; i <= end; i ++ {
			ch <- i
		}
		close(ch)
	}(ch)

	return ch
}
```

### 抽象工厂模式 Abstract Factory

提供一个创建一系列相关或相互依赖对象的接口, 而无需指定它们具体的类，而由类实现接口相应的方法。


### 原型模式 Prototype Pattern

复制一个已存在的实例(?这也算模式?)

## 结构模式

### 装饰模式 Decorator Pattern

装饰模式**使用对象组合的方式动态改变或增加对象行为**， 在原对象的基础上增加功能.

换句话说，装饰模式就是将对象组合，并借助组合对象实现的方法来表达新对象的行为，是一种结构体行为方法复用的设计模式。


### 代理模式 Proxy Pattern

代理模式用于**延迟处理操作或者在进行实际操作前后对真实对象**进行其它处理。

具体做法：
创建一个代理结构体，来包裹真实对象，对真实对象的操作进行拦截和特殊处理。

### 适配器模式

**设计思想**:
1. 目标接口（示例中的Player）
2. 被适配者
3. 核心是通过适配器Adapter转换为目标接口（组合的方式包含被适配者）

>If the Target and Adaptee has similarities then the adapter has just to delegate
the requests from the Target to the Adaptee.
If Target and Adaptee are not similar, then the adapter might have to convert the
data structures between those and to implement the operations required by the Target
but not implemented by the Adaptee.

```go
import "fmt"

//音乐播放器
type Player interface {
	PlayMusic()
}

type MusicPlayer struct {
	Src string
}

func (music *MusicPlayer) PlayMusic() {
	fmt.Println("play music: " + music.Src)
}

//对外接口
func Play(player Player) {
	player.PlayMusic()
}

//在音乐播放基础上实现游戏播放
//定义游戏结构体
type GamePlayer struct {
	Src string
}

//game的方法
func (game *GamePlayer) PlaySound() {
	fmt.Println("play sound: " + game.Src)
}

//这里要实现调用play方法的时候，实现GamePlayer的播放
//通过组合的方式，声明一个Game的适配器
type GamePlayerAdapter struct {
	Game GamePlayer
}
//继承Player interface, 调用GamePlayer的方法
func (game *GamePlayerAdapter) PlayMusic() {
	game.Game.PlaySound()
}
```
个人理解认为适配器模式有点多此一举，直接调用 GamePlayer 的 PlaySound 方法会更为简洁易懂。

## 组合模式

组合模式有助于表达数据结构, 将对象组合成树形结构以表示"部分-整体"的层次结构, 常用于树状的结构。

## 享元模式 Flyweight

把多个实例对象共同需要的数据，独立出一个享元，从而减少对象数量和节省内存。
与单例模式相仿，但单例模式强调全局单例，享元模式多用于局部函数。相应的，使用适当使用指针传递参数，也能减少内存复制的开销。

## 外观模式 Facade
设计思想:通过组合模式来实现外观模式, 为子系统实现统一的访问 api
	
```go
//示例：微服务框架： 音乐服务、视频服务
//创建子服务struct
//music struct
type Music struct {
	Name string
}

func (m *Music) GetMusic() string {
	return m.Name
}

//Video struct
type Video struct {
	Id int64
}

func (v *Video) GetVideoId() int64 {
	return v.Id
}

//count struct
type Count struct {
	Comment int64
	Praise 	int64
	Collect int64
}

func (c *Count) GetComment() int64 {
	return c.Comment
}

//外观结构Facade
type Facade struct {
	music Music
	count Count
	video Video
}

//对外访问接口
func (f *Facade) PrintServerInfo() {
	f.music.GetMusic()
	f.video.GetVideoId()
	f.count.GetComment()
}

func NewFacade(music Music, count Count, video Video) *Facade {
	return &Facade{
		music: music,
		video: video,
		count: count,
	}
}
```

### 桥接模式
桥接模式分离抽象部分和实现部分，使得两部分可以独立扩展。



Refs:
- [github.com/sevenelevenlee:Golang设计模式](https://github.com/sevenelevenlee/go-patterns)