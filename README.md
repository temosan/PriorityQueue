## 우선순위 큐를 접목한 METHOD큐 실행

앱을 처음 실행시키고 시작하는 Initialize때, 일반적으로 초기화 과정은 다음과 같다.

1. App Intro Code시작
2. 분기 (#1)
3. 분기 #1에서 성공했을 경우 (#2)
4. 분기 #2에서 실패했을 경우 (#3)
5. …

---

다음과 같을때 해당 분기와 맞지 않는 코드와 검증코드까지 많아지게 된다. 제일 좋은 방법으로는 각 분기에서 실행되는 묶음이 성공되고 나서 이후의 단계로 가는것이 이상적이다.

따라서 하나의 Queue를 두고 해당 Queue에서 실행 될 코드를 꺼내와 실행하게 한다. Queue에서 실행된 코드가 성공적일 경우에는 다음 Queue에 들어있는 항목을 실행하게 된다.

편의성을 더하기 위해 일단 Queue에 넣어 두고 Queue에서 빼낼 때, 가장 우선 순위에 있는 Queue를 꺼내면 되는데, 이를 우선순위 큐(Priority Queue)라고 한다.

하나의 코드를 실행하기 위해서는 해당 코드의 우선순위(Priority)와 실행될 코드가 존재하면 된다.

*PriorityRunnalble.kt*

    private class PriorityRunnable(var priority: Int, var runnable: Runnable?): Comparable<PriorityRunnable> {
     
        override fun compareTo(other: PriorityRunnable): Int {
            return if (other.priority<= this.priority) { 1 } else { -1 }
        }
     
        fun run() { runnable?.run() }
    }
     
    class PriorityDispatchQueue {
     
        private val introQueue = PriorityQueue<PriorityRunnable>()
     
        fun push(priority: Int, runnable: () -> Unit) {
            introQueue.add(PriorityRunnable(priority, Runnable { runnable() }))
        }
     
        fun execNext(): Boolean {
            val runnable = introQueue.poll() ?: return false
            runnable.run(); return true
        }
    }

Queue에 넣는 하나의 단위는 `PriorityRunnable`이며 넣을 Queue에는 두가지 Method가 존재한다.

- `push`: Queue에 `PriorityRunnable`를 추가한다.
- `execNext`:다음 Queue에서 꺼내와 실행한다.

`execNext`는 코드 실행분기중 분기 성공때나 패스시에 실행한다.

Main코드에서는 Queue를 생성해주고 Queue에 각 항목을 Push해주면 된다. Push할 `PriorityRunnable`를 반환하는 Method를 만들면 다음과 같다.

    private fun login(): PriorityRunnable {
        return PriorityRunnable(LoginPriority.Login.rowValue, Runnable {
            // ...
            queue.execNext()
        })
    }

---

Main의 Push코드

    // Define Priorities
    enum class LoginPriority(val rowValue: Int) {
        Auth(1),
        Login(2),
        PrivacyPolicy(3),
        FetchServerList(4),
    }
     
    // Intro Class
    private val queue = PriorityDispatchQueue()
     
     override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
        
            queue.push(login())
     }

---

위의 방법으로 코드를 초기화 했을 경우에는 Privacy Policy, 인증, Intro Pop-up등의 순서제어가 손쉬웠다. 단순히 Define된 Priority의 상수값을 변경만 하면 되기 때문이다.

또한 하나의 Push되는 묶음으로 실행되기 때문에 다른 분기에 영향을 미칠 영향이 없다는 점이 장점이다.

만약 해당 분기가 무조건 통과되어야 한다거나 Fail시에 재시도를 해야 하는 경우에는 Queue에 현재 실행되고 있는 자기자신의 `PriorityRunnalbe`를 추가하고 `execNext()`를 호출하면 자연스럽게 다시 한번 Call을 하게된다.

재시도가 빈번할 경우에는 execNext()에서 같은 `priority`를 가질경우에는 Delay를 줌으로써 어느정도 해결이 가능하다.
