### Android 异步

Android UI是线程不安全的，如果想要在子线程里进行UI操作，就需要借助Android的异步消息处理机制。

#### Handler

![img](http://img.blog.csdn.net/20130817090611984?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

1. Handler的sendMessage()方法
2. Handler的post()方法
3. View的post()方法
4. Activity的runOnUiThread()方法

- 原理

[Android Handler消息机制实现原理](https://www.jianshu.com/p/6cc4d4b4676b)

#### AsyncTask

AsyncTask是一个抽象类，所以如果我们想使用它，就必须要创建一个子类去继承它。在继承时我们可以为AsyncTask类指定三个泛型参数，这三个参数的用途如下：

1. Params : 在执行AsyncTask时需要传入的参数，可用于在后台任务中使用。
2. Progress : 后台任务执行时，如果需要在界面上显示当前的进度，则使用这里指定的泛型作为进度单位。
3. Result : 当任务执行完毕后，如果需要对结果进行返回，则使用这里指定的泛型作为返回值类型。

子类经常需要重写的方法有：

1. onPreExecute() : 这个方法会在后台任务开始执行之间调用，用于进行一些界面上的初始化操作，比如显示一个进度条对话框等。
2. doInBackground(Params...) : **必须重写**，这个方法中的所有代码都会在子线程中运行，在这里去处理所有的耗时任务。任务一旦完成就可以通过return语句来将任务的执行结果进行返回，如果AsyncTask的第三个泛型参数指定的是Void，就可以不返回任务执行结果。注意，在这个方法中是不可以进行UI操作的，如果需要更新UI元素，比如说反馈当前任务的执行进度，可以调用publishProgress(Progress...)方法来完成。
3. onProgressUpdate(Progress...) : 当在后台任务中调用了publishProgress(Progress...)方法后，这个方法就很快会被调用，方法中携带的参数就是在后台任务中传递过来的。在这个方法中可以对UI进行操作，利用参数中的数值就可以对界面元素进行相应的更新。
4. onPostExecute(Result) : 当后台任务执行完毕并通过return语句进行返回时，这个方法就很快会被调用。返回的数据会作为参数传递到此方法中，可以利用返回的数据来进行一些UI操作，比如说提醒任务执行的结果，以及关闭掉进度条对话框等。

一个比较完整的示例：

```java
class DownloadTask extends AsyncTask<Void, Integer, Boolean> {

	@Override
	protected void onPreExecute() {
		progressDialog.show();
	}

	@Override
	protected Boolean doInBackground(Void... params) {
		try {
			while (true) {
				int downloadPercent = doDownload();
				publishProgress(downloadPercent);
				if (downloadPercent >= 100) {
					break;
				}
			}
		} catch (Exception e) {
			return false;
		}
		return true;
	}

	@Override
	protected void onProgressUpdate(Integer... values) {
		progressDialog.setMessage("当前下载进度：" + values[0] + "%");
	}

	@Override
	protected void onPostExecute(Boolean result) {
		progressDialog.dismiss();
		if (result) {
			Toast.makeText(context, "下载成功", Toast.LENGTH_SHORT).show();
		} else {
			Toast.makeText(context, "下载失败", Toast.LENGTH_SHORT).show();
		}
	}
}
```

在整个应用程序中的所有AsyncTask实例都会共用同一个SerialExecutor，其源码如下：

```java
private static class SerialExecutor implements Executor {  
    final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();  
    Runnable mActive;  
  
    public synchronized void execute(final Runnable r) {  
        mTasks.offer(new Runnable() {  
            public void run() {  
                try {  
                    r.run();  
                } finally {  
                    scheduleNext();  
                }  
            }  
        });  
        if (mActive == null) {  
            scheduleNext();  
        }  
    }  
  
    protected synchronized void scheduleNext() {  
        if ((mActive = mTasks.poll()) != null) {  
            THREAD_POOL_EXECUTOR.execute(mActive);  
        }  
    }  
}  
```

每次当一个任务执行完毕后，下一个任务才会得到执行，SerialExecutor模仿的是单一线程池的效果，如果我们快速地启动了很多任务，同一时刻只会有一个线程正在执行，其余的均处于等待状态。

如果不想使用默认的线程池，还可以自由地进行配置。比如使用如下的代码来启动任务：

```java
Executor exec = new ThreadPoolExecutor(15, 200, 10,  
        TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>());  
new DownloadTask().executeOnExecutor(exec); 
```

这样就可以使用我们自定义的一个Executor来执行任务，而不是使用SerialExecutor。上述代码的效果允许在同一时刻有15个任务正在执行，并且最多能够存储200个任务。