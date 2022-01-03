## Chapter 12. Serialization（序列化）

### Chapter 12 Introduction（章节介绍）

THIS chapter concerns object serialization, which is Java’s framework for encoding objects as byte streams (serializing) and reconstructing objects from their encodings (deserializing). Once an object has been serialized, its encoding can be sent from one VM to another or stored on disk for later deserialization. This chapter focuses on the dangers of serialization and how to minimize them.

本章关注对象序列化，它是 Java 的框架，用于将对象编码为字节流（序列化），并从对象的编码中重构对象（反序列化）。对象序列化后，可以将其编码从一个 VM 发送到另一个 VM，或者存储在磁盘上，以便今后反序列化。本章主要讨论序列化的风险以及如何将其最小化。

### Contents of the chapter（章节目录）
- [Item 85: Prefer alternatives to Java serialization（优先选择 Java 序列化的替代方案）](/Chapter-12/Chapter-12-Item-85-Prefer-alternatives-to-Java-serialization.md)
- [Item 86: Implement Serializable with great caution（非常谨慎地实现 Serializable）](/Chapter-12/Chapter-12-Item-86-Implement-Serializable-with-great-caution.md)
- [Item 87: Consider using a custom serialized form（考虑使用自定义序列化形式）](/Chapter-12/Chapter-12-Item-87-Consider-using-a-custom-serialized-form.md)
- [Item 88: Write readObject methods defensively（防御性地编写 readObject 方法）](/Chapter-12/Chapter-12-Item-88-Write-readObject-methods-defensively.md)
- [Item 89: For instance control, prefer enum types to readResolve（对于实例控制，枚举类型优于 readResolve）](/Chapter-12/Chapter-12-Item-89-For-instance-control-prefer-enum-types-to-readResolve.md)
- [Item 90: Consider serialization proxies instead of serialized instances（考虑以序列化代理代替序列化实例）](/Chapter-12/Chapter-12-Item-90-Consider-serialization-proxies-instead-of-serialized-instances.md)
