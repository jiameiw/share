# vue 中@Action 的异常处理

#### 1. 知识点总结：

1. 在某行代码有错误抛出后，后面的代码都不会被执行，直接阻断
2. try-catch 语句里的代码有一行有错误，后面也阻断，但是跳出 catch 语句后面的代码依然可以执行，因为抛出的错误被 catch 接到了

- 举个栗子：

```
 try {
    test()
    console.log('after-test');
  } catch (e) {
    console.log('catch-error', e);
  }
  console.log('after-trycatch');
  function test() {
    throw new Error()
  }
```

- 输出：
  ![image.png](https://upload-images.jianshu.io/upload_images/6123292-15707179ac1121f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2. 问题来源

- **测试 bug**：断网时提示不正确，应该提示网络出错，但是却显示某一接口的获取失败

- **具体代码如下：**

```
//页面请求接口
try {
      await this.fetchRecommendedQuestions(params)
    } catch (e) {
      if (e && e.isHandled) {
        return
      }
      this.$message.error('获取变式题失败')
    }

//store里请求数据
 @Action({ rawError: true })
  async fetchRecommendedQuestions(
    params: RecommendedQuestionsParams,
  ): Promise<RecommendedQuestions[]> {
    const url = shouyueApi.variantQuestion.recommendedQuestions()
    const response = await axios.post(url, params)
    const { data } = response
    this.context.commit('setRecommendedQuestions', {
      type: params.recommendedType,
      data,
    })
    return data
  }
```

- **原因：**`e.isHandled`为 false，也即此时 catch 住的 error 并非项目里由 axios 抛出的 requestError 类型（该类型包含 isHandled 属性）

- **打印 error：**抛出两个异常，一个是`ERR_ACTION_ACCESS_UNDEFINED`，一个是`error:无法链接或出错`。其中第二个是 requestError 类型的。但是基于上面**知识点 1**中所说的代码里 throw error，后面的代码应该不执行的，但是却执行了 return data，从而导致 mutation 报错，说明@Action 做了一些工作。

- **分析源码：** 那么页面 catch 住的是什么 error 呢，查看`vuex-module-decorators`中关于`@Action`的源码：
  ![image.png](https://upload-images.jianshu.io/upload_images/6123292-d6cb07a076df5190.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可知：@Action 在不给 rawError 置为 true 时，会 new 一个新的 Error，并且带上 aciton 里抛出的错误，包裹到一起抛出，因此页面 catch 住的 error 是@Action 抛出的错误，打印就会看到原 error 的信息

- **查错：** 查询第一个报错可参考https://github.com/championswimmer/vuex-module-decorators/issues/26

- **解决方法：** 加`rawError`属性，`@Action({rawError:true})`，即暴露出原生 error。

- **走的弯路：** 因为打印 error 会有原 error 也就是 requestError 类型出现，误以为页面 catch 住了两个 error，归结原因在于对**知识点 1**不够清晰。

### 3. 对当前项目的启示

需要成功获取接口数据后才更改某一状态或者某一数据值的，不应该将代码写在 trycatch 后，因为即使抛出错误，catch 后的代码也会被执行，所以对于这种代码可以写在 try 语句里，await 后。
比如：批改页面打 tag，需要在成功将 tag 传给后台成功后再变为选中的蓝色状态，以及做后面的操作。

参考：[https://github.com/championswimmer/vuex-module-decorators/issues/26](https://github.com/championswimmer/vuex-module-decorators/issues/26)

[https://github.com/championswimmer/vuex-module-decorators/blob/master/src/action.ts](https://github.com/championswimmer/vuex-module-decorators/blob/master/src/action.ts)
