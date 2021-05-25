# Effective-Java-3rd-edition-Chinese-English-bilingual
Effective Java（第 3 版）各章节的中英文学习参考，希望对 Java 技术的提高有所帮助，欢迎通过 issue 或 pr 提出建议和修改意见。

## 目录（Contents）
- **Chapter 2. Creating and Destroying Objects（创建和销毁对象）**
    - [Chapter 2 Introduction（章节介绍）](Chapter-2/Chapter-2-Introduction.md)
    - [Item 1: Consider static factory methods instead of constructors（考虑以静态工厂方法代替构造方法）](Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)
    - [Item 2: Consider a builder when faced with many constructor parameters（在面对多个构造方法参数时，请考虑构建器）](Chapter-2/Chapter-2-Item-2-Consider-a-builder-when-faced-with-many-constructor-parameters.md)
    - [Item 3: Enforce the singleton property with a private constructor or an enum type（使用私有构造方法或枚举类型实施单例属性）](Chapter-2/Chapter-2-Item-3-Enforce-the-singleton-property-with-a-private-constructor-or-an-enum-type.md)
    - [Item 4: Enforce noninstantiability with a private constructor（用私有构造方法实施不可实例化）](Chapter-2/Chapter-2-Item-4-Enforce-noninstantiability-with-a-private-constructor.md)
    - [Item 5: Prefer dependency injection to hardwiring resources（依赖注入优于硬连接资源）](Chapter-2/Chapter-2-Item-5-Prefer-dependency-injection-to-hardwiring-resources.md)
    - [Item 6: Avoid creating unnecessary objects（避免创建不必要的对象）](Chapter-2/Chapter-2-Item-6-Avoid-creating-unnecessary-objects.md)
    - [Item 7: Eliminate obsolete object references（排除过时的对象引用）](Chapter-2/Chapter-2-Item-7-Eliminate-obsolete-object-references.md)
    - [Item 8: Avoid finalizers and cleaners（避免使用终结器和清除器）](Chapter-2/Chapter-2-Item-8-Avoid-finalizers-and-cleaners.md)
    - [Item 9: Prefer try with resources to try finally（使用 try-with-resources 优于 try-finally）](Chapter-2/Chapter-2-Item-9-Prefer-try-with-resources-to-try-finally.md)
- **Chapter 3. Methods Common to All Objects（对象的通用方法）**
    - [Chapter 3 Introduction（章节介绍）](Chapter-3/Chapter-3-Introduction.md)
    - [Item 10: Obey the general contract when overriding equals（覆盖 equals 方法时应遵守的约定）](Chapter-3/Chapter-3-Item-10-Obey-the-general-contract-when-overriding-equals.md)
    - [Item 11: Always override hashCode when you override equals（当覆盖 equals 方法时，总要覆盖 hashCode 方法）](Chapter-3/Chapter-3-Item-11-Always-override-hashCode-when-you-override-equals.md)
    - [Item 12: Always override toString（始终覆盖 toString 方法）](Chapter-3/Chapter-3-Item-12-Always-override-toString.md)
    - [Item 13: Override clone judiciously（明智地覆盖 clone 方法）](Chapter-3/Chapter-3-Item-13-Override-clone-judiciously.md)
    - [Item 14: Consider implementing Comparable（考虑实现 Comparable 接口）](Chapter-3/Chapter-3-Item-14-Consider-implementing-Comparable.md)
- **Chapter 4. Classes and Interfaces（类和接口）**
    - [Chapter 4 Introduction（章节介绍）](Chapter-4/Chapter-4-Introduction.md)
    - [Item 15: Minimize the accessibility of classes and members（尽量减少类和成员的可访问性）](Chapter-4/Chapter-4-Item-15-Minimize-the-accessibility-of-classes-and-members.md)
    - [Item 16: In public classes use accessor methods not public fields（在公共类中，使用访问器方法，而不是公共字段）](Chapter-4/Chapter-4-Item-16-In-public-classes-use-accessor-methods-not-public-fields.md)
    - [Item 17: Minimize mutability（减少可变性）](Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)
    - [Item 18: Favor composition over inheritance（优先选择复合而不是继承）](Chapter-4/Chapter-4-Item-18-Favor-composition-over-inheritance.md)
    - [Item 19: Design and document for inheritance or else prohibit it（继承要设计良好并且具有文档，否则禁止使用）](Chapter-4/Chapter-4-Item-19-Design-and-document-for-inheritance-or-else-prohibit-it.md)
    - [Item 20: Prefer interfaces to abstract classes（接口优于抽象类）](Chapter-4/Chapter-4-Item-20-Prefer-interfaces-to-abstract-classes.md)
    - [Item 21: Design interfaces for posterity（为后代设计接口）](Chapter-4/Chapter-4-Item-21-Design-interfaces-for-posterity.md)
    - [Item 22: Use interfaces only to define types（接口只用于定义类型）](Chapter-4/Chapter-4-Item-22-Use-interfaces-only-to-define-types.md)
    - [Item 23: Prefer class hierarchies to tagged classes（类层次结构优于带标签的类）](Chapter-4/Chapter-4-Item-23-Prefer-class-hierarchies-to-tagged-classes.md)
    - [Item 24: Favor static member classes over nonstatic（静态成员类优于非静态成员类）](Chapter-4/Chapter-4-Item-24-Favor-static-member-classes-over-nonstatic.md)
    - [Item 25: Limit source files to a single top level class（源文件仅限有单个顶层类）](Chapter-4/Chapter-4-Item-25-Limit-source-files-to-a-single-top-level-class.md)
- **Chapter 5. Generics（泛型）**
    - [Chapter 5 Introduction（章节介绍）](Chapter-5/Chapter-5-Introduction.md)
    - [Item 26: Do not use raw types（不要使用原始类型）](Chapter-5/Chapter-5-Item-26-Do-not-use-raw-types.md)
    - [Item 27: Eliminate unchecked warnings（消除 unchecked 警告）](Chapter-5/Chapter-5-Item-27-Eliminate-unchecked-warnings.md)
    - [Item 28: Prefer lists to arrays（list 优于数组）](Chapter-5/Chapter-5-Item-28-Prefer-lists-to-arrays.md)
    - [Item 29: Favor generic types（优先使用泛型）](Chapter-5/Chapter-5-Item-29-Favor-generic-types.md)
    - [Item 30: Favor generic methods（优先使用泛型方法）](Chapter-5/Chapter-5-Item-30-Favor-generic-methods.md)
    - [Item 31: Use bounded wildcards to increase API flexibility（使用有界通配符增加 API 的灵活性）](Chapter-5/Chapter-5-Item-31-Use-bounded-wildcards-to-increase-API-flexibility.md)
    - [Item 32: Combine generics and varargs judiciously（明智地合用泛型和可变参数）](Chapter-5/Chapter-5-Item-32-Combine-generics-and-varargs-judiciously.md)
    - [Item 33: Consider typesafe heterogeneous containers（考虑类型安全的异构容器）](Chapter-5/Chapter-5-Item-33-Consider-typesafe-heterogeneous-containers.md)
- **Chapter 6. Enums and Annotations（枚举和注解）**
    - [Chapter 6 Introduction（章节介绍）](Chapter-6/Chapter-6-Introduction.md)
    - [Item 34: Use enums instead of int constants（用枚举类型代替 int 常量）](Chapter-6/Chapter-6-Item-34-Use-enums-instead-of-int-constants.md)
    - [Item 35: Use instance fields instead of ordinals（使用实例字段替代序数）](Chapter-6/Chapter-6-Item-35-Use-instance-fields-instead-of-ordinals.md)
    - [Item 36: Use EnumSet instead of bit fields（用 EnumSet 替代位字段）](Chapter-6/Chapter-6-Item-36-Use-EnumSet-instead-of-bit-fields.md)
    - [Item 37: Use EnumMap instead of ordinal indexing（使用 EnumMap 替换序数索引）](Chapter-6/Chapter-6-Item-36-Use-EnumSet-instead-of-bit-fields.md)
    - [Item 38: Emulate extensible enums with interfaces（使用接口模拟可扩展枚举）](Chapter-6/Chapter-6-Item-38-Emulate-extensible-enums-with-interfaces.md)
    - [Item 39: Prefer annotations to naming patterns（注解优于命名模式）](Chapter-6/Chapter-6-Item-39-Prefer-annotations-to-naming-patterns.md)
    - [Item 40: Consistently use the Override annotation（坚持使用 @Override 注解）](Chapter-6/Chapter-6-Item-40-Consistently-use-the-Override-annotation.md)
    - [Item 41: Use marker interfaces to define types（使用标记接口定义类型）](Chapter-6/Chapter-6-Item-41-Use-marker-interfaces-to-define-types.md)
- **Chapter 7. Lambdas and Streams（λ 表达式和流）**
    - [Chapter 7 Introduction（章节介绍）](Chapter-7/Chapter-7-Introduction.md)
    - [Item 42: Prefer lambdas to anonymous classes（λ 表达式优于匿名类）](Chapter-7/Chapter-7-Item-42-Prefer-lambdas-to-anonymous-classes.md)
    - [Item 43: Prefer method references to lambdas（方法引用优于 λ 表达式）](Chapter-7/Chapter-7-Item-43-Prefer-method-references-to-lambdas.md)
    - [Item 44: Favor the use of standard functional interfaces（优先使用标准函数式接口）](Chapter-7/Chapter-7-Item-44-Favor-the-use-of-standard-functional-interfaces.md)
    - [Item 45: Use streams judiciously（明智地使用流）](Chapter-7/Chapter-7-Item-45-Use-streams-judiciously.md)
    - [Item 46: Prefer side effect free functions in streams（在流中使用无副作用的函数）](Chapter-7/Chapter-7-Item-46-Prefer-side-effect-free-functions-in-streams.md)
    - [Item 47: Prefer Collection to Stream as a return type（优先选择 Collection 而不是流作为返回类型）](Chapter-7/Chapter-7-Item-47-Prefer-Collection-to-Stream-as-a-return-type.md)
    - [Item 48: Use caution when making streams parallel（谨慎使用并行流）](Chapter-7/Chapter-7-Item-48-Use-caution-when-making-streams-parallel.md)
- **Chapter 8. Methods（方法）**
    - [Chapter 8 Introduction（章节介绍）](Chapter-8/Chapter-8-Introduction.md)
    - [Item 49: Check parameters for validity（检查参数的有效性）](Chapter-8/Chapter-8-Item-49-Check-parameters-for-validity.md)
    - [Item 50: Make defensive copies when needed（在需要时制作防御性副本）](Chapter-8/Chapter-8-Item-50-Make-defensive-copies-when-needed.md)
    - [Item 51: Design method signatures carefully（仔细设计方法签名）](Chapter-8/Chapter-8-Item-51-Design-method-signatures-carefully.md)
    - [Item 52: Use overloading judiciously（明智地使用重载）](Chapter-8/Chapter-8-Item-52-Use-overloading-judiciously.md)
    - [Item 53: Use varargs judiciously（明智地使用可变参数）](Chapter-8/Chapter-8-Item-53-Use-varargs-judiciously.md)
    - [Item 54: Return empty collections or arrays, not nulls（返回空集合或数组，而不是 null）](Chapter-8/Chapter-8-Item-54-Return-empty-collections-or-arrays-not-nulls.md)
    - [Item 55: Return optionals judiciously（明智地的返回 Optional）](Chapter-8/Chapter-8-Item-55-Return-optionals-judiciously.md)
    - [Item 56: Write doc comments for all exposed API elements（为所有公开的 API 元素编写文档注释）](Chapter-8/Chapter-8-Item-56-Write-doc-comments-for-all-exposed-API-elements.md)
- **Chapter 9. General Programming（通用程序设计）**
    - [Chapter 9 Introduction（章节介绍）](Chapter-9/Chapter-9-Introduction.md)
    - [Item 57: Minimize the scope of local variables（将局部变量的作用域最小化）](Chapter-9/Chapter-9-Item-57-Minimize-the-scope-of-local-variables.md)
    - [Item 58: Prefer for-each loops to traditional for loops（for-each 循环优于传统的 for 循环）](Chapter-9/Chapter-9-Item-58-Prefer-for-each-loops-to-traditional-for-loops.md)
    - [Item 59: Know and use the libraries（了解并使用库）](Chapter-9/Chapter-9-Item-59-Know-and-use-the-libraries.md)
    - [Item 60: Avoid float and double if exact answers are required（若需要精确答案就应避免使用 float 和 double 类型）](Chapter-9/Chapter-9-Item-60-Avoid-float-and-double-if-exact-answers-are-required.md)
    - [Item 61: Prefer primitive types to boxed primitives（基本数据类型优于包装类）](Chapter-9/Chapter-9-Item-61-Prefer-primitive-types-to-boxed-primitives.md)
    - [Item 62: Avoid strings where other types are more appropriate（其他类型更合适时应避免使用字符串）](Chapter-9/Chapter-9-Item-62-Avoid-strings-where-other-types-are-more-appropriate.md)
    - [Item 63: Beware the performance of string concatenation（当心字符串连接引起的性能问题）](Chapter-9/Chapter-9-Item-63-Beware-the-performance-of-string-concatenation.md)
    - [Item 64: Refer to objects by their interfaces（通过接口引用对象）](Chapter-9/Chapter-9-Item-64-Refer-to-objects-by-their-interfaces.md)
    - [Item 65: Prefer interfaces to reflection（接口优于反射）](Chapter-9/Chapter-9-Item-65-Prefer-interfaces-to-reflection.md)
    - [Item 66: Use native methods judiciously（明智地使用本地方法）](Chapter-9/Chapter-9-Item-66-Use-native-methods-judiciously.md)
    - [Item 67: Optimize judiciously（明智地进行优化）](Chapter-9/Chapter-9-Item-67-Optimize-judiciously.md)
    - [Item 68: Adhere to generally accepted naming conventions（遵守被广泛认可的命名约定）](Chapter-9/Chapter-9-Item-68-Adhere-to-generally-accepted-naming-conventions.md)
- **Chapter 10. Exceptions（异常）**
    - [Chapter 10 Introduction（章节介绍）](Chapter-10/Chapter-10-Introduction.md)
    - [Item 69: Use exceptions only for exceptional conditions（仅在确有异常条件下使用异常）](Chapter-10/Chapter-10-Item-69-Use-exceptions-only-for-exceptional-conditions.md)
    - [Item 70: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors（对可恢复情况使用 checked 异常，对编程错误使用运行时异常）](Chapter-10/Chapter-10-Item-70-Use-checked-exceptions-for-recoverable-conditions-and-runtime-exceptions-for-programming-errors.md)
    - [Item 71: Avoid unnecessary use of checked exceptions（避免不必要地使用 checked 异常）](Chapter-10/Chapter-10-Item-71-Avoid-unnecessary-use-of-checked-exceptions.md)
    - [Item 72: Favor the use of standard exceptions（鼓励复用标准异常）](Chapter-10/Chapter-10-Item-72-Favor-the-use-of-standard-exceptions.md)
    - [Item 73: Throw exceptions appropriate to the abstraction（抛出能用抽象解释的异常）](Chapter-10/Chapter-10-Item-73-Throw-exceptions-appropriate-to-the-abstraction.md)
    - [Item 74: Document all exceptions thrown by each method（为每个方法记录会抛出的所有异常）](Chapter-10/Chapter-10-Item-74-Document-all-exceptions-thrown-by-each-method.md)
    - [Item 75: Include failure capture information in detail messages（异常详细消息中应包含捕获失败的信息）](Chapter-10/Chapter-10-Item-75-Include-failure-capture-information-in-detail-messages.md)
    - [Item 76: Strive for failure atomicity（尽力保证故障原子性）](Chapter-10/Chapter-10-Item-76-Strive-for-failure-atomicity.md)
    - [Item 77: Don’t ignore exceptions（不要忽略异常）](Chapter-10/Chapter-10-Item-77-Don’t-ignore-exceptions.md)
- **Chapter 11. Concurrency（并发）**
    - [Chapter 11 Introduction（章节介绍）](Chapter-11/Chapter-11-Introduction.md)
    - [Item 78: Synchronize access to shared mutable data（对共享可变数据的同步访问）](Chapter-11/Chapter-11-Item-78-Synchronize-access-to-shared-mutable-data.md)
    - [Item 79: Avoid excessive synchronization（避免过度同步）](Chapter-11/Chapter-11-Item-79-Avoid-excessive-synchronization.md)
    - [Item 80: Prefer executors, tasks, and streams to threads（Executor、task、流优于直接使用线程）](Chapter-11/Chapter-11-Item-80-Prefer-executors,-tasks,-and-streams-to-threads.md)
    - [Item 81: Prefer concurrency utilities to wait and notify（并发实用工具优于 wait 和 notify）](Chapter-11/Chapter-11-Item-81-Prefer-concurrency-utilities-to-wait-and-notify.md)
    - [Item 82: Document thread safety（文档应包含线程安全属性）](Chapter-11/Chapter-11-Item-82-Document-thread-safety.md)
    - [Item 83: Use lazy initialization judiciously（明智地使用延迟初始化）](Chapter-11/Chapter-11-Item-83-Use-lazy-initialization-judiciously.md)
    - [Item 84: Don’t depend on the thread scheduler（不要依赖线程调度器）](Chapter-11/Chapter-11-Item-84-Don’t-depend-on-the-thread-scheduler.md)
- **Chapter 12. Serialization（序列化）**
    - [Chapter 12 Introduction（章节介绍）](Chapter-12/Chapter-12-Introduction.md)
    - [Item 85: Prefer alternatives to Java serialization（Java 序列化的替代方案）](Chapter-12/Chapter-12-Item-85-Prefer-alternatives-to-Java-serialization.md)
    - [Item 86: Implement Serializable with great caution（非常谨慎地实现 Serializable）](Chapter-12/Chapter-12-Item-86-Implement-Serializable-with-great-caution.md)
    - [Item 87: Consider using a custom serialized form（考虑使用自定义序列化形式）](Chapter-12/Chapter-12-Item-87-Consider-using-a-custom-serialized-form.md)
    - [Item 88: Write readObject methods defensively（防御性地编写 readObject 方法）](Chapter-12/Chapter-12-Item-88-Write-readObject-methods-defensively.md)
    - [Item 89: For instance control, prefer enum types to readResolve（对于实例控制，枚举类型优于 readResolve）](Chapter-12/Chapter-12-Item-89-For-instance-control-prefer-enum-types-to-readResolve.md)
    - [Item 90: Consider serialization proxies instead of serialized instances（考虑以序列化代理代替序列化实例）](Chapter-12/Chapter-12-Item-90-Consider-serialization-proxies-instead-of-serialized-instances.md)
