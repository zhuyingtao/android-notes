## Android IPC

![img](assets/android-ipc/aHR0cHM6Ly9pbWdjb252ZXJ0LmNzZG5pbWcuY24vYUhSMGNEb3ZMM1Z3Ykc5aFpDMXBiV0ZuWlhNdWFtbGhibk5vZFM1cGJ5OTFjR3h2WVdSZmFXMWhaMlZ6THprME5ETTJOUzB5TkRBM016QTBOemt3T1RBd1pXRTRMbkJ1Wnc.png)

### IPC 方式

##### 使用Bundle

- 四大组件中的三大组件（Activity、Service、Receiver）都支持在Intent中传递Bundle数据
- Bundle中的数据除了基本类型，其他的都需要可序列化
- 适用于从一个进程启动另一个进程，比如在一个进程中启动另一个进程中的Activity、Service、Receiver，与此同时传递数据的情况

##### 使用文件共享

- 2个进程通过读写同一个文件来交换数据
- 当然要把数据写入文件，必然要求数据可以序列化和反序列化
- 文件共享对文件格式没有要求，只要读写双方约定好数据格式
- 文件共享的方式存在并发读写的问题，适合对数据同步要求不高的进程间通信，并且要妥善处理并发读写的问题

##### 使用Messenger

- 使用Messenger来传递Message，Message中能使用的字段只有what、arg1、arg2、Bundle和replyTo,自定义的Parcelable对象无法通过object字段来传输
- Message中的Bundle支持多种数据类型，replyTo字段用于传输Messager对象，以便进程间相互通信
- Messenger以串行的方式处理客户端发来的消息，不适合有大量并发的请求
- Messenger方法只能传递消息，不能跨进程调用方法

##### 使用AIDL

- AIDL接口可以通过编写AIDL文件然后由系统生成对应的Binder类，通过Binder我们就可以进行跨进程通信了
- AIDL文件中只支持如下几种类型：
  - 基本类型，如int、long等
  - String和CharSequence
  - List：只支持ArrayList，里面的每个元素都必须被AIDL支持
  - Map：只支持HashMap，里面的key、value都必须被AIDL支持
  - AIDL：所有的AIDL接口本身也可以在AIDL文件中使用
- AIDL文件中用到的自定义Parcelable对象和AIDL对象必须要显示的import进来
- 出了基本数据类型，其他类型的参数必须标明是入参还是出参，in表示输入型参数，out表示输出型参数，inout表示输入输出型参数