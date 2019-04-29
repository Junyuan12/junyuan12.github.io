---
title: C++多线程编程中std::future的使用
updated: 2019-04-23 12:00:00
categories: C++
tags:
     - C++
     - thread
---
最近一直在搞多线程，C++的底子较烂，直接用`.detach()`方法创建线程，无法使输入顺序和输出顺序同步，而且输入数据在不断的产生，类似生产者和消费者问题，不断的创建新线程会浪费时间，所以想到用线程池的方法。

 <!-- more -->

>std::future是一个类模板(class template)，其对象存储未来的值，一个std::future对象在内部存储一个将来会被赋值的值，并提供了一个访问该值的机制，通过get()成员函数实现。但如果有人视图在get()函数可用之前通过它来访问相关的值，那么get()函数将会阻塞，直到该值可用。
*来源：https://blog.csdn.net/lijinqi1987/article/details/78507623*

我理解这段话的意思就是因为多线程输出没办法控制输出的顺序，那么`future`就为所有会有输出的位置放一个占位符，认定这个位置会有一个输出，但是具体什么时间会有不确定。

**测试结果：**

*来源：https://github.com/progschj/ThreadPool*

`GitHub`2k多`star`的一个线程池的项目，改了一下`example`，看输出更直观一些。

话说昨天有个小破站的后台源码被`push`上来了，`golang`的教程可能要火了，`push`一时爽，牢饭吃到老，职业道德还是要有的。

```
hello hello hello hello hello hello hello hello hello hello 1 2 0 1 2 2 0 1 0 0 world 
1world 2  world 0 world 1 hello 2 world 2 world 2 hello 0 hello 3 world 0 world 0 hello 1
hello 3 hello 0 hello 1 hello 2 world 0 world 1 hello 1 hello 2 world 2 world 0 hello 3 
world 3 world 1 world 3 world 0world 1  world 2 world 2 world 1 world 3

0 1 2 0 1 2 0 1 2 0 1 2 3 0 1 2 3 0 1 2 3
```

可以发现，创建了10个线程，线程的执行是并行的，但是输出的结果和输出的顺序是一样的。

**完整代码：**

```c++
//*****************ThreadPool.h**********************
#ifndef THREAD_POOL_H
#define THREAD_POOL_H

#include <vector>
#include <queue>
#include <memory>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <future>
#include <functional>
#include <stdexcept>

class ThreadPool {
public:
    ThreadPool(size_t);
    template<class F, class... Args>
    auto enqueue(F&& f, Args&&... args) 
        -> std::future<typename std::result_of<F(Args...)>::type>;
    ~ThreadPool();
private:
    // need to keep track of threads so we can join them
    std::vector< std::thread > workers;
    // the task queue
    std::queue< std::function<void()> > tasks;
    
    // synchronization
    std::mutex queue_mutex;
    std::condition_variable condition;
    bool stop;
};
 
// the constructor just launches some amount of workers
inline ThreadPool::ThreadPool(size_t threads)
    :   stop(false)
{
    for(size_t i = 0;i<threads;++i)
        workers.emplace_back(
            [this]
            {
                for(;;)
                {
                    std::function<void()> task;

                    {
                        std::unique_lock<std::mutex> lock(this->queue_mutex);
                        this->condition.wait(lock,
                            [this]{ return this->stop || !this->tasks.empty(); });
                        if(this->stop && this->tasks.empty())
                            return;
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }

                    task();
                }
            }
        );
}

// add new work item to the pool
template<class F, class... Args>
auto ThreadPool::enqueue(F&& f, Args&&... args) 
    -> std::future<typename std::result_of<F(Args...)>::type>
{
    using return_type = typename std::result_of<F(Args...)>::type;

    auto task = std::make_shared< std::packaged_task<return_type()> >(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
        
    std::future<return_type> res = task->get_future();
    {
        std::unique_lock<std::mutex> lock(queue_mutex);

        // don't allow enqueueing after stopping the pool
        if(stop)
            throw std::runtime_error("enqueue on stopped ThreadPool");

        tasks.emplace([task](){ (*task)(); });
    }
    condition.notify_one();
    return res;
}

// the destructor joins all threads
inline ThreadPool::~ThreadPool()
{
    {
        std::unique_lock<std::mutex> lock(queue_mutex);
        stop = true;
    }
    condition.notify_all();
    for(std::thread &worker: workers)
        worker.join();
}

#endif
```

```c++
//*****************example.cpp**********************
#include <iostream>
#include <vector>
#include <chrono>

#include "ThreadPool.h"

std::vector<std::vector<int>> a;

int main()
{ 
    ThreadPool pool(10);
    //create a <future> vector to wait the output
    std::vector< std::future<std::vector<int>> > results;
    std::vector<int> batch = {3, 3, 3};
    int m = 3;

    for (int b = 0; b < batch.size(); b++)
    {
        if (m > 0) {
            batch.push_back(4);
        }
        for (int i = 0; i < batch[b]; i++)
        {
            results.emplace_back(
                pool.enqueue([i] {
                    std::vector<int> res = {i}; 
                    std::cout << "hello " << i << " ";
                    std::this_thread::sleep_for(std::chrono::seconds(1));
                    std::cout << "world " << i << " ";
                    return res;
                }));
        }
        m--;
    }
	
    //get output
    for(auto && result: results)
        a.emplace_back(result.get());
    std::cout << std::endl;

    std::cout << std::endl;
	
    //after get all output, print it
    for (int i = 0; i < a.size(); i++)
    {
        for (int j = 0; j < a[i].size(); j++)
            std::cout << a[i][j] << " ";
    }

        system("pause");
    return 0;
}
```

