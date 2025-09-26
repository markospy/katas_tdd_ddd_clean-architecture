Perfecto 🚀. Aquí tienes la **versión extendida del kata**, con un enfoque más cercano a Clean Architecture y mayor complejidad para que practiques a fondo.

---

# 🥋 Kata extendido: Validación + Servicios de Aplicación

## 📌 Contexto

Este ejercicio extiende el kata anterior. Ahora no basta con separar validación de datos y reglas de negocio:

* Debemos introducir una **capa de aplicación** que orqueste las validaciones y la lógica de dominio.
* También añadimos un **patrón Strategy** para la validación, lo que permite intercambiar reglas de validación sin tocar el dominio.

---

## 🎯 Instrucciones

1. Mantén la validación de datos fuera del dominio, pero ahora encapsúlala en **estrategias de validación**.
2. Crea un servicio de aplicación `AllocationService` que:

   * Reciba datos de entrada.
   * Aplique validaciones de datos mediante estrategias configurables.
   * Si todo está bien, invoque el dominio (`allocate`).
3. Escribe **tests de integración** para el servicio.
4. El servicio debe devolver un **resultado uniforme** (ej. un diccionario con estado `success` o `error`).

---

## ⚖️ Restricciones

* No usar frameworks web ni librerías externas de validación (ni Pydantic).
* Aplicar **patrón Strategy** para validación de datos.
* Mantener separados dominio, validación y aplicación.
* Hacer TDD: primero los tests del servicio.

---

## 📊 Nivel

**Avanzado**
(porque ya combina arquitectura, patrones y testing).

---

## 🐍 Lenguaje

Python.

---

## 📝 Ejemplo inicial

### tests/test_allocation_service.py

```python
import pytest
from domain import Batch, OrderLine, OutOfStock
from application import AllocationService
from validation import NonEmptySKUValidator, PositiveQuantityValidator

@pytest.fixture
def service():
    validators = [NonEmptySKUValidator(), PositiveQuantityValidator()]
    return AllocationService(validators)

def test_rejects_invalid_input(service):
    result = service.allocate(orderid="o1", sku="", qty=5, batches=[])
    assert result["status"] == "error"
    assert "SKU no puede estar vacío" in result["message"]

def test_rejects_negative_qty(service):
    result = service.allocate(orderid="o1", sku="TABLE", qty=-5, batches=[])
    assert result["status"] == "error"
    assert "Cantidad debe ser positiva" in result["message"]

def test_allocates_successfully(service):
    batch = Batch("b1", "TABLE", 10)
    result = service.allocate(orderid="o1", sku="TABLE", qty=3, batches=[batch])
    assert result["status"] == "success"
    assert result["batch_ref"] == "b1"
    assert batch.available_quantity == 7

def test_raises_out_of_stock(service):
    batch = Batch("b1", "TABLE", 1)
    result = service.allocate(orderid="o1", sku="TABLE", qty=5, batches=[batch])
    assert result["status"] == "error"
    assert "Out of stock" in result["message"]
```

---

### application.py

```python
from domain import OrderLine, allocate, OutOfStock

class AllocationService:
    def __init__(self, validators):
        self.validators = validators

    def allocate(self, orderid, sku, qty, batches):
        # Validación de datos
        for validator in self.validators:
            error = validator.validate(sku=sku, qty=qty)
            if error:
                return {"status": "error", "message": error}

        # Construir objeto de dominio
        line = OrderLine(orderid, sku, qty)

        # Validación de reglas de negocio
        try:
            batch_ref = allocate(line, batches)
            return {"status": "success", "batch_ref": batch_ref}
        except OutOfStock as e:
            return {"status": "error", "message": str(e)}
```

---

### validation.py

```python
from abc import ABC, abstractmethod

class Validator(ABC):
    @abstractmethod
    def validate(self, sku: str, qty: int) -> str | None:
        """Devuelve un mensaje de error o None si es válido."""

class NonEmptySKUValidator(Validator):
    def validate(self, sku: str, qty: int) -> str | None:
        if not sku or not sku.strip():
            return "SKU no puede estar vacío"
        return None

class PositiveQuantityValidator(Validator):
    def validate(self, sku: str, qty: int) -> str | None:
        if qty <= 0:
            return "Cantidad debe ser positiva"
        return None
```

---

## ✅ Criterio de éxito

* Los **tests de integración del servicio** pasan.
* Puedes cambiar validadores sin tocar el dominio.
* El servicio devuelve respuestas uniformes (`{"status": ..., "message"/"batch_ref": ...}`).
* El dominio nunca valida datos de entrada: solo reglas de negocio.

---

## 🚀 Versión extendida plus (para ir todavía más lejos)

1. Añade un **validador configurable por JSON** (ej. longitudes mínimas/máximas).
2. Implementa un **logger** en la capa de aplicación que registre validaciones fallidas y excepciones de dominio.
3. Crea un test que simule un “controller” (API ficticia) que llama al servicio.

---

👉 ¿Quieres que en la siguiente iteración te prepare un **diagrama de capas con el flujo del AllocationService**, similar al que te hice antes para validaciones?
