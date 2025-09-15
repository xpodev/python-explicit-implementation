# Explicit Implementation

[![PyPI version](https://badge.fury.io/py/explicit-implementation.svg)](https://badge.fury.io/py/explicit-implementation)
[![Python Support](https://img.shields.io/pypi/pyversions/explicit-implementation.svg)](https://pypi.org/project/explicit-implementation/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A Python library that provides explicit interface implementations with compile-time safety and clear separation between interface contracts and their implementations.

## Features

- **Explicit Interface Declarations**: Define clear interface contracts using abstract methods
- **Flexible Implementation**: Allow partial implementations by default, enforce completeness with `concrete=True`
- **Compile-time Safety**: Catch implementation errors at class definition time, not runtime
- **Multiple Interface Support**: Implement multiple interfaces in a single class
- **Interface Access Control**: Access implementations through specific interface views
- **Type Safety**: Full typing support with generics for better IDE experience
- **Diamond Inheritance**: Proper handling of complex inheritance patterns

## Installation

```bash
pip install explicit-implementation
```

## Quick Start

```python
from explicit_implementation import Interface, abstractmethod, implements

# Define an interface
class IDrawable(Interface):
    @abstractmethod
    def draw(self) -> str:
        ...

class IPrintable(Interface):
    @abstractmethod
    def print_info(self) -> str:
        ...

# Implement the interfaces explicitly
class Document(IDrawable, IPrintable):
    def __init__(self, content: str):
        self.content = content
    
    @implements(IDrawable.draw)
    def render_document(self) -> str:
        return f"Drawing: {self.content}"
    
    @implements(IPrintable.print_info)
    def document_info(self) -> str:
        return f"Document: {self.content}"

# Use the implementation
doc = Document("Hello World")

# Access through specific interfaces
drawable = doc.as_interface(IDrawable)
printable = doc.as_interface(IPrintable)

print(drawable.draw())        # "Drawing: Hello World"
print(printable.print_info()) # "Document: Hello World"
```

## Key Concepts

### Interface Declaration

Interfaces are defined by inheriting from `Interface` and using `@abstractmethod`:

```python
class IRepository(Interface):
    @abstractmethod
    def save(self, data: dict) -> bool:
        ...
    
    @abstractmethod
    def load(self, id: str) -> dict:
        ...
```

### Explicit Implementation

Use the `@implements` decorator to explicitly map interface methods to implementation methods:

```python
class DatabaseRepository(IRepository):
    @implements(IRepository.save)
    def save_to_database(self, data: dict) -> bool:
        # Implementation here
        return True
    
    @implements(IRepository.load)  
    def load_from_database(self, id: str) -> dict:
        # Implementation here
        return {"id": id}
```

### Interface Access

Access implementations through specific interface views:

```python
repo = DatabaseRepository()

# Access through the IRepository interface
repository_interface = repo.as_interface(IRepository)
repository_interface.save({"name": "example"})
repository_interface.load("123")
```

### Concrete Classes

By default, partial implementations are allowed at class definition time. To enforce complete implementation:

```python
# This is allowed - partial implementation class definition
class PartialRepository(IRepository):
    @implements(IRepository.save)
    def save_to_database(self, data: dict) -> bool:
        return True
    # Missing IRepository.load implementation - class definition succeeds

# However, you still can't instantiate classes with unimplemented abstract methods
# partial = PartialRepository()  # Raises TypeError at instantiation

# This will raise TypeError at class definition time
class ConcreteRepository(IRepository, concrete=True):
    @implements(IRepository.save)
    def save_to_database(self, data: dict) -> bool:
        return True
    # Missing IRepository.load implementation - TypeError at class definition!
```

## Advanced Usage

### Multiple Interface Implementation

```python
class IValidator(Interface):
    @abstractmethod
    def validate(self, data: str) -> bool:
        ...

class IFormatter(Interface):
    @abstractmethod
    def format(self, data: str) -> str:
        ...

class DataProcessor(IValidator, IFormatter):
    @implements(IValidator.validate)
    def check_data(self, data: str) -> bool:
        return len(data) > 0
    
    @implements(IFormatter.format)
    def format_data(self, data: str) -> str:
        return data.upper()
```

### Interface Inheritance

```python
class IBasic(Interface):
    @abstractmethod
    def basic_method(self) -> str:
        ...

class IExtended(IBasic):
    @abstractmethod
    def extended_method(self) -> int:
        ...

class Implementation(IExtended):
    @implements(IBasic.basic_method)
    def basic_impl(self) -> str:
        return "basic"
    
    @implements(IExtended.extended_method)
    def extended_impl(self) -> int:
        return 42
```

### Diamond Inheritance Patterns

In diamond inheritance, each interface path requires explicit implementation:

```python
class IBase(Interface):
    @abstractmethod
    def base_method(self) -> str:
        ...

class ILeft(IBase):
    @abstractmethod
    def left_method(self) -> int:
        ...

class IRight(IBase):
    @abstractmethod
    def right_method(self) -> bool:
        ...

class Diamond(ILeft, IRight):
    # Can only implement base_method once for IBase
    @implements(IBase.base_method)
    def base_impl(self) -> str:
        return "base"
    
    @implements(ILeft.left_method)
    def left_impl(self) -> int:
        return 42
    
    @implements(IRight.right_method)
    def right_impl(self) -> bool:
        return True

# Access base_method only through IBase interface
diamond = Diamond()
base_interface = diamond.as_interface(IBase)
print(base_interface.base_method())  # "base"
```

## Error Handling

The library provides clear error messages for common mistakes:

### Missing Implementation

By default, partial implementations are allowed:

```python
class Incomplete(IDrawable):
    pass  # Missing @implements for IDrawable.draw
    # This is allowed by default - no error raised
```

To enforce complete implementation, use `concrete=True`:

```python
class Concrete(IDrawable, concrete=True):
    pass  # Missing @implements for IDrawable.draw
    # Raises: TypeError at class definition time
```

### Invalid Implementation Target

```python
class Invalid(IDrawable):
    @implements(IPrintable.print_info)  # Wrong interface method
    def some_method(self) -> str:
        return "invalid"
    # Raises: TypeError - method not in base interfaces
```

### Accessing Unimplemented Interface

```python
class Partial(IDrawable):
    @implements(IDrawable.draw)
    def draw_impl(self) -> str:
        return "drawn"

partial = Partial()
# This will raise TypeError:
printable = partial.as_interface(IPrintable)
```

## Type Safety

The library provides full typing support:

```python
from typing import Protocol

def use_drawable(drawable: IDrawable) -> str:
    return drawable.draw()

def process_document(doc: Document) -> tuple[str, str]:
    drawable = doc.as_interface(IDrawable)  # Type: IDrawable
    printable = doc.as_interface(IPrintable)  # Type: IPrintable
    
    return drawable.draw(), printable.print_info()
```

## Comparison with ABC

| Feature | ABC | Explicit Implementation |
|---------|-----|------------------------|
| Method Names | Must match interface | Can be different |
| Interface Access | Direct method calls | Through `.as_interface()` |
| Multiple Interfaces | Name conflicts possible | Clean separation |
| Implementation Clarity | Implicit | Explicit with `@implements` |
| Error Detection | Runtime | Compile-time |

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Changelog

### 0.1.0
- Initial release
- Basic interface and implementation functionality
- Support for multiple interface implementation
- Diamond inheritance pattern support
- Full typing support
- Comprehensive test coverage
