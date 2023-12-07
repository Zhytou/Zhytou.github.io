---
title: "C++实现反射"
date: 2023-10-27T15:17:54+08:00
draft: false
---

最近在实习的过程中，老板要求写一个回测系统。其大致工作模式就是hft-sys写死，会动态链接加载不同model，通过model类名实例化对象并调用内部函数。因此，整个的难点其实就是动态链接+反射。这篇博客就记录一下自己在网上查到的资料以及相关实践。

总的来说，C++实现发射的方法包括：

- 侵入式
- 运行时
- 编译期

## 侵入式

侵入式的做法其实有些类似Qt，在需要支持反射的类定义中添加宏，从而引入一系列帮助系统确认类型的metadata。

其原理也非常容易理解，使用一个全局Map管理类名和其构造函数的键值对，并用宏帮助注册。

```c++
// reflect.h
using ModelConstructorFn = Model *(*) ();

// 用于保存Model Info的全局Map
map<string, ModelConstructorFn> miMap;

// ModelInfo类，用于保存模型信息
struct ModelInfo {
    ModelInfo(const string& name, ObjectConstructorFn ctor):
      name_(name), ctor_(ctor) {
        if (miMap.find(name) == miMap.end()) {
            miMap[name] = ctor;
        }
    }
    ~ModelInfo() {}

    string name_;
    ModelConstructorFn ctor_;
};

#define DECLARE_MODEL(name) \
  protected: \
    static ModelInfo model_info_; \
  public: \
    static Model* CreateModel();

#define IMPLEMENT_MODEL(name) \
  ModelInfo name::model_info_(#name, (ModelConstructorFn) name::CreateModel);\
  Model* name::CreateModel() { \
    return new name; \
  }
```

```c++
// model.h
// 基类Model
class Model {
 public:
    Model() {}
    virtual ~Model() {}
    
    // 接口
    virtual void OnQuote(int *) = 0;
};

// model_cta.cc
// 需要支持反射的子类
class ModelCTA : public Model {
    DECLARE_MODEL(ModelCTA)
 public:
    void OnQuote(int *) {
        // ...
    }
};

IMPLEMENT_MODEL(ModelCTA)
```

从上面的例子我们可以看到，通过宏DECLARE_MODEL在子类中额外定义了一个静态变量model_info_和CreateModel函数。由于静态变量的特性，它会在main函数之前初始化，即：会执行其构造函数中`miMap[name] = ctor;`的操作，以达成向miMap中注册信息的目的。当然，这只是最简单的反射，它还存在很多局限性，包括：无法根据类名动态的判断其内部属性是否存在（python getattr操作）、无法使用带参数的构造函数等。

## 运行时

事实上，仔细观察侵入式反射的例子，对于只是希望根据类名创建对象来说，其中的MapInfo类其实可以省略。只要找到某种机制向miMap写入键值对即可。由此便有了下面这种方法:

```c++
// modelfactory.h
class Model;

using ModelConstructorFn = Model *(*) ();

class ModelFactory {
 public:
    static ModelFactory *GetInstance();
    Model *Create(const string &name) {
        if (miMap_ == nullptr) {
            return nullptr;
        }
        auto iter = miMap_->find(name);
        if (miMap_->end() != iter) {
            return iter->second();
        }
        return nullptr;
    }
    bool Register(const string &name, ModelConstructorFn constructor) {
        if (miMap_ == nullptr) {
            miMap_ = new map<string, ModelConstructorFn>();
        }
        auto iter = miMap_->find(name);
        if (miMap_->end() == iter) {
            miMap_->emplace(name, ctor);
            return true;
        }
        return false;
    }

 private:
    ModelFactory() {}
    ~ModelFactory() {}

    static ModelFactory *instance_;
    static map<string, ModelConstructorFn> *miMap_;
};
```

```c++
// model_cta.cc
class ModelCTA : public Model {
    DECLARE_MODEL(ModelCTA)
 public:
    void OnQuote(int *) {
        // ...
    }
};

struct RegitserHelper {
    struct RegitserHelper(string name, ModelConstructorFn ctor) {
        ModelFactory *mf = ModelFactory::GetInstance();
        mf->Register(name, ctor);
    }
}

static RegitserHelperregister_modelcta_helper("ModelCTA", []() -> Model * { return new ModelCTA(); });
```

为了替代RegisterHelper类，我们可以利用gcc constructor特性使得某个函数在main函数之前运行，从而达成注册Model的目的：

```c++
__attribute__((constructor)) static void initilize() {
    ModelFactory *mf = ModelFactory::GetInstance();
    mf->Register("ModelCTA", []() -> Model * { return new ModelCTA(); });
    return;
}
```

## 参考

![如何优雅的实现C++编译器反射](https://netcan.github.io/2020/08/01/%E5%A6%82%E4%BD%95%E4%BC%98%E9%9B%85%E7%9A%84%E5%AE%9E%E7%8E%B0C-%E7%BC%96%E8%AF%91%E6%9C%9F%E9%9D%99%E6%80%81%E5%8F%8D%E5%B0%84/)