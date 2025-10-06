## 코루틴 함수 정의
### Code
```python
async def say_after(delay, what):  
    await asyncio.sleep(delay)
    print(f"  {what}")
```

### Lesson
- `async` - 코루틴 함수 정의

---
## async 함수 호출 및  확인
### Code
```python
async def step_by_step():
	coro = say_after(1, "test")
	print(f"  return: {coro}")
	print(f"  type: {type(coro)}")
	
asyncio.run(step_by_step())
```

### Result
```
return: <coroutine object say_after at 0x10976ea80>
type: <class 'coroutine'>
```

### Lesson
- `await` 없이 실행한다면 코루틴 객체만 생성된다.
---

## await 붙인 후 확인
### Code
```python
async def step_by_step():
	coro = say_after(1, "test")
	result = await coro
	
asyncio.run(step_by_step())
```

### Result
```
test
```

### Lesson
- `await`
	- 코루틴을 실행
	- 현재 코루틴만 일시정지하며 다른 작업 가능 (논블로킹)
---

## async 함수 실행하기
### Code
```python
async def say_after(delay, what):  
    await asyncio.sleep(delay)  
    print(what)
    
# say_after() -> RuntimeWarning: coroutine 'step_by_step' was never awaited
# await say_after() -> SyntaxError: 'await' outside function
asyncio.run(say_after(1, 'hello'))
```

### Result
```bash
# 1초 뒤
hello 
```

### Lesson
- `async` 함수를 실행하려면 어떻게 해야할까? -> `await` 키워드를 붙인다
- `await say_hello()` -> 하지만 `await` 키워드는`async` 안에서만 가능하다
- 이를 해결하기 위해서 필요한 것이 코루틴 함수를 실행시키는 `asyncio.run()` 
---

## Awaitable
### Lesson
- `await` 키워드를 사용하여 실행을 일시 정지하고 완료를 기다릴 수 있는 객체
- 종류
	- coroutine
	- task
	- future
---

## Task
### Code
```python
async def say_after(delay, what):  
    await asyncio.sleep(delay)  
    print(what)  
  
  
async def run_after():  
    task1 = asyncio.create_task(say_after(5, 'hello'))  
    task2 = asyncio.create_task(say_after(3, 'world'))  
  
    result1 = await task1  
    result2 = await task2  
  
    print(f"Task 1 결과: {result1}")  
    print(f"Task 2 결과: {result2}")  
  
asyncio.run(run_after())
```

### Result
```bash
world
hello
```

### Lesson
- Task 예약 - Task 완료 대기용 변수 할당
- Task를 생성하고 즉시 이벤트 루프에 실행을 예약한다. (논블로킹)
	- 현재 실행 중인 코루틴은 Task의 완료를 기다리지 않고 다음 줄로 넘어감
- `await coroutine`은 해당 코루틴이 완료될 때까지 실행 흐름을 멈춤 (블로킹)

[코루틴과 태스크](https://docs.python.org/ko/3.13/library/asyncio-task.html)

---

## 동시성 구현
### Lesson
- `asyncio.gather(coro1, coro2, coro3)`
	- gather()를 사용하여 모든 작업을 동시에 실행하고 결과를 모아 반환
---

## 활용 - 동기 방식
### Code
```python
import requests  
import time  
  
def fetcher(session, url):  
    with session.get(url) as response:  
        return response.text
  
  
def main():  
    urls = ["https://naver.com", "https://google.com", "https://instagram.com", "https://youtube.com"] * 10  
  
    with requests.Session() as session:  
        result = [fetcher(session, url) for url in urls]  
        print(result)  
  
  
if __name__ == '__main__':  
    start = time.time()  
    main()  
    end = time.time()  
    print(f"Execution time: {end - start}")
```

### Result
```
...
Execution time: 20.99924087524414
```

### Lesson
- 응답이 올 때까지 대기 이후 다음 url 실행
- requests는 동기적인 코드가 있는 패키지
	- [Sessions](https://requests.readthedocs.io/en/latest/api/#requests.Session)

---

## 활용 - 비동기 방식
### Code
```python  
import asyncio  
import aiohttp  
import time  
  
async def fetcher(session, url):  
  
    async with session.get(url) as response:  
        return await response.text()  
  
  
async def main():  
    urls = ["https://naver.com", "https://google.com", "https://instagram.com", "https://youtube.com"] * 10  
  
    async with aiohttp.ClientSession() as session:  
        result = await asyncio.gather(*[fetcher(session, url) for url in urls])  
        print(result)  
  
  
if __name__ == '__main__':  
    start = time.time()  
    asyncio.run(main())  
    end = time.time()  
    print(f"Execution time: {end - start}")
```

### Result
```
Execution time: 1.3898160457611084
```

### Lesson
- 각각의 url에 대해 fetcher를 실행하지만 완료를 기다리지 않고 다음 url로 fetcher 실행
	- 이후 응답을 한번에 받고 반환