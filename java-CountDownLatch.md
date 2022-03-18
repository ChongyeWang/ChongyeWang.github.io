## Java CountDownLatch


CountDownLatch 用来保证一个任务开始前要等待所有其他线程完成。

它相当于一个计数器，初始值是线程的数量，每当一个任务完成，计数器的值减一，当计数器的值为0表示所有线程都已经完成，此时在CountDownLatch上等待的线程就可以继续任务。


关于CountDownLatch：

创建CountDownLatch时constructor的参数就是线程的数量。

当前独立于其他线程的线程要等所有其它线程完成count down。所有在awiit()等待的线程在count down等于0的时候一起继续进行。

contDown()方法减少count，await()方法会一直阻塞直到count为0。


```
// Java Program to demonstrate how
// to use CountDownLatch, Its used
// when a thread needs to wait for other
// threads before starting its work.
import java.util.concurrent.CountDownLatch;

public class CountDownLatchDemo
{
	public static void main(String args[])
				throws InterruptedException
	{
		// Let us create task that is going to
		// wait for four threads before it starts
		CountDownLatch latch = new CountDownLatch(4);

		// Let us create four worker
		// threads and start them.
		Worker first = new Worker(1000, latch,
								"WORKER-1");
		Worker second = new Worker(2000, latch,
								"WORKER-2");
		Worker third = new Worker(3000, latch,
								"WORKER-3");
		Worker fourth = new Worker(4000, latch,
								"WORKER-4");
		first.start();
		second.start();
		third.start();
		fourth.start();

		// The main task waits for four threads
		latch.await();

		// Main thread has started
		System.out.println(Thread.currentThread().getName() +
						" has finished");
	}
}

// A class to represent threads for which
// the main thread waits.
class Worker extends Thread
{
	private int delay;
	private CountDownLatch latch;

	public Worker(int delay, CountDownLatch latch, String name)
	{
		super(name);
		this.delay = delay;
		this.latch = latch;
	}

	@Override
	public void run()
	{
		try
		{
			Thread.sleep(delay);
			latch.countDown();
			System.out.println(Thread.currentThread().getName()
							+ " finished");
		}
		catch (InterruptedException e)
		{
			e.printStackTrace();
		}
	}
}



```
```
Output:

WORKER-1 finished
WORKER-2 finished
WORKER-3 finished
WORKER-4 finished
main has finished
```

(代码来自：GeeksforGeeks)
