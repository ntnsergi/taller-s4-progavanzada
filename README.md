
# Promps usados para taller-s4

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
