### 关于kernel需要配备的第三方库：
- 需要numpy, pandas, statsmodels, seaborn, matplotlib

### 关于回测系统的使用：
- 原则上用户只需要引入BackTest模块，这个模块中提供了两个回测类，命名为Single_BackTest和 Multiple_BackTest，您应当使用这两个类创建实例，然后使用这些实例的fit方法或者multiple_signals_fit方法载入您的信号，在使用show，LnS_PnL_show方法查看结果。
- 如果上面这个流程让您觉得有些抽象，请查看 ./BackTest/sample.ipynb 这个文件给出了相对完整的使用示例。
- 原则上，Multiple_BackTest继承了单因子回测类的所有属性和方法，因此您使用Multiple_BackTest类做单因子回测也是可行的，但是并不推荐。
- 用户可以使用Demos模块提供的示例因子查看一个本系统可接受的标准因子输入是什么样子的（见下面具体介绍的方法）。为了本系统的运行效率，我们对接受的因子格式有严格的要求（事实上，本系统有专门的检验函数，在您进行fit前，系统会首先检验您的因子格式），如果不符合格式，将会打印一个错误信息。
- 目前Demos里提供了两个生成标准因子的函数，分别是get_MACD_signal, get_reversal_signal

### 关于回测的结果：
- 这个包最小平仓间隔为1天，换言之，不支持日内策略的回测。原因在于这个包使用的示例数据是日级别的HLOC，没有日内数据。
- 如果您不知道回测函数的标准输入，请 from BackTest.Demos import standard_input_demo，然后调用这个方法，他会返回一个示例df，将您的信号merge到这个demo上（on = ['stk_id', 'date']），drop掉第三列（demo自带的signal）再回测即可。
- 我们计算总收益的时候，是(1+每日收益率)的累乘
- 夏普永远是日度的，需要年化夏普请自己调成一下（乘以\sqrt{252}），每当我们提到年化时，都是按一年252个交易日近似计算的
- 因为singal数值有些时候可能出现极端偏态的分布，我们计算Pnl固定会在每日做一个rank，这也就是说，把您的信号翻倍，并不会改变任何回测结果。

### 关于提供的回测方法：
- 总体而言，单因子系统和多因子系统都提供了三类回测：
- 第一类是没有限制的正常回测
- 第二类是按照预测值大小分组的”多空头分组回测“
- 第三类是给定一个股票池的股票池内回测
- 这三类具体都是些什么，请阅读中期报告文档；如何使用这三类，请参考./BackTest/sample.ipynb中的示例。

### 关于拟合后的属性：
- 我们推荐您使用show或LnS_PnL_show方法，这两个方法会可视化收益曲线，并自动打印关键指标。
- 但您也可以对fit后的示例直接调用属性来查看这些属性。每种实例具体有哪些属性，请参考中期报告文档，文档里有的属性全部都实现了。

### 关于中期报告文档：
- 中期报告文档里唯一没有得以实现的是Optimizers模块里的动量优化器，因为我想了想觉得这个东西没什么用，所以在Multiple_BackTest里，默认使用线性模型组合因子
- 但Multiple_BackTest里确实支持你传入一个自己的函数，然后用它来组合因子，但这会有点复杂。

### 其它：
##### 关于基础要求：
- 作为课程任务（实现在【任何时间区间】回测【n】日反转策略），我们详细说说get_reversal_signal这个方法：
- 它的参数细节是这样：n:int = 5, start_time: pd.Timestamp = pd.Timestamp('2020-01-02 00:00:00'), end_time: pd.Timestamp = pd.Timestamp('2022-12-29 00:00:00'), Stock_list:list or str = 'all'
- 这个函数的提供了document解释这些参数，但直观的说，您可以提供n，start_time, end_time来实验基础要求
- 但是，您也发现了，这些参数的类型是被严格限制死的，所以要注意使用pd.Timestamp类型的起始与终止日期。./BackTest/sample.ipynb最后给出了一个示例。

##### 关于项目结构及命名：
- 这个项目中有一些东西，比如set_up.py, __init__.py等等，没什么用，最初是计划发布到pypi上构建的项目结构，但实际部署过程中感觉完成度不够，发布不了.

##### 关于数据：
- 我没有使用新数据，因此data folder下的东西跟老师给的是一样的。
- 但是这个系统有点“屎山代码”的地方在于，写系统时有一堆相对地址引用，导致改变数据存放位置会有点问题，因此我的repo里自带了一个data文件夹
- 最后，请一定在test.ipynb截面使用系统，否则 current working folder(cwd) 的问题可能导致报错
