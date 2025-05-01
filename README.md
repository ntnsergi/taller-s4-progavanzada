
# Promps usados para taller-s4

## Problema 1
## Usuario:
**¿Puedes ayudarme describiendo que hace cada cosa de este programa?**

### Código:
```java
public class RCndtnExample {
    private static int counter = 0;

    public static void main(String[] args) throws InterruptedException {
      Runnable task = () -> {
          for(var i = 0; i < 10_000; i ++) {
              counter ++;
          }
      };
      var thread1 = new Thread(task);
      var thread2 = new Thread(task);

      thread1.start();
      thread2.start();

      thread1.join();
      thread2.join();

      System.out.printf("Counter: %d%n", counter);
    }
}
```

## ChatGPT:
¡Claro, te explico parte por parte lo que hace este programa Java! Este es un ejemplo clásico para ilustrar una **condición de carrera (race condition)** en programación concurrente. Vamos por partes:

1. `public class RCndtnExample {` - Define una clase pública llamada `RCndtnExample`.
2. `private static int counter = 0;` - Declara una variable compartida y estática `counter`, inicialmente en 0.
3. Se define un `Runnable` con una lambda que incrementa el `counter` 10,000 veces.
4. Se crean dos hilos que ejecutan la misma tarea.
5. Se inicia la ejecución de los hilos.
6. El método `join()` espera a que los hilos terminen.
7. Se imprime el valor final de `counter`.

El programa puede imprimir un número menor a 20,000 debido a una condición de carrera. Esto ocurre porque `counter++` no es una operación atómica.

¿Quieres que te muestre cómo corregir este programa usando una de esas soluciones?

---

## Usuario:
**¿Por qué sucede el error? Explícame por qué me da valores aleatorios como 11510 o 15983.**

## ChatGPT:
El error sucede por una **condición de carrera**. El problema está en `counter++`, que en realidad se compone de tres pasos: leer el valor, sumarle 1 y escribirlo de nuevo. Si dos hilos hacen esto al mismo tiempo, pueden interferirse y perder incrementos.

### Ejemplo:
- `counter = 100`
- Hilo 1 lee 100
- Hilo 2 también lee 100
- Ambos suman y escriben 101
- Resultado: solo se incrementó una vez

Esto puede pasar muchas veces aleatoriamente durante las 20,000 iteraciones, por eso ves resultados como 11,510 o 15,983.

### Soluciones:
1. Usar `AtomicInteger`
2. Usar `synchronized` para hacer el incremento seguro

¿Te gustaría que te muestre la versión corregida completa con una de esas técnicas?

---

## Usuario:
**Explícame cómo funciona `synchronized`**

## ChatGPT:
`synchronized` es una palabra clave que evita condiciones de carrera. Garantiza que **solo un hilo a la vez** pueda ejecutar un bloque o método crítico.

### Ejemplo:
```java
private static synchronized void increment() {
    counter++;
}
```

Esto significa que mientras un hilo ejecuta `increment()`, los demás deben esperar.

### Usos:
- Métodos **estáticos sincronizados** bloquean la **clase entera**.
- Métodos **no estáticos sincronizados** bloquean solo la **instancia actual**.

### Ejemplo completo con `synchronized`:
```java
public class RCndtnExample {
    private static int counter = 0;

    private static synchronized void increment() {
        counter++;
    }

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            for (int i = 0; i < 10_000; i++) {
                increment();
            }
        };

        var thread1 = new Thread(task);
        var thread2 = new Thread(task);

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.printf("Counter: %d%n", counter);
    }
}
```

Este código imprimirá siempre **Counter: 20000**.

¿Te gustaría que te explique cómo funciona el bloqueo con bloques `synchronized` también?

## Problema 2
## Usuario:
**Ayúdame explicándome el funcionamiento del programa y por qué no termina la ejecución**

### Código:
```java
public class DdlckExample {
    private static final Object lock1 = new Object();
    private static final Object lock2 = new Object();

    public static void main(String[] args){
      Thread thread1 = new Thread(() -> {
          synchronized (lock1) {
              System.out.println("Thread 1: Acquired lock1");
              try{ Thread.currentThread().sleep(100);} catch (InterruptedException _){}
              synchronized (lock2) {
                  System.out.println("Thread 1: Acquired lock2");
              }
          }
      });

      Thread thread2 = new Thread(() -> {
          synchronized (lock2) {
              System.out.println("Thread 2: Acquired lock2");
              try{ Thread.currentThread().sleep(100);} catch (InterruptedException _){}
              synchronized (lock1) {
                  System.out.println("Thread 2: Acquired lock1");
              }
          }
      });
      thread1.start();
      thread2.start();
    }
}
```

## ChatGPT:
Este programa es un ejemplo clásico de un **_deadlock_ (interbloqueo)**. La razón por la que **no termina su ejecución** es porque **ambos hilos quedan esperando indefinidamente** a que el otro libere un recurso que necesitan.

### 🔁 ¿Qué pasa exactamente en el código?

#### 🔒 Hilo 1:
1. Adquiere `lock1`
2. Espera 100 ms (`sleep`)
3. Intenta adquirir `lock2`

#### 🔒 Hilo 2:
1. Adquiere `lock2`
2. Espera 100 ms (`sleep`)
3. Intenta adquirir `lock1`

### 💥 ¿Dónde ocurre el problema?

- **Thread 1** entra primero, bloquea `lock1` y duerme.
- **Thread 2** entra casi al mismo tiempo, bloquea `lock2` y duerme también.
- Después de los 100 ms:
  - Thread 1 quiere `lock2`, pero ya lo tiene el Thread 2 → **espera**.
  - Thread 2 quiere `lock1`, pero ya lo tiene el Thread 1 → **espera**.

🎯 Ambos hilos están **esperando mutuamente** uno por el otro. **Ninguno libera su lock**, por lo que **el programa nunca avanza ni termina.**

### 📌 Resultado en consola:
```
Thread 1: Acquired lock1
Thread 2: Acquired lock2
```
Y luego se queda colgado.

### ✅ ¿Cómo evitar un deadlock?

Una regla sencilla es **adquirir los locks en el mismo orden** en todos los hilos:

```java
Thread thread2 = new Thread(() -> {
    synchronized (lock1) {
        System.out.println("Thread 2: Acquired lock1");
        try{ Thread.sleep(100); } catch (InterruptedException _) {}
        synchronized (lock2) {
            System.out.println("Thread 2: Acquired lock2");
        }
    }
});
```

Ahora ambos hilos usan el orden `lock1 → lock2`, por lo tanto no se bloquean entre sí.

---

## Usuario:
**¿Por qué el error se soluciona al ponerlos al mismo orden? puede que sea porque en caso de ponerlos en el mismo orden el hilo 1 
adquiere el lock 1 y despues adquiere el lock 2 libreando el uno y asi que el hilo 2 pueda adquirir lock 1 y asi seguir su proceso?**

## ChatGPT:
Buena observación, estás cerca, pero te aclaro con más precisión:

### ✅ ¿Por qué se soluciona el deadlock al usar el mismo orden?

Cuando **ambos hilos adquieren los locks en el mismo orden** (`lock1` luego `lock2`), **ya no pueden bloquearse mutuamente**, porque:

- Si **Thread 1** adquiere `lock1` primero, y luego `lock2`, **Thread 2 no podrá adquirir `lock1` hasta que Thread 1 lo libere.**
- Entonces **Thread 2 nunca llegará a bloquear `lock2` antes de `lock1`**, porque está esperando que se libere `lock1` primero.

### 🚫 En el caso original (orden opuesto):
- Thread 1: `lock1 → sleep → intenta lock2`
- Thread 2: `lock2 → sleep → intenta lock1`

Ambos se quedan **esperando un lock que el otro ya tiene**, y ninguno lo libera porque está esperando a adquirir el segundo.

### ✅ En el caso corregido (mismo orden `lock1 → lock2`):
- Thread 1: `lock1 → lock2`
- Thread 2: espera a que se libere `lock1`
- Cuando Thread 1 termina, **libera ambos locks**
- Entonces Thread 2 puede ejecutar todo **sin interbloqueo**

### 🧠 Conclusión:

El deadlock se evita porque **se elimina la posibilidad de que dos hilos bloqueen recursos en orden inverso**. Todos siguen la **misma jerarquía de adquisición de locks**, por lo que **uno siempre terminará antes de que el otro empiece**.

## Problema 3 

## Usuario:
### Puedes explicarme el funcionamiento de este programa y que ejemplo de error esta sucediendo aqui? 
```java
public class StrvtnExample {
    private static final ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args){
      Runnable task = () -> {
        while(true) {
            lock.lock();
            try{
                System.out.printf("%s acquired the lock.%n", Thread.currentThread().getName());
            } finally {
                lock.unlock();
            }
        }
      };

      for(var i = 0; i < 10; i++) {
          Thread t = new Thread(task, "Hilo-" + i);
          if( i % 2 == 0) t.setPriority(Thread.MAX_PRIORITY);
          else t.setPriority(Thread.MIN_PRIORITY);
          t.start();
      }
    }
}

```

## ChatGPT:
### Este programa es un ejemplo clásico del problema de starvation (inaniación o inanición de hilos), un error común en programación concurrente.

### 🧠 ¿Qué hace este programa?
Crea una instancia de ReentrantLock, una clase que permite controlar el acceso exclusivo a una sección crítica del código.

Define un Runnable que:

- Ejecuta un bucle infinito **(while(true))**

- Intenta adquirir el lock

- Si lo consigue, imprime qué hilo lo adquirió

- Luego libera el lock

Crea 10 hilos:

- Los hilos con índice par (Hilo-0, Hilo-2, etc.) tienen prioridad máxima

- Los hilos con índice impar tienen prioridad mínima

- Todos ejecutan el mismo task

### ⚠️ ¿Qué error sucede aquí? — Starvation
Starvation ocurre cuando uno o más hilos no pueden acceder al recurso compartido porque otros hilos acaparan el control continuamente.

En este programa:

- Los hilos de alta prioridad (pares) tienen más probabilidad de ser elegidos por el planificador de hilos (scheduler).

- Como el bucle es infinito y ellos acaparan el lock, los hilos de baja prioridad (impares) casi nunca logran ejecutarse, incluso si están listos.

- Resultado: los hilos de baja prioridad se “mueren de hambre” esperando el lock.

### 🖥️ ¿Qué verías en consola?
Probablemente verías algo como esto:
Hilo-0 acquired the lock.
Hilo-2 acquired the lock.
Hilo-4 acquired the lock.
Hilo-0 acquired the lock.
Hilo-6 acquired the lock.
...
Y casi nunca aparecerían los hilos impares (**Hilo-1**, **Hilo-3**, etc.).

### ✅ ¿Cómo evitar starvation?
1. Usar fair locks (locks justos):
   ```java
   private static final ReentrantLock lock = new ReentrantLock(true); // fair = true
   ```
    Esto hace que el lock respete el orden de llegada (FIFO).
2. Evitar usar bucles infinitos sin pausas (como **while(true)**)
3. No depender en exceso de la prioridad de hilos (en muchos sistemas, la prioridad es solo una “sugerencia” al scheduler)

## Problema 4
## Usuario: 
"Ayúdame explicado el funcionamiento que tiene este programa de una manera fácil de comprender"
## ChatGPT: 
Este programa tiene dos hilos (Thread-1 y Thread-2) que intentan adquirir dos locks (lock1 y lock2) de forma concurrente usando la clase ReentrantLock.

Cada hilo intenta:
1. Obtener el primer lock.
2. Esperar un poco (con Thread.sleep(50)).
3. Obtener el segundo lock.

Si no consigue el segundo lock, *libera el primero* y vuelve a intentarlo.

### 🔄 Objetivo:
Evitar un *deadlock*. Un deadlock ocurre cuando dos hilos se bloquean entre sí porque cada uno tiene un recurso que el otro necesita. En este caso, se evita liberando el primer lock si no se consigue el segundo.

## Usuario: 
"¿Cómo se podría optimizar el programa para que los hilos tarden menos tiempo en adquirir los locks?"

## ChatGPT: 
Se puede optimizar de las siguientes maneras:

### ✅ 1. *Unificar el orden de adquisición de locks*
En el código original, un hilo usa lock1 → lock2 y el otro lock2 → lock1. Esto crea riesgo de deadlock. Si ambos hilos usan el *mismo orden*, se elimina ese riesgo.

### ✅ 2. *Usar lock() en lugar de tryLock()*
tryLock() devuelve de inmediato si no puede adquirir el lock, lo que puede llevar a reintentos y pérdida de rendimiento. Si no hay riesgo de deadlock, lock() es más eficiente.

### ✅ 3. *Eliminar Thread.sleep() y Thread.yield()*
Estas instrucciones ralentizan el programa. Con un orden de locks uniforme, ya no son necesarias.

## Usuario: 
"A este código implementale la versión optimizada, ten en cuenta que attemptBoth recibe parámetros."

## ChatGPT: 
# Código Java para Evitar Deadlock utilizando ReentrantLock

```java
import java.util.concurrent.locks.ReentrantLock;

public class LvlckExample {
    private static final ReentrantLock lock1 = new ReentrantLock();
    private static final ReentrantLock lock2 = new ReentrantLock();

    public static void main(String[] args) {
        // Ambos hilos usan el mismo orden: lock1 → lock2
        Runnable r1 = () -> attemptBoth(lock1, lock2, "Thread-1");
        Runnable r2 = () -> attemptBoth(lock1, lock2, "Thread-2");

        new Thread(r1).start();
        new Thread(r2).start();
    }

    private static void attemptBoth(ReentrantLock first, ReentrantLock second, String name) {
        first.lock();
        System.out.println(name + " adquirió primer lock");

        second.lock();
        System.out.println(name + " adquirió segundo lock, va a proceder");

        try {
            System.out.println(name + " está trabajando en la sección crítica");
        } finally {
            second.unlock();
            first.unlock();
            System.out.println(name + " liberó ambos locks");
        }
    }
}
```
### 🔧 Mejoras implementadas:
- Orden fijo en adquisición de locks: todos los hilos usan el mismo orden lock1 → lock2 para eliminar riesgo de deadlock.

- Reemplazo de tryLock() por lock(): como no hay riesgo de bloqueo circular, podemos usar bloqueos normales (más rápidos).

- Eliminación del Thread.sleep(50) y yield(): ya no son necesarios si los hilos no compiten erráticamente.
