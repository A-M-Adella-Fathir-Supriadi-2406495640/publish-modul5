# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. In BambangShop, all subscribers share the same fixed structure (url and name) and behave
identically. There is no need for polymorphism since there is only one type of subscriber.
Therefore, a single Model struct is sufficient. A trait would only be necessary if there
were multiple subscriber types with different behaviors that needed to be used
interchangeably.
2. Since url must be unique, using a Vec would require O(n) linear search for lookups,
duplicate checks, and deletions. DashMap provides O(1) average-case operations using the
url as a key, making it both more correct and more efficient. DashMap is necessary here
to enforce uniqueness and maintain good performance.
3. We need both. The Singleton pattern (implemented via lazy_static!) ensures only one global
instance of SUBSCRIBERS exists. However, Singleton alone does not provide thread safety.
DashMap is a concurrent HashMap that allows safe multi-threaded reads and writes without
explicit locking. Without DashMap, we would need to wrap a regular HashMap in a Mutex or
RwLock, which works but is less performant. So DashMap and Singleton complement each other:
Singleton for single instance, DashMap for thread-safe concurrent access.

#### Reflection Publisher-2
1. Separating Service and Repository from the Model follows the Single Responsibility
Principle (SRP). If the Model handles data storage, business logic, and data access all at
once, the class becomes bloated and hard to maintain. Repository is responsible only for
data access and persistence, while Service handles business logic and orchestration. This
separation makes each component easier to test, modify, and reuse independently. For
example, we can swap the storage backend in Repository without touching the business logic
in Service.
2. If we only use the Model, each model would need to know about and directly interact with
the others. For example, the Program model would need to handle subscriber management and
notification sending. Subscriber would need to know about Programs it is subscribed to.
Notification would need to fetch subscriber lists and trigger HTTP calls. This creates
tight coupling between models, making the code harder to understand, test, and change.
Any modification in one model could break the others, and the models would grow very large
and complex over time.
3. Postman is a very useful API testing tool. It allows me to send HTTP requests (GET, POST,
DELETE, etc.) to my running Rocket server and immediately see the responses without
needing to write a frontend. For this project, I used Postman to test the subscribe and
unsubscribe endpoints by sending POST requests with JSON bodies. Postman features that I
find helpful include: saving requests in collections so they can be reused, setting
environment variables for base URLs, viewing response status codes and bodies in a clean
format, and being able to share collections with teammates. For group projects, sharing a
Postman collection would make it much easier for everyone to test the API consistently.

#### Reflection Publisher-3
1. In this tutorial, we use the Push model. When a product is created, deleted, or promoted,
the publisher (BambangShop) immediately pushes the notification data directly to each
subscriber by making an HTTP POST request to the subscriber's URL. The subscriber does
not need to ask or poll for updates — the data is sent automatically.
2. If we used the Pull model instead, subscribers would need to periodically request
(poll) the publisher to check for new notifications. 

Advantages of Pull model:
- Subscribers have control over when they fetch data, so they are not overwhelmed by
  sudden bursts of notifications.
- The publisher does not need to know the subscriber's URL or send HTTP requests,
  making the publisher simpler.
- Subscribers can choose what data they want to fetch, reducing unnecessary data transfer.

Disadvantages of Pull model:
- Subscribers may receive notifications with a delay depending on how frequently they
  poll, making it less real-time.
- If there are many subscribers polling frequently, it can put a heavy load on the
  publisher's server.
- More complex logic is needed on the subscriber side to implement polling and track
  what has already been fetched.
3. If we do not use multi-threading, the notify function would send HTTP POST requests to
each subscriber one by one, sequentially. This means the program would block and wait
for each HTTP request to complete before moving on to the next subscriber. If there are
many subscribers or if some subscribers are slow to respond, the entire notification
process would take a very long time. During this time, the main thread would be blocked,
making the server unable to handle other incoming requests. This would significantly
degrade performance and responsiveness. With multi-threading (thread::spawn), each
subscriber is notified in a separate thread concurrently, so the main thread is not
blocked and the server remains responsive.

