### 反射

#### isAccessible()

这个方法不是告诉你属性是否是可访问的，而是说属性的修饰符当前是否应该被忽略的。所以，即使修饰符是public/protected的，返回也可能是 false。

#### setAccessible(boolean)

调用这个方法后，当属性被使用时，Java访问检查开关会禁用/启用。当为 true时，禁用访问检查；当为 false 时，启用访问检查。由于访问检查耗时较多，所以禁用安全检查会提升反射的速度。

#### getFields()

获取所有 public 属性，包括父类。

#### getDeclaredFields()

获取所有属性，不包括父类。