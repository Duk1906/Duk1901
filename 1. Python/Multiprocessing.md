### python multiprocessing 模块
```python
import time
from multiprocessing import Pool   # 作用于进程
from multiprocessing.dummy import Pool as ThreadPool     # 作用于线程
```


​	
```python
def test(msg):
    print(msg)
    time.sleep(0.5)     # test函数太简单了，不加延时都看不出区别
```


​	
```python
if __name__ == '__main__':
    begin = time.time()
    n = 200
    l = [str(i) for i in range(n)]

    # 普通的执行打印任务
    for s in l:         
        test(s)

    end = time.time()
    print(end-begin)
    
    结果显示，n=200 个打印任务耗时为 100.1969358921051 秒

    # 使用multiprocessing模块去执行
    pool = Pool()
    pool.map(test, l)   # map 第二个参数是任务列表
    pool.close()
    pool.join()
 
    耗时 27.28937602043152 秒
```
