## 单例模式

单例模式：使程序运行期间只存在一个实例对象

### 代码实现

#### Double-Checked Locking 模式

```go
var (
	singleInstance *Single
	lock           sync.Mutex
)

type Single struct {
	Name string
}

func GetInstance(name string) *Single {
	if singleInstance == nil {
		lock.Lock()
		defer lock.Unlock()
		if singleInstance == nil {
			singleInstance = &Single{Name: name}
		}
	}
	return singleInstance
}
```
##### 为什么有两次判空检查？

1. 第一次检查 (without lock)：在没有获取锁之前，首先检查 singleInstance 是否为 nil。如果 singleInstance 不为 nil，那么意味着已经有一个实例被创建，此时不需要获取锁，直接返回现有的实例。这可以提高性能，因为在大多数情况下，只有第一次需要获取锁来创建实例。
2. 第二次检查 (with lock)：如果 singleInstance 为 nil，表示没有实例被创建，此时获取锁以进入临界区。然后再次检查 singleInstance 是否为 nil，因为在获取锁之前可能有其他线程进入了临界区。如果在获取锁之前已经有一个线程创建了实例，那么当前线程就不再创建实例，而是返回已创建的实例。

#### sync.Once

```go
var (
	singleInstance *Single
	once           sync.Once
)

type Single struct {
	Name string
}

// sync.Once
func GetInstance(name string) *Single {
	if singleInstance == nil {
		once.Do(func() {
			singleInstance = &Single{
				Name: name,
			}
		})
	}
	return singleInstance
}
```

> `once.Do` 来保证 某个对象只会初始化一次，有一点要要注意的是这个 once.Do 只会被运行一次，哪怕 Do 函数里面的发生了异常，对象初始化失败了，这个 Do 函数也不会被再次执行了



使用场景：

公司在项目中的应用

```go
// dao(repository)层/xxx-repo.go
type xxxRepo struct {
	repo.RepositoryInterface
	*xxxRepoFields
}

var (
	xxxRepoInstance *xxxRepo
	xxxRepoOnce     sync.Once
)

func GetxxxRepo() *xxxRepo {
	xxxRepoOnce.Do(func() {
		repoImpl := repo.NewRepository(new(BrandToolboxTypeface)) // 仓库实现类
		// repoImpl.Tm = tmMango
		repoImpl.Tm = pgsql.NewTransactionManager("xxx", "xxx")
		repoFields := new(xxxRepoFields)
		repoImpl.InitRepoFields(repoFields)
		xxxRepoInstance = &xxxRepo{
			RepositoryInterface:	repoImpl,
			xxxRepoFields:          repoFields,
		}
	})
	return xxxRepoInstance
}
```





### 饿汉式、懒汉式

饿汉模式：是指在程序启动时就进行数据加载，这样避免了数据冲突，也是线程安全的，但是这可能会造成内存浪费。比如在程序启动时就构建 Config 对象，加载配置信息，但是如果全局都没有用到 Config 对象，就会造成内存浪费。

懒汉模式：是指程序需要 config 对象时，再主动去加载数据，这样做可以避免内存的浪费，比如当需要调用 Config 对象获取数据时，再去 new 一个 Config 对象，然后通过对象获取配置相关信息。但是，懒汉式并不是线程安全的，如果多个线程同时调用 getInstance()，有可能会创建多个实例。



## 工厂模式

工厂模式：定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。（通过定义抽象接口来决定并生成特定类型对象）

优点：

- 灵活性与解耦程度提升：工厂方法避免直接构造对象，增强了系统灵活性和松耦合特性。
- 维护效率优化：变更创建逻辑仅需调整工厂方法内部代码，简化维护工作。
- 鼓励复用与提高清晰度：该模式有利于代码复用，结构更加清晰，利于团队协作。



```go
type IService interface {
	Get(id int) string
}

type ServiceFactory struct {
}

func (sf *ServiceFactory) Create(name string) IService {
	switch name {
	case "one":
		return &IServiceOne{}
	case "two":
		return &IServiceTwo{}
	default:
		return nil
	}
}

func NewServiceFactory() *ServiceFactory {
	return &ServiceFactory{}
}

type IServiceOne struct {
}

func (one *IServiceOne) Get(id int) string {
	return "单挑新闻内容one"
}

type IServiceTwo struct {
}

func (two *IServiceTwo) Get(id int) string {
	return "单挑新闻内容two"
}
```



使用场景：

数据源适配器工厂

```go
type DataSource interface {
	Connect() error
}
type DataSourceFactory struct{}

func (f *DataSourceFactory) CreateDataSource(sourceType string) DataSource {
	switch sourceType {
	case "MySQL":
		return &MySQLAdapter{}
	case "PostgreSQL":
		return &PostgreSQLAdapter{}
	default:
		return nil
	}
}
```

公司在项目中的应用：根据合规检测类型创建特定类型对象

```go
type IPipelineTypeFactory interface {
	Execute(ctx context.Context, pipeLog *pipeline.DocPipelineLog) error
}

func NewPipelineTypeFactory(pipelineType detect_proto.PipelineType) (IPipelineTypeFactory, error) {
   switch pipelineType {
   case detect_proto.PipelineType_PIPELINE_AI_TAG:
      return &VideoSmartTag{}, nil
   case detect_proto.PipelineType_PIPELINE_ASR:
      return &VideoAsr{}, nil
   case detect_proto.PipelineType_PIPELINE_CONTENT_TAG:
      return &CvContentTaggingObtain{}, nil
   case detect_proto.PipelineType_PIPELINE_IMG_REPEAT_SCAN:
      return &ImageRepeat{}, nil
   case detect_proto.PipelineType_PIPELINE_EVALUATE_IMAGE_SCREENSHOT_DETECT:
      return &ImageScreenShot{}, nil
   case detect_proto.PipelineType_PIPELINE_OCR, detect_proto.PipelineType_PIPELINE_EVALUATE_IMAGE_COMPLIANCE_DETECT:
      return &ImageCompliance{}, nil
   case detect_proto.PipelineType_PIPELINE_EVALUATE_IMAGE_WATERMARK_DETECT:
      return &ImageWaterMark{}, nil
   case detect_proto.PipelineType_PIPELINE_EVALUATE_VIDEO_WATERMARK_DETECT:
      return &VideoWatermark{}, nil
   case detect_proto.PipelineType_PIPELINE_EVALUATE_VIDEO_FRAME_COMPLIANCE_DETECT:
      return &VideoFrameCompliance{}, nil
   }
   return nil, errors.New("invalid pipelineType")
}
```

