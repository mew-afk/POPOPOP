package ru.demo.interference1;
import java.util.concurrent.atomic.AtomicInteger;

// Лаунчер
public class Main {
    public static void main(String[] args) throws InterruptedException {
        new InterferenceExample().example();
    }
}
class InterferenceExample {
    private static final int HUNDRED_MILLION = 100_000_000;
    // Общий атомарный счётчик "разрешённых" итераций
    private final AtomicInteger counter = new AtomicInteger();

    // Возвращает true, когда лимит исчерпан.
// incrementAndGet() атомарен, поэтому ровно HUNDRED_MILLION вызовов вернут false,
// т.е. суммарно потоки сделают ровно столько итераций.
    boolean stop() {
        return counter.incrementAndGet() > HUNDRED_MILLION;
    }

    void example() throws InterruptedException {
        InterferenceThread thread1 = new InterferenceThread(this);
        InterferenceThread thread2 = new InterferenceThread(this);
        thread1.start();
        thread2.start();

// Ждём завершения обоих потоков
        thread1.join();
        thread2.join();
        System.out.println("Expected: " + HUNDRED_MILLION);
        System.out.println("Result: " + thread1.getI());
    }
}
class InterferenceThread extends Thread {
    private final InterferenceExample checker;

    // ИСПРАВЛЕНИЕ: написать static
// Он общий для всех потоков (static), а incrementAndGet() выполняется атомарно, поэтому обновления больше не теряются.
    private static final AtomicInteger i = new AtomicInteger();
    InterferenceThread(InterferenceExample checker) {
        this.checker = checker;
    }
    private void increment() {
        i.incrementAndGet(); // атомарный аналог i++
    }
    int getI() {
        return i.get();
    }
    @Override
    public void run() {
// Работаем, пока общий счётчик не исчерпает лимит
        while (!checker.stop()) {
            increment();
        }
    }
}
