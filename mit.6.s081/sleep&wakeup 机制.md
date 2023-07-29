这里的机制是，如果一个线程需要等待某些事件，比如说等待UART硬件愿意接收一个新的字符，线程调用sleep函数并等待一个特定的条件。当特定的条件满足时，代码会调用wakeup函数。这里的sleep函数和wakeup函数是成对出现的。我们之后会看sleep函数的具体实现，它会做很多事情最后再调用switch函数来出让CPU。



sleep和wakeup函数需要通过某种方式链接到一起。也就是说，如果我们调用wakeup函数，我们只想唤醒正在等待刚刚发生的特定事件的线程。所以，sleep函数和wakeup函数都带有一个叫做sleep channel的参数。我们在调用wakeup的时候，需要传入与调用sleep函数相同的sleep channel。不过sleep和wakeup函数只是接收表示了sleep channel的64bit数值，它们并不关心这个数值代表什么。当我们调用sleep函数时，我们通过一个sleep channel表明我们等待的特定事件，当调用wakeup时我们希望能传入相同的数值来表明想唤醒哪个线程。



sleep函数为什么需要一个锁使用作为参数传入之前，我们先来看看假设我们有了一个更简单的不带锁作为参数的sleep函数，会有什么样的结果。这里的结果就是lost wakeup。

sleep函数不需要知道你在等待什么事件，它还是需要你知道你在等待什么数据，并且传入一个用来保护你在等待数据的锁。sleep函数需要特定的条件才能执行，而sleep自己又不需要知道这个条件是什么。在我们的例子中，sleep函数执行的特定条件是tx_done等于1。虽然sleep不需要知道tx_done，但是它需要知道保护这个条件的锁，也就是这里的uart_tx_lock。在调用sleep的时候，锁还被当前线程持有，之后这个锁被传递给了sleep。

在接口层面，sleep承诺可以原子性的将进程设置成SLEEPING状态，同时释放锁。这样wakeup就不可能看到这样的场景：锁被释放了但是进程还没有进入到SLEEPING状态。所以sleep这里将释放锁和设置进程为SLEEPING状态这两个行为合并为一个原子操作。

![img](https://906337931-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MHZoT2b_bcLghjAOPsJ%2F-MRDtbr2ScpNJk7lg7cC%2F-MREMzp89sA7KpGFvpRR%2Fimage.png?alt=media&token=d5e367f5-c7ac-490b-b636-25df3e3adb36)

![img](https://906337931-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MHZoT2b_bcLghjAOPsJ%2F-MRDtbr2ScpNJk7lg7cC%2F-MREOF_8qjRnwcWlpTMo%2Fimage.png?alt=media&token=9302dc21-f229-4cf6-9bf7-7eeb30db01d4)