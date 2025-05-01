
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
