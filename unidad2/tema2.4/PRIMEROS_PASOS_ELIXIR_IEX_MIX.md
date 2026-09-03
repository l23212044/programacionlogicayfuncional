# Investigación: primeros pasos en Elixir con IEx y Mix

**Contexto:** Tema 2.4 — Evaluación perezosa · **Fecha de consulta:** 3 de
septiembre de 2026.

## 1. Propósito

Elixir es un lenguaje funcional, concurrente y de tipado dinámico que se ejecuta
sobre la máquina virtual BEAM de Erlang. Para empezar a programar conviene
distinguir dos herramientas que se instalan con Elixir:

| Herramienta | Para qué se usa | Idea central |
|---|---|---|
| **IEx** (*Interactive Elixir*) | Probar expresiones, explorar módulos y depurar de forma interactiva. | Un REPL: escribe, evalúa y observa el resultado. |
| **Mix** | Crear, compilar, probar, formatear y administrar proyectos. | La herramienta de automatización y construcción del proyecto. |

No son alternativas: un flujo habitual es crear el proyecto con Mix y abrir IEx
con el contexto del proyecto ya compilado mediante `iex -S mix`.

## 2. Preparación y verificación

Después de instalar Erlang/OTP y Elixir, verifica que tanto el intérprete como
la herramienta de proyectos están disponibles en el `PATH`:

```bash
elixir --version
mix --version
```

En una instalación nueva también se pueden instalar los gestores auxiliares que
Mix utiliza para dependencias de Elixir y Erlang:

```bash
mix local.hex --force
mix local.rebar --force
```

`Hex` es el gestor y repositorio de paquetes de Elixir; `rebar3` se usa para
compilar ciertas dependencias escritas en Erlang. Ambos comandos solo preparan
el entorno local; no crean una aplicación.

## 3. IEx: aprender mediante experimentación

Inicia la consola con:

```bash
iex
```

El prompt `iex(n)>` indica el número de expresión evaluada. IEx muestra el
valor resultante, por lo que es útil para comprender la inmutabilidad, el
*pattern matching*, las tuberías y las colecciones antes de escribir un archivo.

```elixir
iex(1)> nombre = "Ada"
"Ada"
iex(2)> "Hola, " <> nombre
"Hola, Ada"
iex(3)> 1..10 |> Enum.filter(&(rem(&1, 2) == 0)) |> Enum.map(&(&1 * &1))
[4, 16, 36, 64, 100]
```

La variable `nombre` queda asociada al valor `"Ada"`; no se modifica el texto
original. En el último ejemplo, `|>` entrega el resultado de cada operación a
la siguiente: primero filtra los números pares y después los eleva al cuadrado.

### Ayuda, documentación y comandos útiles

IEx permite explorar la biblioteca sin salir de la terminal:

```elixir
iex> h Enum.map/2       # documentación de una función
iex> i %{curso: "PLF"} # tipo y representación de un valor
iex> v(1)               # resultado de la expresión 1
iex> r "lib/saludo.ex" # recompila un archivo cargado
iex> c "ejemplo.ex"    # compila un archivo .ex
iex> q()                # sale de IEx
```

Los comandos `h`, `i`, `v`, `r`, `c` y `q` son ayudas del entorno interactivo;
no forman parte de la sintaxis que se escribe en los módulos de la aplicación.
Para ejecutar una expresión sin abrir el prompt se puede usar:

```bash
elixir -e 'IO.puts(Enum.sum(1..10))'
# 55
```

## 4. Mix: pasar de experimentos a un proyecto reproducible

Mix organiza el código, proporciona tareas estándar y mantiene separadas las
dependencias del proyecto. `mix help` lista las tareas disponibles y
`mix help NOMBRE_DE_TAREA` explica una tarea concreta.

### Crear una aplicación mínima

```bash
mix new saludo --module Saludo
cd saludo
mix test
```

El modificador `--module Saludo` define el módulo principal generado. La tarea
`mix test` compila el proyecto cuando es necesario y ejecuta las pruebas de
ExUnit. Una estructura inicial típica es:

```text
saludo/
├── lib/saludo.ex        # módulo de la aplicación
├── test/saludo_test.exs # pruebas automatizadas
├── mix.exs              # configuración, aplicación y dependencias
└── .formatter.exs       # configuración del formateador
```

El archivo `mix.exs` es código Elixir que define un módulo de proyecto. Allí se
declaran, entre otros datos, el nombre, la versión, la versión mínima de Elixir
y las dependencias. No se deben descargar paquetes manualmente dentro de
`lib/`: se registran en `deps` y Mix se encarga de resolverlos.

### Ejecutar código del proyecto desde IEx

Desde el directorio `saludo`, abre:

```bash
iex -S mix
```

La opción `-S mix` inicia IEx después de ejecutar Mix: compila y carga los
módulos del proyecto y sus dependencias. Por ello el módulo creado ya se puede
consultar:

```elixir
iex> Saludo.hello()
:world
```

Usar únicamente `iex` fuera de ese contexto no carga automáticamente el
proyecto. Esta diferencia evita una fuente común de errores iniciales del tipo
`undefined function` o `undefined module`.

## 5. Ciclo de trabajo básico

Un ciclo pequeño y repetible para cada cambio es:

```bash
mix format
mix compile
mix test
mix format --check-formatted
```

1. Escribe los módulos en `lib/` y las pruebas en `test/`.
2. Aplica `mix format` para mantener el estilo estándar de Elixir.
3. Comprueba que el proyecto compila con `mix compile`.
4. Ejecuta las pruebas con `mix test`.
5. En integración continua, usa `mix format --check-formatted` para detectar
   archivos que aún no se han formateado.

Para ejecutar una expresión puntual con el proyecto cargado, sin entrar al
REPL, utiliza `mix run`:

```bash
mix run -e 'IO.inspect(Saludo.hello())'
```

## 6. Dependencias: ejemplo controlado

Una dependencia de Hex se declara en la función privada `deps` de `mix.exs`.
Por ejemplo, para trabajar con JSON:

```elixir
defp deps do
  [
    {:jason, "~> 1.4"}
  ]
end
```

Después se descargan y compilan las dependencias declaradas:

```bash
mix deps.get
mix deps.compile
```

El requisito de versión `~> 1.4` permite actualizaciones compatibles dentro de
la serie 1.x según las reglas de versionado de Hex. Antes de añadir una
dependencia conviene revisar su documentación, mantenimiento y licencia.

## 7. Relación con los Streams del tema

IEx permite observar la diferencia entre una colección ya materializada y un
flujo perezoso. En la primera expresión `Enum` devuelve de inmediato una lista;
en la segunda, `Stream` describe el procesamiento y `Enum.take/2` lo consume:

```elixir
iex> Enum.map(1..5, &(&1 * 2))
[2, 4, 6, 8, 10]
iex> 1..1_000_000 |> Stream.map(&(&1 * 2)) |> Enum.take(5)
[2, 4, 6, 8, 10]
```

Esta segunda forma no construye primero una lista de un millón de resultados;
solo solicita los cinco valores necesarios. El archivo `stream.exs` del tema
amplía esta idea con un *pipeline* de datos.

## 8. Errores frecuentes y cómo evitarlos

| Situación | Causa habitual | Acción recomendada |
|---|---|---|
| `mix` o `iex` no se encuentran | El directorio `bin` de Elixir no está en `PATH`. | Verificar la instalación con `elixir --version` y ajustar el gestor de versiones o el `PATH`. |
| Un módulo no existe en IEx | Se abrió `iex` en vez de `iex -S mix`, o falta compilación. | Entrar al directorio del proyecto y usar `iex -S mix`. |
| Las pruebas fallan tras editar código | El comportamiento y la expectativa ya no coinciden. | Leer el fallo, corregir el módulo o la prueba y repetir `mix test`. |
| El formateo falla en CI | Se editó código sin ejecutar el formateador. | Ejecutar `mix format` y confirmar con `mix format --check-formatted`. |
| Una dependencia no se resuelve | Falta declararla, hay una restricción incompatible o no hay acceso a Hex. | Revisar `mix.exs`, ejecutar `mix deps.get` y consultar el mensaje de Mix. |

## 9. Conclusión

IEx reduce la fricción al aprender Elixir: permite comprobar una idea en una
línea y consultar documentación en el mismo entorno. Mix convierte esas pruebas
en software mantenible al aportar una estructura, compilación, pruebas,
formateo y administración de dependencias. La combinación recomendada para
principiantes es: **experimentar en IEx → crear y probar con Mix → explorar el
proyecto con `iex -S mix`**.

## Fuentes consultadas

* [Elixir — IEx (documentación oficial)](https://hexdocs.pm/iex/IEx.html).
* [Elixir — Mix (documentación oficial)](https://hexdocs.pm/mix/Mix.html).
* [Elixir — tarea `mix new` (documentación oficial)](https://hexdocs.pm/mix/Mix.Tasks.New.html).
* [Elixir — Introduction to Mix (guía oficial)](https://elixir-lang.org/getting-started/mix-otp/introduction-to-mix.html).
* [Guía de instalación de Elixir del repositorio](../../instalacion/06_elixir.md).
