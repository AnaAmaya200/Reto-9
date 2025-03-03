# Programación Orientada a Objetos - UNAL

# Ana María Amaya Gómez

## Reto 9: Decoradores

Agregue el decorador @property al paquete Shape, para que se pueda acceder a todos los datos protegidos de esta manera.
Agregue el decorador @classmethod a Shape, para poder cambiar, definir y cambiar el tipo de forma de cada clase.
Agregue un decorador personalizado en Shape para mostrar el tiempo de cálculo de al menos una operación. por ejemplo: área_calculada.
```python
import math
import time

class Point:
    def __init__(self, x=0.0, y=0.0):
        self.x = float(x)
        self.y = float(y)

    def compute_distance(self, point:"Point"):
        return ((self.x - point.x)**2 + (self.y - point.y)**2)**0.5


class Line:
    def __init__(self, start: Point, end: Point):
        self.start = start
        self.end = end

    def compute_length(self):
        self.length = round(Point.compute_distance(self.start, self.end), 2)
        return self.length


class Shape:
    def __init__(self):
        self._is_regular = None
        self._vertices = []
        self._edges = []
        self._inner_angles = []

    @property
    def is_regular(self):
        return self._is_regular

    @property
    def vertices(self):
        return self._vertices

    @property
    def edges(self):
        return self._edges

    @property
    def inner_angles(self):
        return self._inner_angles

    @classmethod
    def set_type(cls, shape_type):
        cls.shape_type = shape_type

    @staticmethod
    def calculate_time(func):
        def wrapper(*args, **kwargs):
            start_time = time.time()
            result = func(*args, **kwargs)
            end_time = time.time()
            print(f"Tiempo de cálculo para {func.__name__}: {end_time - start_time:.4f} segundos")
            return result
        return wrapper

    @calculate_time
    def compute_area(self):
        pass

    @calculate_time
    def compute_perimeter(self):
        pass

    @calculate_time
    def compute_inner_angles(self):
        pass


class Rectangle(Shape):
    def __init__(self, Bottom_left_corner, Upper_right_corner):
        super().__init__()
        self._is_regular = False
        Bottom_right_corner = Point(Upper_right_corner.x, Bottom_left_corner.y)
        Upper_left_corner = Point(Bottom_left_corner.x, Upper_right_corner.y)
        self._vertices.append(Bottom_left_corner)
        self._vertices.append(Upper_left_corner)
        self._vertices.append(Upper_right_corner)
        self._vertices.append(Bottom_right_corner)
        self._edges.append(Line(Bottom_left_corner, Upper_left_corner))
        self._edges.append(Line(Upper_left_corner, Upper_right_corner))
        self._edges.append(Line(Upper_right_corner, Bottom_right_corner))
        self._edges.append(Line(Bottom_right_corner, Bottom_left_corner))

    @Shape.calculate_time
    def compute_area(self):
        return Line.compute_length(self._edges[0]) * Line.compute_length(self._edges[1])

    @Shape.calculate_time
    def compute_perimeter(self):
        return (2 * Line.compute_length(self._edges[0])) + (2 * Line.compute_length(self._edges[1]))

    @Shape.calculate_time
    def compute_inner_angles(self):
        self._inner_angles = [90] * 4
        return self._inner_angles


class Square(Rectangle):
    def __init__(self, Bottom_left_corner, Upper_right_corner):
        super().__init__(Bottom_left_corner, Upper_right_corner)
        self._is_regular = True

    def compute_area(self):
        return super().compute_area()

    def compute_inner_angles(self):
        return super().compute_inner_angles()


class Triangle(Shape):
    def __init__(self, Bottom_left_corner, Bottom_right_corner, Upper_corner):
        super().__init__()
        self._is_regular = False
        self._vertices.append(Bottom_left_corner)
        self._vertices.append(Bottom_right_corner)
        self._vertices.append(Upper_corner)
        self._edges.append(Line(Bottom_left_corner, Bottom_right_corner))
        self._edges.append(Line(Bottom_right_corner, Upper_corner))
        self._edges.append(Line(Upper_corner, Bottom_left_corner))

    def compute_perimeter(self):
        return Line.compute_length(self._edges[0]) + Line.compute_length(self._edges[1]) + Line.compute_length(self._edges[2])


class Isosceles(Triangle):
    def __init__(self, Bottom_left_corner, Bottom_right_corner, Upper_corner):
        super().__init__(Bottom_left_corner, Bottom_right_corner, Upper_corner)

    @Shape.calculate_time
    def compute_area(self):
        h = ((Line.compute_length(self._edges[1]))**2 + (Line.compute_length(self._edges[0])**2))**1/2
        return (Line.compute_length(self._edges[0]) * h) / 2

    def compute_perimeter(self):
        return super().compute_perimeter()

    def compute_inner_angles(self):
        A = math.degrees(math.acos(((Line.compute_length(self._edges[1]))**2 + (Line.compute_length(self._edges[2]))**2 - (Line.compute_length(self._edges[0]))**2) / (2 * (Line.compute_length(self._edges[1])) * (Line.compute_length(self._edges[2])))))
        B = math.degrees(math.acos(((Line.compute_length(self._edges[0]))**2 + (Line.compute_length(self._edges[2]))**2 - (Line.compute_length(self._edges[1]))**2) / (2 * (Line.compute_length(self._edges[0])) * (Line.compute_length(self._edges[2])))))
        C = 180 - A - B
        self._inner_angles.append(A)
        self._inner_angles.append(B)
        self._inner_angles.append(C)
        return self._inner_angles


class Equilateral(Triangle):
    def __init__(self, Bottom_left_corner, Bottom_right_corner, Upper_corner):
        super().__init__(Bottom_left_corner, Bottom_right_corner, Upper_corner)
        self._is_regular = True

    @Shape.calculate_time
    def compute_area(self):
        return (((Line.compute_length(self._edges[0]))**2) * math.sqrt(3)) / 4

    def compute_perimeter(self):
        return super().compute_perimeter()

    def compute_inner_angles(self):
        self._inner_angles = [60] * 3
        return self._inner_angles


class Scalene(Triangle):
    def __init__(self, Bottom_left_corner, Bottom_right_corner, Upper_corner):
        super().__init__(Bottom_left_corner, Bottom_right_corner, Upper_corner)

    def compute_perimeter(self):
        return super().compute_perimeter()

    @Shape.calculate_time
    def compute_area(self):
        s = self.compute_perimeter() / 2
        return math.sqrt(s * (s - Line.compute_length(self._edges[0])) * (s - Line.compute_length(self._edges[1])) * (s - Line.compute_length(self._edges[2])))

    def compute_inner_angles(self):
        A = math.degrees(math.acos(((Line.compute_length(self._edges[1]))**2 + (Line.compute_length(self._edges[2]))**2 - (Line.compute_length(self._edges[0]))**2) / (2 * (Line.compute_length(self._edges[1])) * (Line.compute_length(self._edges[2])))))
        B = math.degrees(math.acos(((Line.compute_length(self._edges[0]))**2 + (Line.compute_length(self._edges[2]))**2 - (Line.compute_length(self._edges[1]))**2) / (2 * (Line.compute_length(self._edges[0])) * (Line.compute_length(self._edges[2])))))
        C = 180 - A - B
        self._inner_angles.append(A)
        self._inner_angles.append(B)
        self._inner_angles.append(C)
        return self._inner_angles


class Trirectangle(Triangle):
    def __init__(self, Bottom_left_corner, Bottom_right_corner, Upper_corner):
        super().__init__(Bottom_left_corner, Bottom_right_corner, Upper_corner)

    def compute_perimeter(self):
        return super().compute_perimeter()

    @Shape.calculate_time
    def compute_area(self):
        if self._vertices[1].y == self._vertices[2].y:
            return (Line.compute_length(self._edges[0]) * Line.compute_length(self._edges[1])) / 2
        else:
            return (Line.compute_length(self._edges[0]) * Line.compute_length(self._edges[2])) / 2

    def compute_inner_angles(self):
        a = 90
        if self._vertices[1].y == self._vertices[2].y:
            b = math.atan(Line.compute_length(self._edges[1]) / Line.compute_length(self._edges[0]))
        else:
            b = math.atan(Line.compute_length(self._edges[2]) / Line.compute_length(self._edges[0]))
        c = a - b
        self._inner_angles.append(a)
        self._inner_angles.append(b)
        self._inner_angles.append(c)
        return self._inner_angles

def main():
    # Crear puntos
    p1 = Point(0, 0)
    p2 = Point(4, 0)
    p3 = Point(4, 3)
    p4 = Point(0, 3)
    p5 = Point(2, 2)

    # Crear un rectángulo
    rectangle = Rectangle(p1, p3)
    print(f"Área del rectángulo: {rectangle.compute_area()} unidades cuadradas")
    print(f"Perímetro del rectángulo: {rectangle.compute_perimeter()} unidades")
    print(f"Ángulos interiores del rectángulo: {rectangle.compute_inner_angles()} grados")

    # Acceder a las propiedades del rectángulo
    print(f"\n¿Es el rectángulo una forma regular? {rectangle.is_regular}")
    print(f"Vértices del rectángulo: {[(v.x, v.y) for v in rectangle.vertices]}")
    print(f"Longitud de los lados del rectángulo: {[e.compute_length() for e in rectangle.edges]}")

    # Cambiar el tipo de forma utilizando el método de clase
    Shape.set_type("Cuadrado")
    print(f"\nTipo de forma después de set_type: {Shape.shape_type}")

    # Crear un cuadrado
    square = Square(p1, p2)
    print(f"\nÁrea del cuadrado: {square.compute_area()} unidades cuadradas")
    print(f"Perímetro del cuadrado: {square.compute_perimeter()} unidades")
    print(f"Ángulos interiores del cuadrado: {square.compute_inner_angles()} grados")

    # Crear un triángulo equilátero
    equilateral_triangle = Equilateral(p1, p2, p5)
    print(f"\nÁrea del triángulo equilátero: {equilateral_triangle.compute_area()} unidades cuadradas")
    print(f"Perímetro del triángulo equilátero: {equilateral_triangle.compute_perimeter()} unidades")
    print(f"Ángulos interiores del triángulo equilátero: {equilateral_triangle.compute_inner_angles()} grados")

    # Crear un triángulo isósceles
    isosceles_triangle = Isosceles(p1, p2, p5)
    print(f"\nÁrea del triángulo isósceles: {isosceles_triangle.compute_area()} unidades cuadradas")
    print(f"Perímetro del triángulo isósceles: {isosceles_triangle.compute_perimeter()} unidades")
    print(f"Ángulos interiores del triángulo isósceles: {isosceles_triangle.compute_inner_angles()} grados")

    # Crear un triángulo escaleno
    scalene_triangle = Scalene(p1, p2, p3)
    print(f"\nÁrea del triángulo escaleno: {scalene_triangle.compute_area()} unidades cuadradas")
    print(f"Perímetro del triángulo escaleno: {scalene_triangle.compute_perimeter()} unidades")
    print(f"Ángulos interiores del triángulo escaleno: {scalene_triangle.compute_inner_angles()} grados")


if __name__ == "__main__":
    main()
```
RESULTADOS

```python
Tiempo de cálculo para compute_area: 0.0000 segundos
Área del rectángulo: 12.0 unidades cuadradas
Tiempo de cálculo para compute_perimeter: 0.0000 segundos
Perímetro del rectángulo: 14.0 unidades
Tiempo de cálculo para compute_inner_angles: 0.0000 segundos
Ángulos interiores del rectángulo: [90, 90, 90, 90] grados

¿Es el rectángulo una forma regular? False
Vértices del rectángulo: [(0.0, 0.0), (0.0, 3.0), (4.0, 3.0), (4.0, 0.0)]
Longitud de los lados del rectángulo: [3.0, 4.0, 3.0, 4.0]

Tipo de forma después de set_type: Cuadrado
Tiempo de cálculo para compute_area: 0.0000 segundos

Área del cuadrado: 0.0 unidades cuadradas
Tiempo de cálculo para compute_perimeter: 0.0000 segundos
Perímetro del cuadrado: 8.0 unidades
Tiempo de cálculo para compute_inner_angles: 0.0000 segundos
Ángulos interiores del cuadrado: [90, 90, 90, 90] grados
Tiempo de cálculo para compute_area: 0.0000 segundos

Área del triángulo equilátero: 6.928203230275509 unidades cuadradas
Perímetro del triángulo equilátero: 9.66 unidades
Ángulos interiores del triángulo equilátero: [60, 60, 60] grados
Tiempo de cálculo para compute_area: 0.0000 segundos

Área del triángulo isósceles: 24.0089 unidades cuadradas
Perímetro del triángulo isósceles: 9.66 unidades
Ángulos interiores del triángulo isósceles: [89.93632926586426, 45.03183536706787, 45.03183536706787] grados
Tiempo de cálculo para compute_area: 0.0000 segundos

Área del triángulo escaleno: 6.0 unidades cuadradas
Perímetro del triángulo escaleno: 12.0 unidades
Ángulos interiores del triángulo escaleno: [53.13010235415599, 36.86989764584401, 90.0] grados
```
