## Introduccion

1. GO es un lenguaje compilado
2. GO esta organizado en paquetes, que serian lo equivalente a librerias de otros lenguajes
3. Un paquete consiste en uno o mas ficheros .go ubicados en el mismo directorio
4. Cada fichero empieza con una declaración que indica a que paquete pertenece
5. Luego estan los imports, que son las importaciones de otros paquetes. Uno de los mas habituales para entrada/salida es fmt
6. El paquete "main" es especial. Define un standalone ejecutable, no una libreria. Asi mismo la función main es "especial" ya que indica donde empieza la ejecución del programa
7. go doc package
8. // /* */ comentarios

*helloworld.go*

```go
package main
import "fmt"

func main() {
    fmt.Println("Hello World!")
}
```

##### Urls

https://apuntes.de/golang/eliminar-elementos-del-slice/#gsc.tab=0

https://tour.golang.org/methods/2

### Argumentos por command line

1. Es necesario usar el paquete "os"
2. La entrada de argumentos se realiza por la variable Args (ios.Args)
3. La variable Args es un array que contiene los argumentos
4. El primer elemento ARGS[0] es el nombre del programa. Luego van el resto de argumentos

*Command line 1*

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	var s, sep string
	for i := 1; i < len(os.Args); i++ {
		s += sep + os.Args[i]
		sep = " "
	}
	fmt.Println(s)
    fmt.Println(os.Args[0])
}
```

*Command line 2 - for range*

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	s, sep := "", ""
	for _, arg := range os.Args[1:] {
		s += sep + arg
		sep = " "
	}
	fmt.Println(s)
	for a, arg := range os.Args[1:] {
		fmt.Println("(Index,value) =", a, arg)
	}

}
```

*Command line 3 - Join*

```go
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	fmt.Println(strings.Join(os.Args[1:], " "))

```

### Buscando lineas duplicadas - Argumentos por entrada de usuario

**Nota:**Control-D (linux you can press "CTRL+D", which signals EOF from terminal.)

*dup1.go*

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	counts := make(map[string]int)
	input := bufio.NewScanner(os.Stdin)
	for input.Scan() {
		counts[input.Text()]++
	}
	// Note: ignoring potential errors from input.Err()
	for line, n := range counts {
		if n > 1 {
			fmt.Printf("%d\t%s\n", n, line)
		}
	}
}
```

1. La instrucción "map" crea una dupla de valores "key,value". La key pueden ser de cualquier tipo que puedan usar el operador "==" (igualdad). El "value" puede ser de cualquier tipo.

2. La función "make" crea una nueva entidad "map" y se le asigna a la variable "counts"

3. Cada vez que se lee una nueva linea del input, la linea (string) es utilizada como la "key" dentro del map (string) y su "value" es aumentado. La linea *counts[input.Text()]++* equivale a:

   1. line := input.Text()
   2. counts[line] = counts[line]+1

4. El tipo "Scanner" del paquete bufio se encarga de leer inputs de teclado y almacenarlo en "words" o "lineas"

5. Cada llamada a la "variable" input (de tipo New Scan) lee lo que se esta introduciendo por teclado.

6. El resultado se recupera utilizando la llamada input.Text()

7. La función "Printf" produce una salida "formateada" de los argumentos que se indiquen

   1. %d: decimal
   2. %x: hexadecimal
   3. %b: binario
   4. %f: float
   5. %t: boolean (true or false)
   6. %s: string
   7. %v: cualquier valor en su formato natural
   8. %T: tipo de cualquier valor
   9. %%: Porcentaje caracter

   #### Ficheros

   *dup2.go*

   ```go
   package main
   
   import (
   	"bufio"
   	"fmt"
   	"os"
   )
   
   func main() {
   	counts := make(map[string]int)
   	files := os.Args[1:]
   	if len(files) == 0 {
   		countLines(os.Stdin, counts)
   	} else {
   		for _, arg := range files {
   			f, err := os.Open(arg)
   			if err != nil {
   				fmt.Fprintf(os.Stderr, "dup2: %v\n", err)
   				continue
   			}
   			countLines(f, counts)
   			f.Close()
   		}
   
   	}
   	for line, n := range counts {
   		if n > 1 {
   			fmt.Printf("%d\t%s\n", n, line)
   		}
   	}
   }
   
   func countLines(f *os.File, counts map[string]int) {
   	input := bufio.NewScanner(f)
   	for input.Scan() {
   		counts[input.Text()]++
   	}
   
   }
   ```

   1. La función os.Open retorna dos valores. El primero es un fichero abierto ( si es posible). El segundo es un valor de tipo "error".
   2. Map es una "referencia" a la estructura de datos creada por make. Cuando un map es pasado a una función, la función recibe una copia de la referencia, por lo tanto, cualquier cambio realizado en la función sobre la estructura **tambien sera modificado en el estructura original**
      1. Dicho de otra forma, lo que se le pasa es la dirección de memoria de la estructura, por lo tanto cualquier cambio realizado dentro de la función en la estructura, fuera de función tambien sera visible. 

*dup3.go*

```go
package main

import (
	"fmt"
	"io/ioutil"
	"os"
	"strings"
)

func main() {
	counts := make(map[string]int)
	for _, filename := range os.Args[1:] {
		data, err := ioutil.ReadFile(filename)
		if err != nil {
			fmt.Fprint(os.Stderr, "dup3: %v\n", err)
			continue
		}
		for _, line := range strings.Split(string(data), "\n") {
			counts[line]++
		}
	}
	for line, n := range counts {
		if n > 1 {
			fmt.Printf("%d\t%s\n", n, line)
		}
	}

}

```

1. En este caso, el programa lee del tiron todos los datos, de golpe, luego separa las lineas y las procesa
2. La función readfile es la que lee todo el fichero y lo almacena en dara
3. La funcion strings.Split divide esos datos en strings

*Ejercicio 1.4*

Modifica dup2 para mostrar el nombre de cada fichero en cada linea repetida.

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	counts := make(map[string]int)
	files := os.Args[1:]
	if len(files) == 0 {
		countLines(os.Stdin, counts)
	} else {
		for _, arg := range files {
			f, err := os.Open(arg)
			if err != nil {
				fmt.Fprintf(os.Stderr, "dup2: %v\n", err)
				continue
			}
			countLines(f, counts)
			f.Close()
		}

	}
	for line, n := range counts {
		if n > 1 {
			fmt.Printf("%d\t%s\t%v\n", n, line, files[0])
		}
	}
}

func countLines(f *os.File, counts map[string]int) {
	input := bufio.NewScanner(f)
	for input.Scan() {
		counts[input.Text()]++

	}

}

```

**NOTA: REVISAR MAKE, MAPS, Y EL COMPARADOR DE LINEAS**

#### Fetching URL - Servidor web

*fetch.go*

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"os"
)

func main() {
	for _, url := range os.Args[1:] {
		resp, err := http.Get(url)
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: %v\n", err)
			os.Exit(1)
		}
		b, err := ioutil.ReadAll(resp.Body)
		resp.Body.Close()
		if err != nil {
			fmt.Fprintf(os.Stderr, "fetch: reading %s: %v\n", url, err)
			os.Exit(1)
		}
		fmt.Printf("%s", b)
	}
}
```

1. El paquete net/http provee una función que permite realizar un request. http.Get
2. El resultado de http.Get se devuelve a la estructura resp.
3. El campo Body de res, contiene la respuesta del GET (es un stream)
4. ioutil.ReadAll lee la respuesta entera y la guarda en b
5. Cierra el stream para evitar leaks
6. Se imprime la respuesta

*fetchall.go*

```go
package main

import (
	"fmt"
	"io"
	"io/ioutil"
	"net/http"
	"os"
	"time"
)

func main() {
	start := time.Now()
	ch := make(chan string)
	for _, url := range os.Args[1:] {
		go fetch(url, ch) // start a goroutine
	}
	for range os.Args[1:] {
		fmt.Println(<-ch) //receive from channel ch
	}
	fmt.Printf("%.2fs elapsed\n", time.Since(start).Seconds())
}

func fetch(url string, ch chan<- string) {
	start := time.Now()
	resp, err := http.Get(url)
	if err != nil {
		ch <- fmt.Sprint(err) // send to channel ch
		return
	}

	nbytes, err := io.Copy(ioutil.Discard, resp.Body)
	resp.Body.Close()
	if err != nil {
		ch <- fmt.Sprintf("While reading %s: %v", url, err)
		return
	}
	secs := time.Since(start).Seconds()
	ch <- fmt.Sprintf("%.2fs %7d %s", secs, nbytes, url)

}
```

1. Una go routine es una funcion concurrente en ejecucion
2. Un channel es un mecanismo de comunicacion que permite pasar datos de un determinado tipo a otra go routine
3. La funcio  main es una go routine y se crea otra con la sentencia go
4. Desde main se crea un canal de strings (se usa make)
5. Se llama de forma asincrona a la goroutine fechall por cada elemento que se haya introducido
6. La función io.Copy lee el body de la respuesta 
7. Cuando una goroutine intenta recibir o enviar a un canal se bloquea hasta que otra subrutina intenta leer o recibir de ese canal, la informacion entonces es comunicada y las goroutines se desbloquean.
8. En este ejemplo, cada goroutine envia un valor ch <- expression y main lo recibe <-ch. Una vez main recibido y procesado todo como una unidad, finaliza todo, el main y las goroutines al mismo tiempo

*web server*

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {

	http.HandleFunc("/", handler) //each request calls hander
	log.Fatal(http.ListenAndServe("localhost:9000", nil))
}

//handler echoes the Path component of the request URL

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Url.Path = %q\n", r.URL.Path)
}

```

1. La funcion main conecta un handler para recibir las URLS cuyo path empiece por /
2. Levanta un servidor en el puerto 9000
3. Cada request es representado como una estructura http.Request que contiene un numero de campos relacionados, uno de los cuales es la URL 
4. Del request se extrae el URL.Path y se escribe por pantalla.

*server2*

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"sync"
)

var mu sync.Mutex
var count int

func main() {
	http.HandleFunc("/", handler)
	http.HandleFunc("/count", counter)
	log.Fatal(http.ListenAndServe("localhost:9000", nil))
}

func handler(w http.ResponseWriter, r *http.Request) {
	mu.Lock()
	count++
	mu.Unlock()
	fmt.Fprintf(w, "Url.Path = %q\n", r.URL.Path)
}

func counter(w http.ResponseWriter, r *http.Request) {
	mu.Lock()
	fmt.Fprintf(w, "Count %d\n", count)
	mu.Unlock()
}
```

1. En este caso se utiliza mu.Lock y Mu.Unlock para bloquear el acceso a la variable. Se utiliza en go routines

*server3.go*

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"sync"
)

var mu sync.Mutex
var count int

func main() {
	http.HandleFunc("/", handler)
	http.HandleFunc("/count", counter)
	log.Fatal(http.ListenAndServe("localhost:9000", nil))
}

func handler(w http.ResponseWriter, r *http.Request) {
	mu.Lock()
	count++
	mu.Unlock()
	fmt.Fprintf(w, "%s %s %s\n", r.Method, r.URL, r.Proto)
	for k, v := range r.Header {
		fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
	}
	fmt.Fprintf(w, "Host = %q\n", r.Host)
	fmt.Fprintf(w, "RemoteAddr = %q\n", r.RemoteAddr)
	if err := r.ParseForm(); err != nil {
		log.Print(err)
	}
	for k, v := range r.Form {
		fmt.Fprintf(w, "Form[%q] = %q\n", k, v)
	}
}

func counter(w http.ResponseWriter, r *http.Request) {
	mu.Lock()
	fmt.Fprintf(w, "Count %d\n", count)
	mu.Unlock()
}

```

1. En el if decimos err := r.ParseForm dentro del mismo if

2. Esto se podria haber hecho asi:

   err := r.ParseForm()

   if err != nil {

   log. Print(err)}

3. Pero de esta manera, el scope de err, es solo durante el if

### Control flow

```go
switch flip() {

​	case "xxx":

​	xxx+

case "details":

default:

}
```



## Cap 2 Program Structure

### Declaraciones

1. var
2. const
3. type
4. func

*declarations.go*

```go
package main

import (
	"fmt"
)

const boiling = 212.0

func main() {
	var f = boiling
	var c = (f - 32) * 5 / 9
	fmt.Printf("Boiling point = %gºF or %gºC", f, c)

}
```

*declarations2.go*

```go
package main

import (
	"fmt"
)

func main() {
	const freezingF, boilingF = 32.0, 212.0
	fmt.Printf("%gºF = %gºC\n", freezingF, fToC(freezingF))
	fmt.Printf("%gºF = %gºC\n", boilingF, fToC(boilingF))
}

func fToC(f float64) float64 {
	return (f - 32) * 5 / 9
}
```

#### Variables

1. Para declarar una variable:

   ```go
   var name type = expression
   var test int = 3
   var greeting string = hello
   var i, j, k int // int int int
   var b, f, s = true, 2.3, "four" // bool, float, string (en este caso, no se indica el tipo pero go lo deduce por los valores)
   ```

2. Si no se le asigna valor, go de forma automatica le asigna un zero null en base  a tu tipo ( 0 si es un int, "" si es una string, false si es un boleano etc)

3. Un juego de variables tambien pueden ser inicializadas por una función que devuelve multiples variables

   ```go
   var f, err = os.Open(name)
   ```

4. La declaracion de esta manera se suele usar para variables locales que necesitan indicar el tipo de forma explicita. Por ejemplo, pq su valor inicial no importa y mas adelante cambiara 

#### Declaracion de variables short

```go
name := expression go
test := 3
i, j := 0, 1
```

1. El tipo es determinado por el tipo de la expression
2. Este tipo de declaracion se usa para definir variables locales
3. := es una declarion
4. = es un asignamiento

```go
i, j = j, i // swap values of i and j
```

5. Tambien puede ser usada para llamar a funciones

#### Punteros

1. Una variable es una "pieza" de la memoria que almacena un valor
2. Un puntero almacena la direccion de una variable, es la dirección de la memoria donde la variable esta almacenada
3. Con un puntero, podemos leer/modificar el valor de una variable de forma indirecta, sin saber su nombre, ya que tenemos su direccion

```go
var x := 3 //int
p := &x //Direccion de la variable X asignada a P. P es del tipo *int. Es un puntero a Int (ya que almacena la direccion de una variable que contiene un int)
*p=4
//Ahora x tambien valdra 4
```

*pointers.go*

```go
package main

import "fmt"

func main() {
	v := 1
	fmt.Println(v)
	incr(&v)
	fmt.Println(incr(&v))
}

func incr(p *int) int {
	*p++
	return *p
}
```

#### The new function

1. La expresion new(T) crea una unnamed variable de tipo T, inicializada al valor 0 correspondiente y devulve su dirección, que es un valor de tipo *T
2. Cada llamada a new retorna una unica direccion. Si se hacen 2 llamadas, se obtienen dos direcciones diferentes de dos variables sin nombre.

```go
package main

import "fmt"

func main() {
	v := 1
	fmt.Println(v)
	incr(&v)
	fmt.Println(incr(&v))

	p := new(int)
	fmt.Println(p)
	fmt.Println(*p)
	fmt.Println(&p)
	*p = 2
	fmt.Println(p)
	fmt.Println(*p)
	fmt.Println(&p)
}

func incr(p *int) int {
	*p++
	return *p
}
```

#### Tiempo de vida de una variable

1. Una variable a nivel de package-level esta viva para todo el programa
2. Una variable local creada a nivel de un for, if, etc, esta viva durante la ejecución del bucle
3. Una variable local de una función, paramertros, resultados etc estan vivos mientras la función se ejecuta. Luego....se reciclan.

#### Asignamientos

1. =
2. +=
3. *=
4. v++ // sumar 1
5. v-- // restar 1
6. x, y = y, x
7. i,j,k = 2,3,5
8. == -> esto es una comparacion
9. != -> indica que es una comparacion cuyo resultado no sea igual

#### Blank identifier

_, err = funcion (retorna los valores a y b)

En este caso _ = a, y err=b

Pues a, se pierde, no se asigna.

#### Declaraciones de tipo

```go
type name underlying-type
type celsius float64
type fahrenheit float64
```



1. Se define un nuevo nombre de tipo que tiene el mismo underlying type ya existentego
2. Celsius y Fahrenheit tienen el mismo subtipo (float64) pero no son el mismo tipo
   1.  var a celsius = 90
   2.  var b fahrenheit = 270
3. A y B son de tipos diferentes

#### Conversiones de tipo

```go
T(x) // donde T es un tipo y x es un valor de otro tipo

	var x float64 = 3.32
	y := int(x)
	fmt.Println(x)
	fmt.Println("esto es ", y)
```

1. Las conversiones de tipo son permitas si los tipos tienen el mismo subtipo igual (como celsius y fahrenheit)
2. Tipos numericos
3. Strings y slices

#### Scope y lifetime

Scope: tiempo de compilacion

LIfetime: Tiempo de ejecución

### Chapter 3 - Basic Data Types

#### Integers

1. int
2. uint
3. otros

#### Float

1. float32
2. float64

#### Booleans

1. true
2. false
3. && - AND
4. || - Or

#### Strings

1. Inmutable secuencia de bytes
2. Funciones en strings package, strconv
3. s:="hello world"
4. fmt.Println(len(s))
5. fmt.Println(s[0], s[7])
6. s[1:]
7. s[0:5]
8. s[:7]
9. s[:]
10. fmt.Println("Hello" + s[5:])
11. t:=s
12. s+= "i m happy"
13. \a - alert (bell)
14. \b - backspace
15. \f -form feed
16. \n - new line
17. \r - carriage return
18. \t - tab
19. \v - vertical tab
20. \\' 
21. \\"
22. \\\

#### Strings to integers

1. y := fmt.Sprintf("%d", x)
2. strconv.Itoa(x) (int->asci)
3. strconv.Atoi (string->int)
4. strconv.ParseInt (string - int)

#### Constantes

```go
const name = expression
const ( pi = 3,14
        e = 2,7)
```

1. Underlying type: boolean, string, number (int, float)
2. iota para generar constantes
3. untyped constants

##### Tipo runa

Rune is a data type in the Go programming language. Rune is character literal which represents int32 value and each int value is mapped to Unicode code point https://en.wikipedia.org/wiki/Code_point

It represents any characters of any alphabet from any language in the world.

Each Character in language has a different meaning. For the English language, the character is from alphabets has ‘B’,‘b’,'&', for Chinese, it is ‘我’

Like other programming languages, Go don’t have a character data type. bytes and rune are used to represent character literals. a byte represents ASCII codes,

rune represents Unicode characters encoded in UTF-8 format.

### Chapter 4 - Tipos compuestos

#### Arrays

1. Un array es un conjunto de elementos del mismo tipo y de tamaño fijo
2. Se puede acceder a un elemento del array mediante indices
3. La funcion len permite saber el tamaño de la array
4. Paso por referencia por puntero:
   1. func zero(ptr *[32]byte)
5. Indicar indice y valor
   1. r := [...]int{99: -1}
   2. Es un array de 100 valores, todos 0, menos el 99 que es -1

#### Slices

1. El concepto es el mismo que un array

2. Es, digamoslo asi, un subconjunto de elementos del array

   1. S:[I,J]
   2. La secuencia es desde I hasta J-1
   3. El numero total de elementos es J-I
   4. (This selects a half-open range which includes the first element, but excludes the last one.)

3. Todos los elementos tienen el mismo tipo

4. Pero, no tiene tamaño fijo

5. []Type

6. Un slice tiene tres componentes: un puntero, un tamaño y una capacidad

   1. El puntero apunta al primer elemento accesible del slice, que no tiene pq ser el primero "literalmente" del slice
   2. El tamaño es el numero actual de elementos. Funcion len
   3. Tamaño < capacidad. Capacidad es el tamaño total. Funcion CAP. Se calcula desde el inicio del slice hasta el ultimo elemento del vector original

7. El indice del slice es s[i:j] 0<=i<=j<=cap(s)

8. Ojo con los indices 

9. slice[1:7] = slice[1:]=slice[:] 

10. Dado que un slice contiene un puntero al array, si se pasa a una función, permite que se modifiquen los valores del array. Es un paso por referencia

##### Funcion make

1.  La función make permite crear un slice de un determinado tipo, longitud y capacidad. Realmente crea una array sin nombre que solo es accesible mediante el slice que devuelve la funcion make
    1. make ([]Type, len): En este caso, al omitir cap, len = cap
    2. make([]Type, len, cap)

##### Funcion Append

1. Se trata de una función para agregar items a los slices
2. En lugar de agregar otro elemento por su índice de referencia podemos usar la instrucción append que y asociarla a una nueva variable. Esto creará un nuevo slice con mayor capacidad sobre esta nueva variable.
3. Eliminar un elemento:
4. Los slices obedecen al concepto de inmutabilidad, esto quiere decir que no pueden eliminarse directamente elementos a través de su índice, por lo que si se requiere eliminar un índice particular, el procedimiento consiste en formar un slice a partir de otros dos.

##### Funcion copy

1. Copia los elementos del slice X a Y

##### Codigo slices

```go
package main

import (
	"fmt"
)

func main() {
	//arrays
	var a [3]int

	fmt.Println(a[0])
	fmt.Println(a[len(a)-1])
	//indices y valores
	for i, v := range a {
		fmt.Printf("%d %d\n", i, v)
	}
	//valores
	for _, v := range a {
		fmt.Printf("%d\n", v)
	}

	var q [3]int = [3]int{1, 2, 3}
	var z [4]int = [...]int{9, 8, 7, 6}

	fmt.Println(len(q))

	for i, v := range q {
		fmt.Printf("%d %d\n", i, v)
	}
	//valores
	for _, v := range z {
		fmt.Printf("%d\n", v)
	}

	//slices

	week := [...]string{1: "Monday", 2: "Tuesday", 3: "Wednesday", 4: "Thuersday", 5: "Friday", 6: "Saturday", 7: "Sunday"}
	fmt.Println(week[5])
	fmt.Println(week[0])
	fmt.Println(len(week))

	laboral := week[1:6]
	fmt.Println(laboral)
	fmt.Println(len(laboral))
	fmt.Println(cap(laboral))
	allweek := week[:]
	fmt.Println(cap(allweek))
	fmt.Println(len(allweek))
	fmt.Println(laboral)
	fmt.Println(week)
	extendlaboral := laboral[:7]
	fmt.Println(extendlaboral)

	var runes []rune
	for _, r := range "Hello World! a" {
		runes = append(runes, r)
	}
	fmt.Printf("%q\n", runes)
	fmt.Println(runes)

	s := make([]int, 5, 8)
	fmt.Println(s)
	v := append(s, 5)
	fmt.Println(v)
	zeta := make([]int, 6, 8)
	copy(zeta, v)
	fmt.Println(zeta)

	//rune = caracter representado por int32

	original := [...]int{1, 2, 3, 4, 5, 6}
	fmt.Println(original)
	nuevo := append(original[:3], original[4:]...) //To append one slice to another, use ... to expand the second argument to a list of arguments.
	//slice = append(slice, elem1, elem2)
	//slice = append(slice, anotherSlice...)
	fmt.Println(nuevo)

}

```

#### Maps

1. Un map es una referencia a una tabla hash

2. Una tabla hash es una coleccion de Key: Value

3. Para crear un map:

   ```go
   map[K]V
   //Donde K es el tipo de las Keys, y V el tipo de las values
   //ejemplo
   ages := make(map[string]int)// mapping from strings to ints
   ages2 := map[string]int{
       "alice": 31,
       "charlie":	34,
   }
   
   ages3 := make(map[string]int)
   ages3["alice"] = 31
   ages4 := map[string]int{}
   delete(ages3, "alice")
   ```

4. No es necesario que el tipo de las Keys y el tipo de las values sea el mismo

5. En el caso que no exista una clave, map devolvera 0. El siguiente codigo es valido:

```go
package main

import (
	"fmt"
)

func main() {

	ages := map[string]int{
		"pep":   39,
		"jordi": 42,
	}

	fmt.Println(ages["pep"])
	fmt.Println(ages["bob"])
	fmt.Println(ages)
	ages["bob"]++
	ages["pep"]++
	fmt.Println(ages["bob"])
	fmt.Println(ages)
	delete(ages, "jordi")
	fmt.Println(ages)

}
```

6. Código para ordenar:

```go
package main

import (
	"fmt"
	"sort"
)

func main() {

	ages := map[string]int{
		"pep":    39,
		"jordi":  42,
		"tarzan": 98,
	}

	for name, age := range ages {
		fmt.Printf("%s\t%d\n", name, age)
	}

	var names []string
	for name, value := range ages {
		fmt.Println(name)
		fmt.Println(value)
		names = append(names, name)
	}

	sort.Strings(names)
	sortednames := make(map[string]int)
	for indicevector, name := range names {
		fmt.Printf("%s\t%d\n", name, ages[name])
		fmt.Println(indicevector)
		sortednames[name] = ages[name]
	}

	fmt.Println(sortednames)
}

```

7. Código para saber si existe una key o no

```
age5 , ok := ages5["bob"]
if !ok {fmt.Println("error")}
```

#### Structs

1. Una estructura es un "grupo, container" que contiene campos (fields). Y los campos pueden ser de diferentes tipos

```go
package main

import "fmt"

func main() {
	type Cat struct {
		Name   string
		Age    int
		Colour string
	}

	var mycat Cat

	mycat.Name = "Batman"

	fmt.Println(mycat)
	fmt.Println(mycat.Name)

	color := &mycat.Colour
	*color = "Black"

	fmt.Println(mycat)

	var age *Cat = &mycat
	age.Age = 9

	fmt.Println(mycat)

}
```

2. Por eficencia, las estructuras se pasan a las funciones como punteros

### Cap 5. Funciones

1. Una declaracion de funciones tiene un nombre, una lista de parametros, una opcional lista de resultados y un cuerpo.

```go
package main

import (
	"fmt"
)

func main() {

	fmt.Println(printer("Hello World"))
	fmt.Println(resta(9, 6))

}

func printer(x string) string {
	fmt.Println("This's the string: ", x)
	return ("ok")
}

func resta(x, y int) (z int) {
	z = x - y
	return
}
```

2. En la lista de parametros podemos encontrar (x int, _ int): _ indicar que hay un parametro que no se utiliza
3. Los argumentos pueden pasarse por valor o por referencia (como punteros, slices etc)
4. Las funciones pueden ser recursivas. Pueden llamarse a si misma de forma directa o indirecta

#### Multiples returns

Una función puede retornar más de un resultado

``` go
func testerror(x string)/*.....*/
{return nil, err}

links, errors := testerror(x)

// para ignorar uno:

links,_ :=testerror(x)

```

2. Si la función retorna 2 resultados, en el código donde se ha reliazado la llamada, se deberan almacenar los dos resultados
3. Si una función tiene "named results" , los operandos de un return statement pueden ser omitidos

```go
// es todo lo mismo
func resta(x, y int) (z int) {
	z = x - y
	return
}

func resta(x, y int) (z int) {
	z = x - y
	return z
}

func resta(x, y int)int {
	z = x - y
	return z
}

func count(n int)(words, images int)

return words, images = return
```

#### Errors

to do





