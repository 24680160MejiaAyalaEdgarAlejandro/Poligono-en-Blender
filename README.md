Guía: Creación de Polígonos con Python en Blender
Esta guía explica cómo generar un polígono regular desde cero utilizando el motor de scripting de Blender.

1. Conceptos Matemáticos Base
Para crear un polígono en un espacio 3D, necesitamos calcular la posición de sus vértices. Usamos coordenadas polares que luego convertimos a cartesianas (x,y,z):

Ángulo: Dividimos una circunferencia completa (2π radianes) entre el número de lados.
Posición X: radio⋅cos(θ)
Posición Y: radio⋅sin(θ)

2. Explicación del Código Paso a Paso
A. Preparación de la Escena
Antes de crear algo nuevo, es buena práctica limpiar los objetos existentes para no encimarlos cada vez que ejecutes el script.

bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()

B. Creación del Contenedor (Mesh y Objeto)
En Blender, un objeto visible se compone de dos partes: los datos (la malla o mesh) y el contenedor (el objeto que vive en la escena).


malla = bpy.data.meshes.new(nombre)         # Crea el "alma" (datos de vértices)
objeto = bpy.data.objects.new(nombre, malla) # Crea el "cuerpo" (el objeto 3D)
bpy.context.collection.objects.link(objeto)  # Lo hace visible en la escena

C. Generación de Geometría
Aquí es donde ocurre la magia. Definimos los puntos (vértices) y cómo se conectan entre ellos (aristas).


for i in range(lados):
    # Calculamos el ángulo para cada vértice
    angulo = 2 * math.pi * i / lados
    x = radio * math.cos(angulo)
    y = radio * math.sin(angulo)
    vertices.append((x, y, 0)) # Añadimos el punto al listado
    
    # Conectamos el vértice actual (i) con el siguiente (i+1)
    # El % lados asegura que el último vértice se conecte con el primero
    aristas.append((i, (i + 1) % lados))
D. Construcción Final
Finalmente, le decimos a Blender que tome esas listas de números y las convierta en geometría real.

Python
malla.from_pydata(vertices, aristas, []) # (Vértices, Aristas, Caras)
malla.update() # Refresca la malla para mostrar los cambios

3. El Script Completo
Puedes copiar y pegar este código directamente en el Text Editor de Blender:


import bpy
import math
def crear_poligono_2d(nombre, lados, radio):
    """
    Crea un polígono regular en el plano XY.
    """
    # 1. Definir la malla y el objeto
    malla = bpy.data.meshes.new(nombre)
    objeto = bpy.data.objects.new(nombre, malla)
    
    # 2. Vincular a la colección activa
    bpy.context.collection.objects.link(objeto)
    
    vertices = []
    aristas = []
    
    # 3. Calcular geometría
    for i in range(lados):
        angulo = 2 * math.pi * i / lados
        x = radio * math.cos(angulo)
        y = radio * math.sin(angulo)
        vertices.append((x, y, 0)) 
        
        # Crear conexión entre puntos
        aristas.append((i, (i + 1) % lados))
        
    # 4. Cargar datos en la malla
    malla.from_pydata(vertices, aristas, [])
    malla.update()

# --- Ejecución ---
if __name__ == "__main__":
    # Limpiar escena
    if bpy.context.object:
        bpy.ops.object.select_all(action='SELECT')
        bpy.ops.object.delete()

    # Crear un octágono de radio 4
    crear_poligono_2d("MiPoligono", lados=8, radio=4)
¿Cómo usar este repositorio?
Abre Blender.
Ve a la pestaña de Scripting.
Haz clic en New y pega el código anterior.
Presiona el botón Run Script.

-------------------AÑADO IMAGEN DEL POLIGONO--------------------------
<img width="515" height="287" alt="image" src="https://github.com/user-attachments/assets/1b713869-d60f-4d94-bbf7-bfb63d418f24" />



Guía: El Patrón "Flor de la Vida" con Ciclos while
En este tutorial aprenderemos a usar la automatización de Blender para repetir una tarea geométrica. El objetivo es crear un patrón donde seis círculos rodean a uno central, intersectándose en sus centros.

Lógica del AlgoritmoPara que los círculos se toquen perfectamente, el centro de cada círculo exterior debe estar sobre el borde del círculo central.
 .Paso Angular: Como una circunferencia tiene 360°, y queremos 6 círculos alrededor, cada uno debe estar separado por $360 / 6 = 60^{\circ}$.
 .Posición: Usamos de nuevo la conversión de grados a radianes para que Python entienda la ubicación en el plano $XY$.

2. Explicación del Código Paso a Paso
   A. El Círculo Base
   Primero colocamos el círculo que servirá como núcleo del patrón.

   #Crea un círculo en el origen (0,0,0) con el radio definido
bpy.ops.mesh.primitive_circle_add(radius=radio, location=(0, 0, 0), vertices=64)

B. El Bucle de Control (while)
En lugar de escribir 6 veces el código para añadir círculos, usamos un ciclo while. Este se ejecuta mientras la condición sea verdadera.Pythonwhile angulo_actual < 360:
    # 1. Calculamos la posición X e Y
    # Importante: math.cos usa RADIANES, por eso convertimos los GRADOS
    x = radio * math.cos(math.radians(angulo_actual))
    y = radio * math.sin(math.radians(angulo_actual))
    
    # 2. Ejecutamos el comando de creación en la nueva posición
    bpy.ops.mesh.primitive_circle_add(radius=radio, location=(x, y, 0), vertices=64)
    
    # 3. ¡Crucial! Aumentamos el ángulo para que el ciclo avance
    # Si no hiciéramos esto, el programa se trabaría en un bucle infinito
    angulo_actual += paso_angular

3. Código Completo y Comentado
 Este script es ideal para entender cómo bpy.ops (operaciones de Blender) puede ser controlado por lógica de Python pura.

import bpy
import math

# --- CONFIGURACIÓN INICIAL ---
# Limpia la escena para evitar acumular objetos de ejecuciones anteriores
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()

# Parámetros ajustables
radio = 3
angulo_actual = 0
paso_angular = 60 # Define la separación entre pétalos

# 1. Crear el círculo central
bpy.ops.mesh.primitive_circle_add(radius=radio, location=(0, 0, 0), vertices=64)

# 2. Iniciar el ciclo de creación de los pétalos exteriores
while angulo_actual < 360:
    # Cálculo de coordenadas cartesianas (X, Y)
    x = radio * math.cos(math.radians(angulo_actual))
    y = radio * math.sin(math.radians(angulo_actual))
    
    # Añadir el círculo en la posición calculada
    bpy.ops.mesh.primitive_circle_add(radius=radio, location=(x, y, 0), vertices=64)
    
    # Incremento para la siguiente iteración
    angulo_actual += paso_angular

print("Patrón 'Flor de la Vida' generado.")

-------------------AÑADO IMAGEN DE LA FLOR DE LA VIDA --------------------------
<img width="585" height="485" alt="image" src="https://github.com/user-attachments/assets/c72c67af-9b27-4668-ad95-0659d13c8223" />
