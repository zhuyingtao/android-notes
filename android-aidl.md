## Android AIDL

### AIDL 关键字

- oneway
  正常情况下Client调用AIDL接口方法时会阻塞，直到Server进程中该方法被执行完。oneway可以修饰AIDL文件里的方法，oneway修饰的方法在用户请求相应功能时不需要等待响应可直接调用返回，非阻塞效果，该关键字可以用来声明接口或者声明方法，如果接口声明中用到了oneway关键字，则该接口声明的所有方法都采用oneway方式。（注意,如果client和Server在同一进程中,oneway修饰的方法还是会阻塞）
- in
  非基本数据类型和string的参数类型必须加参数修饰符,in的意思是只输入,既最终server端执行完后不会影响到参数对象
- out
  与in相反,out修饰的参数只能由server写入并传递到client,而client传入的值并不会传递到server
- inout
  被inout修饰的参数,既可以从client传递到server,也可以server传递到client