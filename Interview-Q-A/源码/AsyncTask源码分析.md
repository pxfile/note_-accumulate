# AsyncTask源码分析
* 使用AsyncTask的规则
 * AsyncTask的类必须在UI线程加载（从4.1开始系统会帮我们自动完成）
 
 * AsyncTask对象必须在UI线程创建
 
 * execute方法必须在UI线程调用
 
 * 不要在你的程序中去直接调用onPreExecute(), onPostExecute, doInBackground, onProgressUpdate方法
 
 * 一个AsyncTask对象只能执行一次，即只能调用一次execute方法，否则会报运行时异常
 
 * AsyncTask不是被设计为处理耗时操作的，耗时上限为几秒钟，如果要做长耗时操作，强烈建议你使用Executor，ThreadPoolExecutor以及FutureTask
 
 * 在1.6之前，AsyncTask是串行执行任务的，1.6的时候AsyncTask开始采用线程池里处理并行任务，但是从3.0开始，为了避免AsyncTask所带来的并发错误，AsyncTask又采用一个线程来串行执行任务
 
 * AsyncTask任务数量上限是128个


