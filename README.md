
# 库安装


首先，安装 Mock 类生成工具 Mockery：



```
go install github.com/vektra/mockery/v2@v2.45.1

```

*实际上，你也可以手动创建 Mock 类。*


# 生成 Mock 类


假设你在 `internal/metrics` 包下有如下定义的接口：



```
package metrics

type Getter[T any] interface {
    Get() (T, error)
}

```

在项目根目录，可以使用以下命令生成 Mock 类：



```
mockery --name=Getter --dir=internal/metrics

```

生成的 Mock 类会在 `mocks` 目录下的 `getter.go` 文件中。


# 编写用例



```
package metrics

import (
	"testing"

	mocks "xxx/mock/internal_/metrics"
	"github.com/stretchr/testify/suite"
)

type GetterTestSuite struct {
	suite.Suite
}

func TestGetter(t *testing.T) {
	suite.Run(t, new(GetterTestSuite))
}

func (t *GetterTestSuite) TestGetterInt() {
	t.T().Logf("TestGetterInt run")
	getter := new(mocks.Getter[int])
	getter.On("Get").Return(1, nil)

	val, err := getter.Get()
	t.Nil(err)
	t.Equal(1, val)
}


```

**说明**：


1. **GetterTestSuite** 是测试集的名称，**每个method**都会作为测试用例调用。**TestGetter** 函数运行时，会调用 **TestGetterInt**。
2. **TestGetterInt** 中引用的 `t` 是 `TestSuite`，包含许多有用的断言函数，如 `Equal` 和 `Nil` 等。
3. 创建 Mock 实例后，可以使用 `On` 方法来标记方法对应的返回值。假设 `Get` 方法可以传递参数，则可以根据不同的参数选择不同的返回值。


**Mock 常见用法**：


假设 `mockObj` 是 Mock 类的实例：


1. `mockObj.On("GetApiKey", mock.Anything).Return("dummy_api_key")`：`GetApiKey` 有一个参数，且无论传什么，都会返回 `dummy_api_key`。
2. `mockObj.On("GetAllClusterInfo").Maybe().Return(GenerateTestClustersInfo())`：如果使用 `Maybe`，则 `GetAllClusterInfo` 不一定必须被调用；如未使用 `Maybe` 且函数未被调用，则断言将失败。
3. `mockObj.On("RunCleanup", true, true).Once().Return(nil, nil)`：`RunCleanup` 有两个参数，所以需要传递两个 Mock 的值进入。`Once` 表示这个函数只应该被调用一次。
4. `mockObj.AssertNumberOfCalls(t.T(), "RunCleanup", 4)`：可以检查方法的调用次数。


通过这些用法，用户可以完全控制 Mock 类的每个方法的行为，并进行一些检查以完善整个测试。


 本博客参考[飞数机场](https://ze16.com)。转载请注明出处！
