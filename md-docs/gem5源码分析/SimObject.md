
# SimObject

`SimObject` 是 gem5 中构建对象所使用的核心模块, 本文针对其[定义](src/sim/sim_object.hh)与[实现](src/sim/sim_object.cc)做初步分析

```cpp
class SimObject : public EventManager, public Serializable, public Drainable,
                  public statistics::Group, public Named
{
  private:
    typedef std::vector<SimObject *> SimObjectList;
    /** List of all instantiated simulation objects. */
    static SimObjectList simObjectList;
    /** Helper to resolve an object given its name. */
    static SimObjectResolver *_objNameResolver;
    /** Manager coordinates hooking up probe points with listeners. */
    ProbeManager *probeManager;

  protected:
    const SimObjectParams &_params;

  public:
    typedef SimObjectParams Params;
    const Params &params() const { return _params; }
    SimObject(const Params &p);
    virtual ~SimObject();
};
```

SimObject 继承如下 5 个父对象

- EventManager: 负责调度、管理、执行事件
- Serializable: 负责对象的序列化
- Drainable: 负责 drain 对象
- statistics::Group: 负责运行过程中统计、管理数据
- Named: 负责给 SimObject 起名

> 其中的 drain 类可以理解为需要预先处理的事件, 只有系统中所有的对象都被 drained，才能开始序列化、更改模型等操作 , 这五个类的声明都在 `src/sim/eventq.hh` 中

## 参考

- [深入理解 Gem5 之三——SimObjectPermalink](https://dingfen.github.io/cpp/2022/03/13/gem5-3.html)
- [configs](https://www.gem5.org/documentation/learning_gem5/part1/example_configs/)
- [statistics](https://www.gem5.org/documentation/learning_gem5/part1/gem5_stats/)