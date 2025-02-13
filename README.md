class Persona:
    def __init__(self, nombre, correo):
        self.nombre = nombre
        self.correo = correo

    def __str__(self):
        return f"{self.nombre} - {self.correo}"

class Usuario(Persona):
    def __init__(self, nombre, correo):
        super().__init__(nombre, correo)
        self.reservas = []

    def hacer_reserva(self, reserva):
        self.reservas.append(reserva)

    def cancelar_reserva(self, reserva):
        if reserva in self.reservas:
            self.reservas.remove(reserva)
            reserva.funcion.sala.asientos_ocupados.difference_update(reserva.asientos)
            print(f"Reserva cancelada para '{reserva.funcion.pelicula.titulo}'.")

class Empleado(Persona):
    def __init__(self, nombre, correo, rol):
        super().__init__(nombre, correo)
        self.rol = rol

    def agregar_funcion(self, pelicula, sala, horario):
        return Funcion(pelicula, sala, horario)

    def agregar_pelicula(self, titulo, duracion, clasificacion, genero):
        return Pelicula(titulo, duracion, clasificacion, genero)

    def agregar_promocion(self, descripcion, descuento, condiciones):
        return Promocion(descripcion, descuento, condiciones)

    def modificar_promocion(self, promocion, nuevo_descuento, nuevas_condiciones):
        promocion.descuento = nuevo_descuento
        promocion.condiciones = nuevas_condiciones
        print(f"Promoción modificada: {nuevo_descuento}% de descuento. {nuevas_condiciones}.")

class Espacio:
    def __init__(self, nombre):
        self.nombre = nombre

class Sala(Espacio):
    def __init__(self, nombre, tipo, capacidad):
        super().__init__(nombre)
        self.tipo = tipo
        self.capacidad = capacidad
        self.asientos_ocupados = set()

    def verificar_disponibilidad(self, cantidad):
        return len(self.asientos_ocupados) + cantidad <= self.capacidad

    def reservar_asientos(self, asientos):
        if self.verificar_disponibilidad(len(asientos)):
            self.asientos_ocupados.update(asientos)
            return True
        return False

class ZonaComida(Espacio):
    def __init__(self, nombre):
        super().__init__(nombre)
        self.productos = {} 

    def agregar_producto(self, nombre, precio, stock):
        
        if nombre in self.productos:
            self.productos[nombre][1] += stock  # Aumentar el stock si el producto ya existe
        else:
            self.productos[nombre] = [precio, stock]  # Agregar nuevo producto
        print(f"Producto '{nombre}' agregado a la zona de comida con {stock} unidades.")

    def vender_producto(self, nombre, cantidad):
        """Vende una cantidad específica de un producto."""
        if nombre in self.productos:
            precio, stock = self.productos[nombre]
            if stock >= cantidad:
                self.productos[nombre][1] -= cantidad  # Actualizar el stock
                print(f"Se han vendido {cantidad} unidades de '{nombre}' por ${precio * cantidad}.")
            else:
                print(f"No hay suficiente stock de '{nombre}' para vender {cantidad}.")
        else:
            print(f"Producto '{nombre}' no disponible en la zona de comida.")

    def mostrar_productos(self):
        """Muestra todos los productos disponibles en la zona de comida."""
        if self.productos:
            print("Productos disponibles en la zona de comida:")
            for nombre, (precio, stock) in self.productos.items():
                print(f"{nombre} - ${precio} (Stock: {stock})")
        else:
            print("No hay productos en la zona de comida.")

class Pelicula:
    def __init__(self, titulo, duracion, clasificacion, genero):
        self.titulo = titulo
        self.duracion = duracion
        self.clasificacion = clasificacion
        self.genero = genero

class Promocion:
    def __init__(self, descripcion, descuento, condiciones):
        self.descripcion = descripcion
        self.descuento = descuento
        self.condiciones = condiciones

    def __str__(self):
        return f"{self.descuento}% de descuento. {self.condiciones}"

class Funcion:
    def __init__(self, pelicula, sala, horario):
        self.pelicula = pelicula
        self.sala = sala
        self.horario = horario

class Reserva:
    def __init__(self, usuario, funcion, asientos, promocion=None):
        self.usuario = usuario
        self.funcion = funcion
        self.asientos = asientos
        self.promocion = promocion
        if self.funcion.sala.reservar_asientos(asientos):
            usuario.hacer_reserva(self)
            print(f"Reserva realizada para '{self.funcion.pelicula.titulo}' en la sala {self.funcion.sala.nombre}.")
        else:
            raise ValueError("No hay suficientes asientos disponibles.")

#Ejemplo de uso
if __name__ == "__main__":
    usuario1 = Usuario("Mia Canales", "mia.cca@email.com")
    usuario2 = Usuario("Daniel Espinoza", "dan.esp@email.com")
    print(f"La persona {usuario1.nombre} ha sido registrada con el correo {usuario1.correo}")
    print(f"La persona {usuario2.nombre} ha sido registrada con el correo {usuario2.correo}")

    empleado1 = Empleado("Oscar Campos", "oscar.camp@email.com", "Administrador")
    pelicula1 = empleado1.agregar_pelicula("Hannibal Lecter", 118, "MA-19", "Suspenso")
    sala1 = Sala("Sala 5", "IMAX", 101)
    funcion1 = empleado1.agregar_funcion(pelicula1, sala1, "17:30")
    promocion1 = empleado1.agregar_promocion("Descuento limitado", 14, "Válido de miercoles a viernes")
    print(f"Promoción: {promocion1}")

    zona_comida = ZonaComida("Zona de Snacks")

    zona_comida.agregar_producto("Palomitas", 9, 100)
    zona_comida.agregar_producto("Refresco", 3, 53)

    zona_comida.vender_producto("Palomitas", 8)  # Vende 10 unidades de Palomitas
    zona_comida.vender_producto("Refresco", 3)   # Vende 5 unidades de Refresco

    zona_comida.mostrar_productos()

    reserva1 = Reserva(usuario1, funcion1, [1, 2, 3], promocion1)
    usuario1.cancelar_reserva(reserva1)

    empleado1.modificar_promocion(promocion1, 30, "Válido todos los días antes de las 5 PM")

    print("\nPersonas registradas")
    for persona in [usuario1, usuario2]:
        print(f"-{persona}")
