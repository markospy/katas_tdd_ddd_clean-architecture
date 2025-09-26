Aquí tienes un **ejercicio estilo kata** basado en lo que hemos trabajado (validación, separación de responsabilidades, TDD, clean architecture y el apéndice de *Cosmic Python*).

---

# 🥋 Kata: Separación de Validación de Datos y Reglas de Negocio

## 📌 Contexto

Este ejercicio surge del **apéndice de validación** del libro *Architecture Patterns with Python*. El texto plantea la importancia de **separar la validación de datos inválidos** (ej. campos vacíos, tipos incorrectos) de la **validación de reglas de negocio** (ej. no asignar más items de los que un lote permite).
El reto es practicar esta separación implementando **tests y código** que reflejen esa arquitectura.

---

## 🎯 Instrucciones

1. Implementa una función `allocate` que asigna una orden (`OrderLine`) a un lote (`Batch`).
2. Antes de llegar al dominio, valida los datos con una capa de validación (ejemplo: un DTO o una función `validate_input`).
3. Dentro del dominio (`Batch`), implementa la validación de reglas de negocio:

   * Un `Batch` no puede asignar más ítems de los que tiene en stock.
   * Si no hay lotes disponibles para un `sku`, se lanza una excepción de dominio `OutOfStock`.
4. Usa **TDD**: primero escribe los tests, luego implementa la lógica.
5. Mantén la **separación de capas**: validación de datos fuera del dominio, reglas de negocio dentro.

---

## ⚖️ Restricciones

* No usar librerías externas de validación (ej. Pydantic). Todo debe implementarse con Python puro.
* Seguir la filosofía de **Clean Architecture**: dominio independiente, validaciones de entrada fuera.
* Escribir tests unitarios antes de la implementación (mínimo 3 casos de datos inválidos y 2 de negocio).

---

## 📊 Nivel

**Intermedio**
(ya no es un “hello world”, pero tampoco llega a patrones avanzados de infraestructura).

---

## 🐍 Lenguaje

Python.

---

## 📝 Ejemplo inicial

### test_allocate.py

```python
import pytest
from domain import Batch, OrderLine, allocate, OutOfStock
from validation import validate_input, InvalidData

def test_rejects_empty_sku():
    with pytest.raises(InvalidData):
        validate_input(sku="", qty=10)

def test_rejects_negative_qty():
    with pytest.raises(InvalidData):
        validate_input(sku="CHAIR", qty=-5)

def test_allocate_to_batch_reduces_available_quantity():
    batch = Batch("batch-001", "TABLE", 20)
    line = OrderLine("order-001", "TABLE", 5)
    allocate(line, [batch])
    assert batch.available_quantity == 15

def test_raises_out_of_stock_if_no_batches_available():
    batch = Batch("batch-001", "TABLE", 0)
    line = OrderLine("order-001", "TABLE", 1)
    with pytest.raises(OutOfStock):
        allocate(line, [batch])
```

### domain.py

```python
class OutOfStock(Exception):
    pass

class OrderLine:
    def __init__(self, orderid, sku, qty):
        self.orderid = orderid
        self.sku = sku
        self.qty = qty

class Batch:
    def __init__(self, ref, sku, qty):
        self.ref = ref
        self.sku = sku
        self._purchased_quantity = qty
        self._allocations = set()

    @property
    def available_quantity(self):
        return self._purchased_quantity - sum(l.qty for l in self._allocations)

    def can_allocate(self, line):
        return self.sku == line.sku and self.available_quantity >= line.qty

    def allocate(self, line):
        if self.can_allocate(line):
            self._allocations.add(line)

def allocate(line, batches):
    for batch in batches:
        if batch.can_allocate(line):
            batch.allocate(line)
            return batch.ref
    raise OutOfStock(f"Out of stock for sku {line.sku}")
```

### validation.py

```python
class InvalidData(Exception):
    pass

def validate_input(sku: str, qty: int):
    if not sku or not sku.strip():
        raise InvalidData("SKU vacío")
    if qty <= 0:
        raise InvalidData("Cantidad debe ser positiva")
    return {"sku": sku, "qty": qty}
```

---

## ✅ Criterio de éxito

* Todos los tests deben pasar.
* La lógica de validación de datos está **separada** del dominio.
* El dominio solo maneja reglas de negocio, no validaciones de formato.

---

## 🚀 Versión extendida (opcional)

* Añade soporte para múltiples validaciones de datos (ej. tipos, longitud máxima).
* Implementa una capa de **servicio de aplicación** que reciba el input del usuario, lo valide y luego invoque al dominio.
* Extiende `allocate` para que devuelva un DTO con el resultado (referencia del batch asignado o error correspondiente).
* Refactoriza usando un patrón de diseño (ej. **Strategy** para validaciones).

---
