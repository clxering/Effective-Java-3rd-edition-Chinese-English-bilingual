## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Chapter 2 Introduction（章节介绍）

This chapter concerns（关注、涉及） creating and destroying objects: when and how to create them, when and how to avoid creating them, how to ensure they are destroyed in a timely manner, and how to manage any cleanup actions that must precede their destruction.

本章涉及创建和销毁对象：何时以及如何创建对象，何时以及如何避免创建对象，如何确保它们被及时销毁，以及如何管理在销毁之前必须执行的清理操作。

### Contents of the chapter（章节目录）
- [Item 1: Consider static factory methods instead of constructors（考虑以静态工厂方法代替构造函数）](/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)
- [Item 2: Consider a builder when faced with many constructor parameters（在面对多个构造函数参数时，请考虑构建器）](/Chapter-2/Chapter-2-Item-2-Consider-a-builder-when-faced-with-many-constructor-parameters.md)
- [Item 3: Enforce the singleton property with a private constructor or an enum type（使用私有构造函数或枚举类型实施单例属性）](/Chapter-2/Chapter-2-Item-3-Enforce-the-singleton-property-with-a-private-constructor-or-an-enum-type.md)
- [Item 4: Enforce noninstantiability with a private constructor（用私有构造函数实施不可实例化）](/Chapter-2/Chapter-2-Item-4-Enforce-noninstantiability-with-a-private-constructor.md)
- [Item 5: Prefer dependency injection to hardwiring resources（依赖注入优于硬连接资源）](/Chapter-2/Chapter-2-Item-5-Prefer-dependency-injection-to-hardwiring-resources.md)
- [Item 6: Avoid creating unnecessary objects（避免创建不必要的对象）](/Chapter-2/Chapter-2-Item-6-Avoid-creating-unnecessary-objects.md)
- [Item 7: Eliminate obsolete object references（排除过时的对象引用）](/Chapter-2/Chapter-2-Item-7-Eliminate-obsolete-object-references.md)
- [Item 8: Avoid finalizers and cleaners（避免使用终结器和清除器）](/Chapter-2/Chapter-2-Item-8-Avoid-finalizers-and-cleaners.md)
- [Item 9: Prefer try with resources to try finally（使用 try-with-resources 优于 try-finally）](/Chapter-2/Chapter-2-Item-9-Prefer-try-with-resources-to-try-finally.md)
