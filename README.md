# django-expanded-model
The function for a recursive transformation of a model with foreign keys into a flat one

```python
from typing import Tuple, List


def extract_fields_and_columns(model) -> Tuple[List, List]:
    """function returns a list of values and a list of columns of model, including foreign models values and columns"""
    values, columns = [], []

    for f in model._meta.fields:
        # ref is a model from foreign key pointer
        value, ref = getattr(model, f.name), model._meta.get_field(f.name).remote_field
        if ref:
            # fill empty space with None values to save the dimension length
            if value is None:
                values.extend([None for i in ref.model._meta.fields])
                # combine names for better expirience, for example 'Car' + 'Engine' = 'Car.Engine'
                columns.extend([ref.model._meta.verbose_name + '.' + ref.model._meta.get_field(c.name).verbose_name for c in ref.model._meta.fields])
            else:
                # recursively get ref fields until all values are of a simple type
                vals, cols = extract_fields_and_columns(value)
                values.extend(vals)
                columns.extend([ref.model._meta.verbose_name + '.' + c for c in cols])
        else:
            values.append(value)
            columns.append(model._meta.get_field(f.name).verbose_name)

    return values, columns
```

### Example

```
class Engine(django.db.models.Model):
    name: str
    volume: float

class Car(django.db.models.Model):
    name: str
    brand: str
    engine: fk

extract_fields_and_columns(Car()) ->
(
    [Car.name, Car.brand, Engine.name, Engine.volume],
    ["X5", "BMW", "B47B20", 2.0]
)
```

### OR if Engine is None

```
extract_fields_and_columns(Car()) ->
(
    [Car.name, Car.brand, Engine.name, Engine.volume],
    ["X5", "BMW", None, None]
)
```

### Use Case

It is usefull fro exporting data into Excel from Django admin panel
