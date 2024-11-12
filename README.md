# Техническое задание

## 1. Название проекта
**Многопоточный брокер сообщений (Kafka-like)**

## 2. Описание проекта
Необходимо создать бэкенд-систему, которая будет обеспечивать управление потоками сообщений между клиентами с использованием многопоточности. Основные функции должны включать обработку сообщений с высокой производительностью, гарантированную доставку и управление очередями сообщений. Система должна обеспечивать изоляцию и синхронизацию между потоками.

## 3. Требования к функционалу

1. **API для работы с сообщениями**
   - Создание очереди сообщений.
   - Отправка сообщений в очередь.
   - Получение сообщений из очереди.
   - Удаление сообщений.
   - Подписка на очередь (можно использовать механизм очередей с подписчиками).

2. **Поддержка многопоточности**:
   - Реализация обработки сообщений с использованием многопоточности.
   - Применение синхронизации для защиты данных и состояний очередей.
   - Использование канкурентных коллекций для обработки сообщений (например, `ConcurrentLinkedQueue`, `BlockingQueue` и т.д.).

3. **Механизмы синхронизации**:
   - Реализовать механизм блокировки для некоторых операций с очередями.
   - Использовать `ReentrantLock`, `Semaphore`, `CountDownLatch` или другие синхронизаторы для управления доступом к данным.

4. **Конкурентные коллекции**:
   - Для хранения сообщений использовать коллекции, поддерживающие безопасную работу в многопоточном окружении.
   - Например: `ConcurrentLinkedQueue`, `BlockingQueue`.

5. **Обработка ошибок и исключений**:
   - Протоколирование ошибок.
   - Надежная обработка сбоев, таких как потеря соединения или переполнение очереди.

6. **Производительность**:
   - Оптимизация работы с большими объемами данных, используя многопоточность и параллелизм.
   - Тестирование на производительность с высокими нагрузками.

## 4. Архитектура

1. **Клиенты (Producer/Consumer)**:
   - Клиенты должны иметь возможность отправлять сообщения в систему и получать их с определенной задержкой или сразу.
   - Реализовать механизм потребителей и продюсеров с использованием многопоточности (например, несколько производителей могут работать с одной очередью).

2. **Система очередей**:
   - Очереди сообщений должны быть реализованы с использованием потокобезопасных структур данных.
   - Каждая очередь должна иметь возможность поддерживать несколько подписчиков (потребителей).

3. **API**:
   - Взаимодействие с клиентами через HTTP эндпоинты (например, с использованием Spring Boot или другого фреймворка).
   - Эндпоинты:
     - `/createQueue` - создание новой очереди.
     - `/sendMessage` - отправка сообщения в очередь.
     - `/receiveMessage` - получение сообщения из очереди.
     - `/subscribe` - подписка на очередь.

4. **Асинхронная обработка**:
   - Для повышения производительности, использование асинхронной обработки запросов (например, через `CompletableFuture`).

## 5. Технологии и инструменты

- **Язык программирования**: Java 17 или выше.
- **Библиотеки и фреймворки**:
  - Spring Boot (для построения REST API).
  - Использование стандартных Java-конкурентных коллекций и синхронизаторов.
  - Для тестирования: JUnit, Mockito.
- **Интерфейс API**:
  - RESTful API для взаимодействия с клиентами.
  - JSON-формат для передачи данных.

## 6. Пример реализации

1. **Producer**: Это класс, который отправляет сообщения в очередь.
   ```java
   public class Producer implements Runnable {
       private final BlockingQueue<String> queue;

       public Producer(BlockingQueue<String> queue) {
           this.queue = queue;
       }

       @Override
       public void run() {
           try {
               while (true) {
                   String message = "Message at " + System.currentTimeMillis();
                   queue.put(message);
                   System.out.println("Produced: " + message);
                   Thread.sleep(1000);  // Имитируем задержку.
               }
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
           }
       }
   }
   ```

2. **Consumer**: Это класс, который извлекает сообщения из очереди.

    ```java
    public class Consumer implements Runnable {
        private final BlockingQueue<String> queue;
    
        public Consumer(BlockingQueue<String> queue) {
            this.queue = queue;
        }
    
        @Override
        public void run() {
            try {
                while (true) {
                    String message = queue.take();
                    System.out.println("Consumed: " + message);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    ```

3. REST API: Эндпоинты, через которые клиент взаимодействует с системой.

    ``` java
    @RestController
    public class MessageController {
    
        private final QueueService queueService;
    
        public MessageController(QueueService queueService) {
            this.queueService = queueService;
        }
    
        @PostMapping("/createQueue")
        public ResponseEntity<String> createQueue(@RequestBody String queueName) {
            queueService.createQueue(queueName);
            return ResponseEntity.ok("Queue created successfully");
        }
    
        @PostMapping("/sendMessage")
        public ResponseEntity<String> sendMessage(@RequestParam String queueName, @RequestBody String message) {
            queueService.sendMessage(queueName, message);
            return ResponseEntity.ok("Message sent");
        }
    
        @GetMapping("/receiveMessage")
        public ResponseEntity<String> receiveMessage(@RequestParam String queueName) {
            String message = queueService.receiveMessage(queueName);
            return ResponseEntity.ok(message);
        }
    }
    ```

7. Важные моменты
Многопоточность:
Все обработчики (продюсеры и потребители) должны работать в отдельных потоках, для чего используйте ExecutorService или другие механизмы.
Тестирование:
Необходимо реализовать юнит-тесты для проверки работы очередей, синхронизации и API.
   
