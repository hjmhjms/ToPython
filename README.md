# ToPython

# 简介
topython是unity与python的静态绑定解决方案。是一个通过反射分析代码并生成包装器类的解决方案。它是一个unity插件，极大地简化了unity,C#,C++与python的集成。它可以自动生成绑定代码来实现Python与Unity直接的互相访问，并将常量、变量、函数、属性、类、枚举、代理、事件映射到python。topython目标是让pythoner可以用上unity；让unity插上python这对翅膀；让双端python开发的项目提高开发效率，统一开发环境。
topython在2018.1.5开始建立开发，兼容2018.1.x以上版本，是否支持老版本unity我并没有去展开工作。

# 特性
+ 内建高性能tick-system
+ 内嵌msgpack, pycrypto, taggeddict(neox)等py库
+ 可以将unity相关类导出成python module 或者python类
+ 支持函数默认参数，ref参数，out参数
+ 支持array，利用python的tuple、list 与 c#的array进行反射转换
+ 支持prop, field, method, delegate, event, const, readonly, struct
+ 支持+与[]操作符重载
+ 支持暖更新
+ 支持Windows7+(64位), Android(armv7), iOS, macOS(64位)平台
+ 持续添加中......

# Python module 与 Python class 区别
+ 导出pythonmodule
  
    - 定义在ExportConfig的moduleList 将会导成module
    - 支持static methos、static prop，注意static prop将会自动添加get_ set_前缀
    - 支持const
    - 支持readonly

+ 导出python类
    - 支持prop(static or not)
    - 支持field(static or not)
    - 支持const
    - 支持readonly
    - 支持delegate and event
      
# 快速入门
在ExportConfig.cs中添加需要导出的类，导成python类的加入到typeList列表，导成python module的加入到moduleList列表，配置ExportConfig.cs 里面的TARGET_DIR 变量为最终生成wrapper的目录。点击菜单栏ToPython->Generate All生成所有的wrapper文件，或者点击ToPython->Clean清除自动生成的文件。

## 1.ToPythonEngine初始化，全局只需要初始化一次：
```csharp
    public class Main : MonoBehaviour
    {
        void Awake()
        {
            ToPythonEngine.Init(Application.dataPath + "/../Scripts", true);  
            //ToPythonEngine.PyInsertPath(Application.dataPath + "/../Scripts/lib") //已经默认初始目录下的lib是python的系统lib
            ToPythonBinder.Bind();
        }
    }
```

## 2.ToPythonEngine的update，主要推动里面的tick系统：
```csharp
    public class Main : MonoBehaviour
    {
        void Update()
        {
            ToPythonEngine.PyRun("main", "Update");
            ToPythonEngine.Update((uint)(Time.fixedDeltaTime * 1000));
        }
    }
```

## 3.ToPythonEngine的释放：
```csharp
    void OnApplicationQuit()
    {
        ToPythonEngine.Unit();
    }
```

## 4.用PyCall（推荐）实现c#调用python，没有返回值：
```csharp
    ToPythonEngine.PyCall<int,float>(testmodule, "Printab",222 , 22.2f);    //无返回值
    ToPythonEngine.PyCall(testmodule, "Printab",222 , 22.2f);    //无返回值
```

## 5.用PyInvoke （推荐）实现c#调用python：
```csharp
    int ret1=ToPythonEngine.PyInvoke<int>("module1", "func1");          //无参数，返回值int
    int ret1=ToPythonEngine.PyInvoke<int,int>("module1", "func1",3);  //1个参数，1个返回值
    float ret1=ToPythonEngine.PyInvoke<float,int>("module1", "func1",3,3);  //2个参数，1个返回值
```

## 6.用PyRun (基础接口，用时小心)实现c#调用python：
```csharp
    object param[] = new object param[]{1,2,3,4,"aaa"}
    PyObjectRef ref = ToPythonEngine.PyRun("examples.example3.main", "End",param);
```

## 7.在python使用unity：
```python
    import UnityEngine 
    UnityEngine.Debug("12345")
```

# Examples
参考ToPythonExamples目录。

# 性能表现
## 测试环境：

CPU: i5-7500 3.4GHz

内存: 16GB

显卡: 英伟达GTX 1050

**ps: Example中的TestPerformance为测试的场景，PerformanceTest为各种类和函数的穿层测试，其他button为Unity API穿层测试**
## 单命名空间: 30000次穿层(Call Function)性能测试：

| To(xx) | 静态类静态函数 | 普通类普通函数 | 普通类静态函数(通过类调用) | 普通类静态函数(通过实例调用) |
| :--- | :---  | :---  | :---  | :---  | 
| ToLua    | 11ms  | 71ms | 11ms | 60ms|
| ToPython | 17ms  | 44ms | 16ms | 40ms|

## 嵌套命名空间: 30000次穿层(Call Function)性能测试：

| To(xx) | 静态类静态函数 | 普通类普通函数 | 普通类静态函数(通过类调用)| 普通类静态函数(通过实例调用) |
| :--- | :---  | :--- | :--- | :--- | 
| ToLua    | 11ms  | 57ms  | 15ms | 62ms|
| ToPython | 16ms  | 43ms  | 16ms | 41ms|

## Unity API: 200000次穿层(Call Function)性能测试：

| To(xx) | Transform.Position | Transform.Rotate | Vector3 | GameObject |
| :--- | :--- | :--- | :--- | :--- | 
| ToLua    | 160ms  | 121ms  | 0 | 1238ms |
| ToPython | 558ms  | 348ms  | 290ms | 1200ms |

## 结论: 
- ToPython在需要创建实例调用函数的时候性能都优于ToLua#
- Unity API穿层性能弱于ToLua是因为ToLua将Vector3等Unity的类型内置了，不需要使用者手动导出，而ToPython没有内置
