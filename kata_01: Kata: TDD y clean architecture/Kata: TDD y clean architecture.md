# TDD y clean architecture

**User:** Anonymous (marcos.seocuba@gmail.com)  
**Created:** 9/25/2025 15:47:04  
**Updated:** 9/25/2025 16:11:21  
**Exported:** 9/25/2025 16:16:10  
**Link:** [https://chatgpt.com/g/g-p-68d36e850ec881918daae17b1a1bb517-architecture-patterns-with-python/c/68d59c37-691c-832d-b996-cb990d8d2fa2](https://chatgpt.com/g/g-p-68d36e850ec881918daae17b1a1bb517-architecture-patterns-with-python/c/68d59c37-691c-832d-b996-cb990d8d2fa2)  

## Prompt:
Ayudame a entender este articulo. Si es necesario adapta los ejemplos a python ya que es el lenguaje que mejor domino y dime cual es el estilo que predomina hoy en dia, especialmente al aplicar TDD en clean architecture.

https://martinfowler.com/articles/mocksArentStubs.html

## Response:
Perfecto, ese artículo de Martin Fowler **“Mocks Aren’t Stubs”** es un clásico sobre TDD y el uso de _test doubles_ (dobles de prueba). Vamos paso a paso para entenderlo, adaptándolo a Python y relacionándolo con Clean Architecture.

* * *

1\. Contexto del artículo
-------------------------

Fowler explica que en TDD no todos los equipos prueban igual. Existen dos “escuelas” principales:

*   **Classic TDD** (a veces llamada “Detroit school” o “Chicago school”)
    *   Se enfoca en pruebas de caja negra: probar que los resultados del sistema son los esperados.
    *   No suele usar _mocks_ ni fakes, excepto cuando es realmente necesario (por ejemplo, una base de datos lenta o una API externa).
    *   Prefiere testear a nivel de integración, incluso usando la implementación real de los objetos.
*   **Mockist TDD** (también llamada “London school”)
    *   Se enfoca en el comportamiento y en las interacciones entre objetos.
    *   Usa _mocks_ para definir expectativas sobre cómo deben colaborar los objetos.
    *   Divide el sistema en componentes aislados y prueba cada uno con dobles de prueba de sus dependencias.

* * *

2\. Tipos de _Test Doubles_
---------------------------

Fowler lista varios tipos (basado en Gerard Meszaros):

*   **Dummy**: un objeto que se pasa pero nunca se usa.
*   **Fake**: una implementación simplificada que funciona, pero no es de producción (ej: una base de datos en memoria).
*   **Stub**: devuelve valores prefijados para que el test siga.
*   **Spy**: registra lo que pasó (ej: cuántas veces se llamó un método).
*   **Mock**: no solo sustituye, sino que además especifica expectativas sobre cómo debe usarse (y el test falla si no se cumplen).

* * *

3\. Ejemplo en Python
---------------------

Supongamos que estamos construyendo un servicio de pagos en una app (contexto común en Clean Architecture):

### Classic TDD (Chicago school)

```python
# Dominio
class PaymentProcessor:
    def charge(self, amount: int, card_number: str) -> bool:
        # Implementación real que llamaría a un gateway externo
        raise NotImplementedError

class OrderService:
    def __init__(self, payment_processor: PaymentProcessor):
        self.payment_processor = payment_processor

    def place_order(self, amount: int, card_number: str) -> str:
        if self.payment_processor.charge(amount, card_number):
            return "success"
        return "failed"
```

En la escuela clásica, se escribiría un test integrando un **fake**:

```python
class FakePaymentProcessor(PaymentProcessor):
    def charge(self, amount: int, card_number: str) -> bool:
        return True  # siempre funciona en test

def test_place_order_success():
    processor = FakePaymentProcessor()
    service = OrderService(processor)

    result = service.place_order(100, "4111111111111111")

    assert result == "success"
```

👉 Aquí no nos importa cómo `charge` fue llamado, solo el resultado final.

* * *

### Mockist TDD (London school)

Aquí sí importan las interacciones:

```python
from unittest.mock import Mock

def test_place_order_success():
    mock_processor = Mock()
    mock_processor.charge.return_value = True

    service = OrderService(mock_processor)
    result = service.place_order(100, "4111111111111111")

    mock_processor.charge.assert_called_once_with(100, "4111111111111111")
    assert result == "success"
```

👉 Aquí **verificamos explícitamente** que el método `charge` fue llamado con ciertos parámetros.

* * *

4\. Relación con Clean Architecture
-----------------------------------

En Clean Architecture:

*   El **dominio** debe estar aislado de frameworks, librerías o servicios externos.
*   Esto encaja muy bien con **fakes** o **stubs** en la capa de infraestructura.
*   Por eso, muchos equipos que hacen Clean Architecture tienden a usar un **enfoque más clásico**:
    *   Tests de dominio puros (sin mocks).
    *   Dobles de prueba solo en puertos externos (bases de datos, APIs, mensajería).

El enfoque “mockist” puede llevar a tests frágiles y acoplados a la implementación, lo cual va en contra de la flexibilidad buscada en Clean Architecture.

* * *

5\. ¿Qué estilo predomina hoy?
------------------------------

En la práctica moderna (2020s–2025):

*   La **escuela clásica** es la más usada en proyectos serios de Clean Architecture.
*   Se recomienda **no abusar de los mocks**, porque producen **tests frágiles** y difíciles de mantener.
*   La tendencia actual es:
    *   Usar **tests de dominio puros** sin dobles.
    *   Usar **fakes** para persistencia (ej. repositorios en memoria).
    *   Usar **mocks/stubs** solo en interacciones con el “mundo exterior” (APIs, colas, emails).

Ejemplo:  
En un microservicio con FastAPI + Clean Architecture, se tiende a probar la lógica de dominio con fakes en memoria, y solo se mockean integraciones externas como un servicio de terceros de pagos.

* * *

✅ En resumen:

*   Fowler aclara que _mock_ es un tipo específico de _test double_, no un sinónimo de todos.
*   Dos estilos: **Classic (resultados)** vs **Mockist (interacciones)**.
*   Hoy predomina el **Classic TDD**, especialmente en Clean Architecture, porque produce tests más robustos y menos acoplados.

* * *

¿Quieres que te arme un **mini ejemplo completo estilo Clean Architecture** (con domain, application e infrastructure) usando **fakes en dominio** y **mocks en infraestructura** para que quede cristal claro cómo se aplica?

## Prompt:
Si, por favor. Pon el ejemplo.

## Response:
¡Perfecto 🚀!  
Vamos a armar un ejemplo **mini pero completo** con Clean Architecture en Python, aplicando TDD y usando fakes/mocks de la manera recomendada hoy en día.

* * *

📌 Escenario
============

Un caso de uso clásico: **sistema de pedidos**.

*   El **dominio** define el pedido (`Order`) y la interfaz de repositorio (`OrderRepository`).
*   La **aplicación** define el caso de uso (`PlaceOrderService`).
*   La **infraestructura** implementa un repositorio (ej. en memoria o base de datos real).
*   En los **tests**:
    *   Usaremos un **fake** de repositorio para testear el dominio y el caso de uso.
    *   Usaremos **mocks** solo si queremos probar la integración con servicios externos (ej. email).

* * *

🏗 1. Dominio
=============

```python
# domain/models.py
from dataclasses import dataclass
from typing import Protocol

@dataclass
class Order:
    id: str
    product: str
    quantity: int
    status: str = "pending"

# domain/repositories.py
class OrderRepository(Protocol):
    def save(self, order: Order) -> None: ...
    def get(self, order_id: str) -> Order: ...
```

👉 Notas:

*   Usamos `Protocol` (PEP 544) para definir interfaces (puertos).
*   El dominio no sabe nada de infraestructura.

* * *

🏗 2. Aplicación (caso de uso)
==============================

```python
# application/place_order.py
from domain.models import Order
from domain.repositories import OrderRepository

class PlaceOrderService:
    def __init__(self, repo: OrderRepository):
        self.repo = repo

    def execute(self, order_id: str, product: str, quantity: int) -> Order:
        order = Order(id=order_id, product=product, quantity=quantity)
        self.repo.save(order)
        return order
```

👉 El caso de uso solo depende de la abstracción `OrderRepository`.

* * *

🏗 3. Infraestructura (fake y real)
===================================

### Fake (para tests)

```python
# infrastructure/fake_repo.py
from domain.models import Order
from domain.repositories import OrderRepository

class FakeOrderRepository(OrderRepository):
    def __init__(self):
        self._storage = {}

    def save(self, order: Order) -> None:
        self._storage[order.id] = order

    def get(self, order_id: str) -> Order:
        return self._storage[order_id]
```

### (Ejemplo real: PostgreSQL)

Aquí no implemento todo, pero sería una clase que usa SQLAlchemy/SQLModel para persistir.

* * *

🧪 4. Tests con Fakes (Classic TDD)
===================================

```python
# tests/test_place_order.py
from application.place_order import PlaceOrderService
from infrastructure.fake_repo import FakeOrderRepository

def test_place_order_creates_new_order():
    repo = FakeOrderRepository()
    service = PlaceOrderService(repo)

    order = service.execute("order-123", "Laptop", 2)

    assert order.id == "order-123"
    assert order.status == "pending"
    assert repo.get("order-123").product == "Laptop"
```

👉 Aquí NO usamos mocks, solo un **fake repo**.  
Estamos probando la **lógica del caso de uso y dominio**.

* * *

🧪 5. Tests con Mocks (solo en bordes)
======================================

Ejemplo: el caso de uso envía un email al confirmar un pedido.  
Aquí sí tiene sentido verificar la interacción.

```python
# application/confirm_order.py
from domain.models import Order
from domain.repositories import OrderRepository

class ConfirmOrderService:
    def __init__(self, repo: OrderRepository, email_sender):
        self.repo = repo
        self.email_sender = email_sender

    def execute(self, order_id: str):
        order = self.repo.get(order_id)
        order.status = "confirmed"
        self.repo.save(order)
        self.email_sender.send(order)
        return order
```

### Test con mock

```python
# tests/test_confirm_order.py
from unittest.mock import Mock
from domain.models import Order
from infrastructure.fake_repo import FakeOrderRepository
from application.confirm_order import ConfirmOrderService

def test_confirm_order_sends_email():
    repo = FakeOrderRepository()
    order = Order(id="order-123", product="Book", quantity=1)
    repo.save(order)

    mock_email = Mock()
    service = ConfirmOrderService(repo, mock_email)

    confirmed = service.execute("order-123")

    assert confirmed.status == "confirmed"
    mock_email.send.assert_called_once_with(confirmed)
```

👉 Aquí sí validamos que se llamó `send()` en el colaborador externo (`email_sender`).

* * *

🔑 Conclusión del ejemplo
=========================

1.  **Dominio limpio:** sin dependencias de infraestructura.
2.  **Fakes** en tests del dominio y aplicación.
3.  **Mocks** solo en bordes (ej. emails, pagos, colas).
4.  Estilo predominante en Clean Architecture + TDD hoy: **Classic TDD con fakes**, y mocks solo cuando son necesarios.

* * *

¿Quieres que arme una **versión extendida** de este ejemplo con FastAPI como capa de interfaz, para ver cómo encajan estos tests dentro de un flujo real (API → aplicación → dominio → infraestructura)?

## Prompt:
Antes de pasar a la version extendida crea un kata que me sirva para practicar esto.

## Response:
¡Excelente idea 💡!  
Te propongo un **kata práctico** que refuerza justo lo que vimos: practicar **dobles de prueba en Clean Architecture con TDD**.

* * *

🥋 **Kata: Librería Online (Pedidos de Libros)**
================================================

### Contexto

Queremos construir el núcleo de una librería online.  
Un cliente puede **hacer un pedido** de un libro y luego **confirmar el pedido**, lo cual dispara un email de confirmación.

* * *

🎯 Objetivos de la práctica
---------------------------

1.  **Dominar fakes** en la capa de repositorios.
2.  **Practicar mocks** en integraciones externas (ej. email).
3.  **Separar dominio, aplicación e infraestructura** siguiendo Clean Architecture.
4.  **Escribir primero los tests** (TDD).

* * *

🏗 Reglas del negocio
---------------------

1.  Un **pedido (Order)** tiene:
    *   `id`
    *   `book_title`
    *   `quantity`
    *   `status` (default `"pending"`)
2.  Un **servicio de aplicación** debe permitir:
    *   `PlaceOrderService`: crear un pedido.
    *   `ConfirmOrderService`: confirmar un pedido → cambia `status` a `"confirmed"` y envía un email.
3.  El **repositorio** debe guardar y recuperar pedidos.
4.  El **servicio de email** solo necesita un método `send(order)`.

* * *

📚 Ejercicios paso a paso
-------------------------

### Paso 1: Modelar el dominio

*   Crea la entidad `Order`.
*   Define el contrato `OrderRepository`.
*   Haz un **fake** `FakeOrderRepository` (solo un diccionario en memoria).
*   ✅ Escribe un test que verifique que un pedido se guarda y recupera bien.

* * *

### Paso 2: Caso de uso `PlaceOrderService`

*   Implementa el servicio que recibe `(id, book_title, quantity)` y crea un pedido en estado `"pending"`.
*   ✅ Escribe un test con el fake repo verificando que el pedido se guardó correctamente.

* * *

### Paso 3: Caso de uso `ConfirmOrderService`

*   Implementa un servicio que:
    1.  Obtiene un pedido por ID.
    2.  Cambia su estado a `"confirmed"`.
    3.  Llama a `email_sender.send(order)`.
*   ✅ Escribe un test usando:
    *   `FakeOrderRepository` para la persistencia.
    *   `Mock` de Python (`unittest.mock`) para el email.
    *   Verifica que:
        *   El estado cambió.
        *   El email fue enviado.

* * *

### Paso 4 (extra): Edge cases

*   Intentar confirmar un pedido inexistente debe lanzar una excepción.
*   Intentar hacer un pedido con `quantity <= 0` debe ser inválido.
*   ✅ Agrega tests que cubran estas reglas.

* * *

🔄 Rutina de práctica (kata estilo TDD)
---------------------------------------

1.  Elige un paso (ej. Paso 1).
2.  Escribe primero el **test** (rojo).
3.  Escribe el mínimo código para pasar el test (verde).
4.  Refactoriza si es necesario.
5.  Repite con el siguiente paso.

* * *

📌 Ejemplo de test inicial (Paso 2)
-----------------------------------

```python
def test_place_order_creates_order_in_repo():
    repo = FakeOrderRepository()
    service = PlaceOrderService(repo)

    order = service.execute("order-1", "Clean Architecture", 1)

    assert order.id == "order-1"
    assert repo.get("order-1").book_title == "Clean Architecture"
    assert order.status == "pending"
```

* * *

Este kata lo puedes repetir varias veces, agregando reglas de negocio poco a poco (descuentos, cancelaciones, notificaciones, etc.), pero **siempre manteniendo la separación de capas y practicando fakes/mocks según el caso**.

* * *

👉 ¿Quieres que te arme también un **guion paso a paso en TDD** (es decir, qué test escribir primero, luego qué mínimo código, etc.) para que lo sigas como si fuera una rutina de dojo?



---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)