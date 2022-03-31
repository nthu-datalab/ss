---
marp: true
paginate: true

---

# Lab 06 - Android Flow
### Software Studio - 2022 Spring

---

#### In coroutines, a `flow` is a type that can **emit multiple values sequentially**, as opposed to suspend functions that return only a single value.

#### A flow is very similar to an Iterator that produces a sequence of values, but it uses `suspend` functions to produce and consume values **asynchronously**. 

#### This means, for example, that the flow can safely make a network request to produce the next value **without blocking the main thread**.

---
## Explain why we use Flow
There is a developer called Pancoho.
![](https://i.imgur.com/6QnbHEP.png)

---
## Explain why we use Flow
![width:850px](https://i.imgur.com/K1GzdPz.png)

---
## Explain why we use Flow
![width:850px](https://i.imgur.com/QtWdMwG.png)

---
## Explain why we use Flow
![width:850px](https://i.imgur.com/bn20KW6.png)

---
## Without Flow

* You can do it with lots of `suspend` functions.
![width:650px bg right](https://i.imgur.com/AMGG0cC.png)

* However, after doing that for a while, it takes a lot of time for developers to invest in the infrastructures.

---
## With Flow

- Instead of requesting data, we **observe** it.
- Observing it like installing tubes for water.
- **Any updates to the source data will flow down to the view automatically**, you don't have to walk to the lake!!

![width:500px bg right](https://i.imgur.com/ir5FsSX.png)

---
## Without Flow
![width:950px](https://i.imgur.com/6TbLCsM.png)

---
## With Flow
![width:950px](https://i.imgur.com/mIaB64M.png)

---
## Kotlin Flow<T>
### Producers **emits** data to the flow.
* Data Sources or Repositories
### Consumers **collects** data from the flow.
* UI layer
![](https://i.imgur.com/WngnMjg.png)

---
## Creating Flows
* Most of the times, you don't need to create a flow. The libraries in your data sources are already integreated with coroutines and flows.

---
![](https://i.imgur.com/R8fKS33.png)

---
## Room DAO
The `Room` library acts as a producer and **emit** the content of the query every time.
```kotlin
@Dao
interface ScheduleDao {

    @Query("SELECT * FROM schedule ORDER BY arrival_time ASC")
    fun getAll(): Flow<List<Schedule>>
}
```

---
## Creating Flows By Yourself!!
```kotlin
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking
import kotlinx.coroutines.delay

fun getSequence(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(1000)
        emit(i)
    }
}
fun main() = runBlocking {
    getSequence()
        .collect { value ->
            println(value)
        }
}

```

---
## Creating Flows By Yourself!!
`flow()` is a flow builder. It would create a `Flow<T>`.

`emit()` send the result into flow. And we know that flow can emit multiple times and values.

`collect()` receive the values from `emit()`. Every time `emit()` is called, the block in `collect()` will be executed.

---
## Flow Builder
1. `flow()`
2. `asFlow()`
3.  `flowOf()`
4. ...

---
## [`flow()`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow.html)
```kotlin
fun <T> flow(
    block: suspend FlowCollector<T>.() -> Unit
): Flow<T>
```
It creates a **cold flow**.
The flow being *cold* means that the block is called every time a terminal operator is applied to the resulting flow.

---
## Example of `flow()`

```kotlin
fun getSequence(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        println("Emit $i")
        emit(i)
    }
}
fun main() = runBlocking {
    val f = getSequence()
    println("Start to collect")
    f.collect { value ->
        delay(1000)
        println("Collected $value")
    }
}
```
---
## Result of `flow()`
```kotlin
Start to collect
Emit 1
Collected 1
Emit 2
Collected 2
Emit 3
Collected 3
```

---
![](https://i.imgur.com/qtmmOsu.png)

---
## [`asFlow()`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/as-flow.html)
```kotlin
fun IntRange.asFlow(): Flow<Int>
```
Creates a flow that produces values from the range.

---
## Example of `asFlow()`
```kotlin
fun main() = runBlocking {
    val f = (1..3).asFlow()
    println("1. Start to collect")
    f.collect { value ->
        println("Collected $value")
    }
    println("2. Start to collect")
    f.collect { value ->
        println("Collected $value")
    }
}
```

---
## Result of `asFlow()`
```kotlin
1. Start to collect
Collected 1
Collected 2
Collected 3
2. Start to collect
Collected 1
Collected 2
Collected 3
```

---
## [`flowOf()`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-of.html)
```kotlin
fun <T> flowOf(vararg elements: T): Flow<T>
```
Creates a flow that produces values from the specified **`vararg`**-arguments.

---
## Example of `flowOf()`
```kotlin
fun main() = runBlocking {
    val f = flowOf(1, 2, 3)
    println("1. Start to collect")
    f.collect { value ->
        println("Collected $value")
    }
    println("2. Start to collect")
    f.collect { value ->
        println("Collected $value")
    }
}
```

---
## Result of `flowOf()`
```kotlin
1. Start to collect
Collected 1
Collected 2
Collected 3
2. Start to collect
Collected 1
Collected 2
Collected 3
```

---
## Operators
Flow is sequential.
Before calling `collect()`, we can use different function operators to deal with or transform the value in flow.

---
## `map()`
```kotlin
fun main() = runBlocking {
    (1..3).asFlow()
        .map { "Hello $it" }
        .collect { println(it) }
}
```
Result : 
```kotlin
Hello 1
Hello 2
Hello 3
```

---
## `filter()`
```kotlin
fun main() = runBlocking {
    (1..10).asFlow()
        .filter { it % 2 == 0 }
        .collect { println(it) }
}
```
Result :
```kotlin
2
4
6
8
10
```
---
## [`transform()`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/transform.html)
`transform()` is a flexible function that may transform emitted element, skip it or emit it multiple times.


```kotlin
fun main() = runBlocking {
    (1..10).asFlow()
        .transform {
            if (it % 2 == 0) {
                emit(it)
                emit(it)
            }
        }
        .collect { println(it) }
}
```

---
## Advanced example
![width:1000px](https://i.imgur.com/aShZ7fn.png)

---
## Advanced example
![width:1000px](https://i.imgur.com/s0LPMRf.png)

---
## Advanced example
![width:1000px](https://i.imgur.com/FYQoRLp.png)

---
## Advanced example
![width:1000px](https://i.imgur.com/wmNcxQP.png)

---
## Advanced example
![width:1000px](https://i.imgur.com/uj4S1Fh.png)

---
## Advanced example
![width:1000px](https://i.imgur.com/UhHjV1l.png)

---
## Advanced example
![width:1000px](https://i.imgur.com/4LBv0fi.png)

---
## Reference
[Kotlin flows on Android](https://developer.android.com/kotlin/flow)
[Asynchronous Flow](https://kotlinlang.org/docs/flow.html)
[Kotlin Coroutine Flow](https://waynestalk.com/kotlin-coroutine-flow-tutorial/)