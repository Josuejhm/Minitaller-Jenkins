# Tutorial: flujo completo con Jenkins

Este tutorial te guia por los dos flujos clave de CI con Jenkins:
un commit que **rompe** el pipeline y uno que lo **arregla**.

No necesitas entender el codigo del juego. Solo copia, pega y sube los cambios.

---

## Parte 1: subir un test con error

Abre el archivo `tests/test_domain.py` y agrega el siguiente bloque
**al final del archivo**, despues de todas las clases existentes:

```python
class TestTableroFacil(unittest.TestCase):
    def test_cantidad_de_minas_en_dificultad_facil(self):
        board = generate_board(8, 8, 10, (0, 0), rng=random.Random(42))
        mine_count = sum(cell == MINE for row in board for cell in row)
        self.assertEqual(mine_count, 5)  # <-- error intencional
```

> El test verifica cuantas minas tiene el tablero. El valor correcto es `10`,
> pero aqui dice `5`, asi que va a fallar.

Guarda el archivo y sube el cambio:

```bash
git add tests/test_domain.py
git commit -m "Tutorial: agregar test con error"
git push
```

Espera un momento y revisa Jenkins. El resultado esperado es:

```
Checkout        -> verde
Static Analysis -> verde
Unit Tests      -> rojo
Build App Image -> no se ejecuta
```

Jenkins detecto que el comportamiento esperado no se cumple y bloqueo
la construccion de la imagen.

---

## Parte 2: corregir el error

En el mismo archivo `tests/test_domain.py`, busca la linea que dice:

```python
        self.assertEqual(mine_count, 5)  # <-- error intencional
```

Y reemplazala por:

```python
        self.assertEqual(mine_count, 10)
```

Guarda y sube la correccion:

```bash
git add tests/test_domain.py
git commit -m "Tutorial: corregir test de minas"
git push
```

Espera un momento y revisa Jenkins. El resultado esperado es:

```
Checkout        -> verde
Static Analysis -> verde
Unit Tests      -> verde
Build App Image -> verde
```

---

## Que demostro este flujo

```
commit roto  ->  Jenkins falla  ->  imagen no se genera
     |
     v
commit fix   ->  Jenkins pasa   ->  imagen lista
```

Un pipeline de CI actua como barrera automatica: si el codigo no cumple
lo que los tests esperan, la version no avanza. Cuando se corrige,
el pipeline lo valida y continua.
