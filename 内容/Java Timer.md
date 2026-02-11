## 1. Timer – the Basics

Timer and TimerTask are java util classes that we use to schedule tasks in a background thread. Basically, TimerTask is the task to perform, and Timer is the scheduler.

## 2. Schedule a Task Once

### 2.1. After a Given Delay

Let’s start by simply running a single task with the help of a Timer:

```java
@Test
public void givenUsingTimer_whenSchedulingTaskOnce_thenCorrect() {
    TimerTask task = new TimerTask() {
        public void run() {
            System.out.println("Task performed on: " + new Date() + "n" +
              "Thread's name: " + Thread.currentThread().getName());
        }
    };
    Timer timer = new Timer("Timer");
    
    long delay = 1000L;
    timer.schedule(task, delay);
}
```

This performs the task after a certain delay, which we gave as the second parameter of the `schedule()` method. In the next section, we’ll see how to schedule a task at a given date and time.

Note that if we’re running this as a JUnit test, we should add a `Thread.sleep(delay * 2)` call to allow the Timer’s thread to run the task before the Junit test stops executing.

### 2.2. At a Given Date and Time

Now let’s look at the `Timer#schedule(TimerTask, Date)` method, which takes a Date instead of a long as its second parameter. This allows us to schedule the task at a certain instant rather than after a delay. 

This time, let’s imagine we have an old [[legacy]] database and want to [[migrate]] its data into a new database with a better schema.

We can create a `DatabaseMigrationTask` class that will [[handle]] this [[migration]]:

```java
public class DatabaseMigrationTask extends TimerTask {
    private List<String> oldDatabase;
    private List<String> newDatabase;

    public DatabaseMigrationTask(List<String> oldDatabase, List<String> newDatabase) {
        this.oldDatabase = oldDatabase;
        this.newDatabase = newDatabase;
    }

    @Override
    public void run() {
        newDatabase.addAll(oldDatabase);
    }
}
```

For simplicity, we represent the two databases by a List of Strings. Simply put, our [[migration]] consists of putting the data from the first list into the second.

To perform this [[migration]] at the desired [[instant]], we’ll have to use the overloaded version of the `schedule()` method:

```java
List<String> oldDatabase = Arrays.asList("Harrison Ford", "Carrie Fisher", "Mark Hamill");
List<String> newDatabase = new ArrayList<>();

LocalDateTime twoSecondsLater = LocalDateTime.now().plusSeconds(2);
Date twoSecondsLaterAsDate = Date.from(twoSecondsLater.atZone(ZoneId.systemDefault()).toInstant());

new Timer().schedule(new DatabaseMigrationTask(oldDatabase, newDatabase), twoSecondsLaterAsDate);
```

As we can see, we give the [[migration]] task and the date of execution to the schedule() method.

Then the [[migration]] is executed at the time indicated by twoSecondsLater:

```java
while (LocalDateTime.now().isBefore(twoSecondsLater)) {
    assertThat(newDatabase).isEmpty();
    Thread.sleep(500);
}
assertThat(newDatabase).containsExactlyElementsOf(oldDatabase);
```

Before that moment, the [[migration]] doesn’t occur.

## 3. Schedule a Repeatable a Task

Now that we’ve covered how to schedule the single execution of a task let’s see how to deal with repeatable tasks.

Once again, the Timer class offers multiple possibilities. We can set up the repetition to observe either a fixed delay or a fixed rate.

A fixed delay means that the execution will start after a period of time after the moment the last execution started, even if it was delayed (therefore being itself delayed).

Let’s say we want to schedule a task every two seconds, with the first execution taking one second and the second one taking two but being delayed by one second. Then the third execution starts at the fifth second:

```
0s     1s    2s     3s           5s
|--T1--|
|-----2s-----|--1s--|-----T2-----|
|-----2s-----|--1s--|-----2s-----|--T3--|
```

On the other hand, a fixed rate means that each execution will respect the initial schedule, no matter if a previous execution has been delayed.

Let’s reuse our previous example. With a fixed rate, the second task will start after three seconds (because of the delay), but the third one will start after four seconds (respecting the initial schedule of one execution every two seconds):

```
0s     1s    2s     3s    4s
|--T1--|       
|-----2s-----|--1s--|-----T2-----|
|-----2s-----|-----2s-----|--T3--|
```

Now that we’ve covered these two principles let’s see how we can use them.

To use fixed-delay scheduling, there are two more overloads of the schedule() method, each taking an extra parameter stating the periodicity in milliseconds.

Why two overloads? Because there’s still the possibility of starting the task at a certain moment or after a certain delay.

As for the fixed-rate scheduling, we have the two scheduleAtFixedRate() methods, which also take the periodicity in milliseconds. Again, we have one method to start the task at a given date and time and another to start it after a given delay.

It’s also worth mentioning that, if a task takes more time than the period to execute, it delays the whole chain of executions, whether we’re using fixed-delay or fixed-rate.

### 3.1. With a Fixed Delay

Now let’s imagine we want to implement a newsletter system, sending an email to our followers every week. In this case, a repetitive task seems ideal.

So let’s schedule the newsletter every second, which is basically spamming, but we’re good to go as the sending is fake.

First, we’ll design a NewsletterTask:

```java
public class NewsletterTask extends TimerTask {
    @Override
    public void run() {
        System.out.println("Email sent at: "
          + LocalDateTime.ofInstant(Instant.ofEpochMilli(scheduledExecutionTime()), ZoneId.systemDefault()));
        Random random = new Random();
        int value = random.ints(1, 7)
                .findFirst()
                .getAsInt();
        System.out.println("The duration of sending the mail will took: " + value);
        try {
            TimeUnit.SECONDS.sleep(value);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

Each time it executes, the task will print its scheduled time, which we gather using the TimerTask#scheduledExecutionTime() method.

So what if we want to schedule this task every second in fixed-delay mode? We’ll have to use the overloaded version of the schedule() that we mentioned earlier:

```java
new Timer().schedule(new NewsletterTask(), 0, 1000);
Thread.sleep(20000);
```

Of course, we only carry the tests for a few occurrences:

```
Email sent at: 2023-02-04T13:59:42.107
The duration of sending the mail will took: 4
Email sent at: 2023-02-04T13:59:46.109
The duration of sending the mail will took: 4
Email sent at: 2023-02-04T13:59:50.113
The duration of sending the mail will took: 1
Email sent at: 2023-02-04T13:59:51.127
The duration of sending the mail will took: 6
Email sent at: 2023-02-04T13:59:57.133
The duration of sending the mail will took: 4
Email sent at: 2023-02-04T14:00:01.146
The duration of sending the mail will took: 3
```

As we can see, there is at least one second between each execution, but they’re sometimes delayed by a millisecond. This phenomenon is due to our decision to use fixed-delay repetition.

### 3.2. With a Fixed Rate

Now, what if we were to use a fixed-rate repetition? Then we would have to use the scheduledAtFixedRate() method:

```java
new Timer().scheduleAtFixedRate(new NewsletterTask(), 0, 1000);
Thread.sleep(20000);
```

This time, executions aren’t delayed by the previous ones:

```
Email sent at: 2023-02-04T13:59:22.082
The duration of sending the mail will took: 1
Email sent at: 2023-02-04T13:59:23.082
The duration of sending the mail will took: 6
Email sent at: 2023-02-04T13:59:24.082
The duration of sending the mail will took: 1
Email sent at: 2023-02-04T13:59:25.082
The duration of sending the mail will took: 4
Email sent at: 2023-02-04T13:59:26.082
The duration of sending the mail will took: 1
Email sent at: 2023-02-04T13:59:27.082
The duration of sending the mail will took: 5
Email sent at: 2023-02-04T13:59:28.082
The duration of sending the mail will took: 1
Email sent at: 2023-02-04T13:59:29.082
The duration of sending the mail will took: 4
Here we can see the difference between fixed rate and schedule.
```

### 3.3. Schedule a Daily Task

Next, let’s run a task once a day:

```java
@Test
public void givenUsingTimer_whenSchedulingDailyTask_thenCorrect() {
    TimerTask repeatedTask = new TimerTask() {
        public void run() {
            System.out.println("Task performed on " + new Date());
        }
    };
    Timer timer = new Timer("Timer");
    
    long delay = 1000L;
    long period = 1000L * 60L * 60L * 24L;
    timer.scheduleAtFixedRate(repeatedTask, delay, period);
}
```

## 4. Cancel Timer and TimerTask

The execution of a task can be cancelled in a few ways.

### 4.1. Cancel the TimerTask Inside Run

The first option is to call the TimerTask.cancel() method inside the run() method’s implementation of the TimerTask itself:

```java
@Test
public void givenUsingTimer_whenCancelingTimerTask_thenCorrect()
  throws InterruptedException {
    TimerTask task = new TimerTask() {
        public void run() {
            System.out.println("Task performed on " + new Date());
            cancel();
        }
    };
    Timer timer = new Timer("Timer");
    
    timer.scheduleAtFixedRate(task, 1000L, 1000L);
    
    Thread.sleep(1000L * 2);
}
```

### 4.2. Cancel the Timer

Another option is to call the Timer.cancel() method on a Timer object:

```java
@Test
public void givenUsingTimer_whenCancelingTimer_thenCorrect() 
  throws InterruptedException {
    TimerTask task = new TimerTask() {
        public void run() {
            System.out.println("Task performed on " + new Date());
        }
    };
    Timer timer = new Timer("Timer");
    
    timer.scheduleAtFixedRate(task, 1000L, 1000L);
    
    Thread.sleep(1000L * 2); 
    timer.cancel(); 
}
```

### 4.3. Stop the Thread of the TimerTask Inside Run

We can also stop the thread inside the run method of the task, thus cancelling the entire task:

```java
@Test
public void givenUsingTimer_whenStoppingThread_thenTimerTaskIsCancelled() 
  throws InterruptedException {
    TimerTask task = new TimerTask() {
        public void run() {
            System.out.println("Task performed on " + new Date());
            // TODO: stop the thread here
        }
    };
    Timer timer = new Timer("Timer");
    
    timer.scheduleAtFixedRate(task, 1000L, 1000L);
    
    Thread.sleep(1000L * 2); 
}
```

Notice the TODO instruction in the run implementation; in order to run this simple example, we’ll need to actually stop the thread.

In a real-world custom thread implementation, stopping the thread should be supported, but in this case, we can ignore the deprecation and use the simple stop API on the Thread class itself.

## 5. Timer vs ExecutorService

We can also make good use of an ExecutorService to schedule timer tasks, instead of using the timer.

Here’s a quick example of how to run a repeated task at a specified interval:

```java
@Test
public void givenUsingExecutorService_whenSchedulingRepeatedTask_thenCorrect() 
  throws InterruptedException {
    TimerTask repeatedTask = new TimerTask() {
        public void run() {
            System.out.println("Task performed on " + new Date());
        }
    };
    ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();
    long delay  = 1000L;
    long period = 1000L;
    executor.scheduleAtFixedRate(repeatedTask, delay, period, TimeUnit.MILLISECONDS);
    Thread.sleep(delay + period * 3);
    executor.shutdown();
}
```

So what are the main differences between the Timer and the ExecutorService solution:

Timer can be sensitive to changes in the system clock; ScheduledThreadPoolExecutor isn’t.
Timer has only one execution thread; ScheduledThreadPoolExecutor can be configured with any number of threads.
Runtime Exceptions thrown inside the TimerTask kill the thread, so the following scheduled tasks won’t run further; with ScheduledThreadExecutor, the current task will be cancelled, but the rest will continue to run.

## 6. Conclusion

In this article, we illustrated the many ways we can use the simple, yet flexible Timer and TimerTask infrastructure built into Java for quickly scheduling tasks. There are, of course, much more complex and complete solutions in the Java world if we need them, such as the Quartz library, but this is a very good place to start.
