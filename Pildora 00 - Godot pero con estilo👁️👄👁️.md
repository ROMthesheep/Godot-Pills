
#### Prologo:

Antes de nada, ¿De qué va esto?
Esto va a ser una serie de pills sobre Godot y sus cosillas, pero por una vez en castellano que falta nos hace contenidos explicados y localizados a nuestro idioma.
En estas píldoras intentare condensar un tema concreto en una lectura rápida, de no mas de 10 minutos y que siempre vengan acompañadas de material que podáis descargar y usar.

Y claro, yo tenia un dilema, ¿Cuál debería ser la primera pill? Pues como no, una guía de como programar para Godot pero con estilo mi ciela.

### Disclaimer e introducción:

Antes de comenzar un disclaimer, esto no dejan de ser **RECOMENDACIONES**, habrá casos donde no se puedan seguir estas reglas o donde causen más problemas que otra cosa. También reforzar que estas directrices buscan crear una homogeneidad en todos los scripts de todos los proyectos, si tu proyecto es solo para tus ojos realmente no tiene ninguna utilidad, **PERO** siempre esta bien acostumbrarse y tener estas indicaciones en mente porque nunca se sabe y a nadie le gusta refactorizar ficheros enteros. Al final del día es mucho más importante mantener una consistencia a lo largo de un proyecto que seguir guías que a ti no te parecen lógicas.

Ahora si vamos a entrar en materia, Godot usa el lenguaje conocido como GDScript, un lenguaje de alto nivel con tipado dinámico basado en Python y esta creado con el único objetivo de ser usado con el motor Godot. Existe una [guía de estilo oficial,](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html) de la que he sacado los contenidos para esta píldora, y esta estrechamente relacionada con la guía de estilo [PEP 8](https://peps.python.org/pep-0008/) de Python, esta guía lleva actualizándose desde Python 2.2.

Importante mencionar que **este post es un resumen de la guía** con lo que a mi me ha parecido más relevante, si os interesa lo que leéis aquí y queréis profundizar id al articulo original.
### Convención de nomenclatura:

Como con cualquier herramienta usaremos funciones y variables incluidas en la misma, es importante por ello adoptar la convención de nombres que usa para evitar inconsistencia.
GDScript usa snake_case, CONSTANT_CASE y PascalCase:

- Snake_case:
	- Nombres de ficheros
	- Nombres de funciones
- PascalCase:
	- Nombres de clases
	- Nombres de nodos
	- Carga de clases a una constante o variable
	- Nombres de enumeradores
- CONSTANT_CASE:
	- Nombre de constantes
	- Nombres de miembros de enumeradores

Aquí tenéis un fichero de ejemplo:
```GDScript
# Este fichero se llama player.gd
# (opcional) Nombre de clase e icono, este campo hara que nuestra clase sea visible en el scope global y nos dara itelisense.
# Tambien cambiara la propiedad "name" que tienen todos los nodos
class_name Player, "res://Assets/optional/icon.svg"
extends Node2D
# Esto es el docstring, te ayudara en proyectos colaborativos
## Rol y funciones de la clase.
##
## Descripcion de las features del script
## Cualquier otra informacion

# Las señales siempre van con un verbo en pasado
signal player_died 
signal player_spawned(position)

enum StartingClass{
	BARBARIAN,
	WIZARD,
	MALAGUEÑO,
	NECROMANCER,
	PUGILIST,
	KEBAMANCER
}

const MAX_HEALTH: int = 120
const MAX_SPEED: float = 69.9
const DeadSprite = preload("res://Assets/Sprites/DeadPlayer.tres")

@export var player_name: String = "Manueh"
@export var level: int = 0

var health: int = MAX_HEALTH:
	set(new_health):
		health = new_health

# Usamos un guion bajo al principio de las variables cuando queremos dejar implicito que son variables privadas
var _attack: int = 15
var _is_dead: bool = false

@onready var weapon: Weapon = $Weapon
@onready var sprite: Sprite2D = $Sprite

# Entre funciones es costumbre dejar dos lineas en blanco y una entre bloques funcionales
# En caso de tener una sobrecarga del ciclo de vida basico deberia ir en la primera seccion de nuestro script
# Estos metodos son por orden _init(), _enter_tree() y _ready()
func _init():
	pass


func _ready():
	pass
	# los comentarios de texto van con un espacio en la primera linea
	# Mientras que los de codigo van sin espacio, como este de abajo
	#print("Me han censurado")

# Funciones propias del motor
func _unhandled_input(event):
	pass


# Metodos publicos
func set_new_stats(stats: Dictionary):
	print("haz una movida")
	print("haz una movida que tiene que ver con la de arriba")
	
	print("haz otra movida que no tiene que ver con la anterior")


# Metodos privados, igual que con las variables privadas comienzan con un guion bajo
func _hit(damage: int):
	health -= 1
	if health == 0:
		_is_dead = true
		player_died.emit()


# Clases internas
class Armadura:
	var protection: int = 10
```

### Orden de código:

Si has prestado atención al fichero anterior ya se ha dejado caer que existe un orden para cada elemento, la propia documentación nos lo indica, este seria el orden:

1. @tool: Anotación para indicar que nuestro script es una [herramienta de desarrollo](https://docs.godotengine.org/en/stable/tutorials/plugins/running_code_in_the_editor.html)
2. class_name
3. extends: Clase Padre
4. docstring: Documentacion de la clase
5. señales
6. enums
7. constantes
8. exports
9. Variables publicas
10. Variables Privadas
11. Onreadys
12. Ciclo de vida del Nodo (\_init(), \_enter_tree() y \_ready())
15. Métodos propios del nodo: Por ejemplo \_unhandled_input() o \_physics_process()
16. Métodos públicos
17. Métodos privados
18. subclases

Este orden se rige por cuatro normas:

1. Propiedades y señales van primero
2. Publico antes que privado
3. Las retrollamadas van antes que la interfaz de clase
4. Los métodos \_init() y \_ready van antes de los métodos que modifican el nodo en tiempo de ejecución


### Tipado Estático y dinámico:

Desde Godot 3.1 GDScript admite tipado estático, esto no solo ayuda a la lectura del código sino que a demás nos dará un mejor rendimiento y ayuda a evitar posibles bugs y crashes al parsear tipos a nuestra espalda.
Yo personalmente no veo que el tipado dinámico ofrezca un beneficio lo suficientemente grande como para usarlo por defecto, de hecho [hay una opción para que el editor te notifique variables dinámicas como error](https://simondalvai.org/blog/godot-static-typing/#project-settings-to-have-stricter-error-checks).
Pero hay recordar que hay casos donde el tipado dinámico puede ser útil y no debemos cerrarnos en banda a él, aqui cada uno debe elegir su bando.

Tenemos entonces dos tipos de tipado y tres formas de aplicarlo, el primero, el tipado dinámico tendría este aspecto
```GDScript
var hello_world = "Hello SCeV"
```
Y para aplicar un tipado de forma estática tenemos dos acercamientos, ambos validos y equivalentes

```GDScript
var hello_world: String = "Hello SCeV"
var hello_world := "Hello SCeV"
```
Es importante con el ultimo método recordar no usarlo como resultado del retorno de una funcion que no sea obvia para evitar una mala lectura de código, por ejemplo

```GDScript
# Esto estaria bien, se entiende el tipo que se esta infiriendo
var hello_world := salute.get_data_to_string()

# Esto esta mal, se pierde informacion
var hello_world := salute.get_data()

# Para remediar el caso superior siempre podemos usar
var hello_world: String = salute.get_data()
```
## Conclusión:

¿Qué, os ha gustado? ¿no esta mal para la primera no? Algo habréis aprendido seguro.
Estas pills serán recopiladas en un repo que tendréis siempre al final de cada pill y también asi podéis hacerle PR y ampliarlas/corregir moviditas que no estén del todo finas.
También en las issues podréis pedir ciertos temas y si me veo lo suficiente valiente y con conocimientos necesarios lo abordare en siguientes entregas.
Esta pill lo malo que tiene es que no hay mucho donde opinar, es el "dogma" que han impuesto los desarrolladores aunque siempre hay espacio para tomarse las reglas como quieras.
Os voy a dejar adjuntado una platilla con los marcadores del orden, para instalarla es tan facil como colocarla dentro de una carpeta que se llame "Node" en la carpeta de las plantillas.
En fin, que me alargo, eso ha sido todo, os dejo [aquí](https://github.com/ROMthesheep/Godot-Pills) el repo, dadle una estrellita si os ha gustado.
