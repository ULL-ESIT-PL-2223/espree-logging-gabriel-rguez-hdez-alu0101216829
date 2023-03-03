# Práctica 4: Espree Logging

- **Asignatura**: Procesadores de Lenguajes
- **Autor**: Gabriel Rodríguez Hernández
- **Email**: *alu0101216829@ull.edu.es*


## Índice       <!-- omit in toc -->
- [Práctica 4: Espree Logging](#práctica-4-espree-logging)
  - [Código](#código)
  - [Retos](#retos)
  - [Publicación de paquete NPM](#publicación-de-paquete-npm)
  - [Estudio de Covering](#estudio-de-covering)

<br>

## Código

Respecto al código, en esta práctica solo hemos tenido que hacer un fichero, llamado `logging-espree.js`. La primera función que encontramos es `transpile()`, función que se encargará de leer un fichero con el código inicial, y luego creara otro fichero que contendrá el codigo final. Para ello hay que hacer un `readFile` y un `writeFile`. Es importante destacar el uso de **await** ya que tuve una serie de problemas por despistes al no usarlo antes de los dos métodos mencionados anteriormente, ya que a la hora de hacer los test, queria comprobar el resultado con el fichero que creaba transpile, pero al no esperar a que esta lo creará, comparaba el resultado esperado con un fichero que aún no existía. El resultado final es el siguiente:

```js
export async function transpile(inputFile, outputFile="salida.js") {
  let input = await fs.readFile(inputFile, 'utf-8', (err) => {
    if (err) {
      console.log(err);
    }
  });
  const afterAdd = addLogging(input);

  await fs.writeFile(outputFile, afterAdd, (err) => {
    if (err)
      console.log(err);
    else {
      console.log("File written successfully\n");
      console.log("The written has the following contents:");
    }
  });
}
```

Por otra parte tenemos la función `addLogging`, cuya responsabilidad es buscar los nodos del AST que nos interesan. Para ello, primero crea el AST con el método `parse()` que tiene **espree**. Una vez tenemos el arbol, exploraremos todos sus nodos buscando los tipos que nos interesan, por ejemplo `FunctionDeclaration` y `FunctionExpression` en nuestro caso. Si encuentra algún nodo de este estilo, llamaremos a la última función. Finalizado el recorrido haremos un `escodegen` del arbol y eso será lo que retorne la funcion y se escriba en el fichero de salida.

```js
export function addLogging(code) {
  const ast = espree.parse(code, {ecmaVersion: 6, loc: true});
    estraverse.traverse(ast, {
        enter: function(node, parent) {
            if (node.type === 'FunctionDeclaration' ||
                node.type === 'FunctionExpression') {
                addBeforeCode(node);
            }
        }
    });
    return escodegen.generate(ast)
}
```

Por último, encontramos la función `addBeforeCode()`, que recibe como parámetro un nodo, en principio el de una función, y segun su tipo, añadiremos un console log en el codigo con su nombre o con `<anonymous function>`(en el caso de que esta sea un `functionExpresion`). Además añadiremos sus parámetros. De manera que el resultado final deberia ser *`console.log(Entering *nombre de la funcion*(*parametros*));`*.

```js
function addBeforeCode(node) {
  const name = node.id ? node.id.name : '<anonymous function>';
  let arrayNames = []
  for (let iterator in node.params) {
    arrayNames.push('${ ' + (node.params[iterator].name) + " }")
  }
  var beforeCode = "console.log(`Entering " + name + "(" + arrayNames.toString() +  ")" + "`);";
  var beforeNodes = espree.parse(beforeCode, {ecmaVersion:12}).body;
  node.body.body = beforeNodes.concat(node.body.body);
}
```

## Retos

Como retos albergamos dos, en primer lugar reconocer las **Funciones de flecha gorda**, y en segundo lugar reconocer el **Numero de línea**. Para albergar estos dos desafíos, se ha añadido código tanto en `addLogging()` como en `addBeforeNode`. En la primera reconoceremos las funciones flechas. Para ello simplemente pondremos en el `if`, un **'or funciones flecha'**.

```js
if (node.type === 'FunctionDeclaration' ||
  node.type === 'FunctionExpression' ||
  node.type === 'ArrowFunctionExpression') {
  addBeforeCode(node);
}
```

Por otra parte, para reconocer la línea es bastante sencillo. Simplemente en la función `addBeforeCode`, añadiremos la siguiente variable a la hora de escribir el console.log: `") at line " + node.loc.start.line `. Cuyo resultado final seria:

```js
var beforeCode = "console.log(`Entering " + name + "(" + arrayNames.toString() +  ") at line " + node.loc.start.line + "`);";
var beforeNodes = espree.parse(beforeCode, {ecmaVersion:12}).body;
node.body.body = beforeNodes.concat(node.body.body);
```

## Publicación de paquete NPM

Para este apartado, nos tendremos que registrar en la página [npmjs](https://www.npmjs.com/). Una vez nos hemos registrado pasaremos a loguearnos desde la terminal con el comando `npm login` Este comando nos abrira una ventana en el navegador para entrar a la pagina. una vez pongamos correctamente nuestro usuario y nuestra contraseña, ya estaremos logueados.

[![Image from Gyazo](https://i.gyazo.com/410d1a38ce790b2e9292b6c7a7a6f954.png)](https://gyazo.com/410d1a38ce790b2e9292b6c7a7a6f954)

Una vez hemos hecho esto, solo queda subir nuestro paquete. Para ello debemos editar el `package.json`. Cambiaremos el `name` y le pondremos nuestro alu seguido del nombre de la practica y además, le pondremos una version de la siguiente manera

```json
"name": "@alu0101216829/espree-logging",
"version": "0.9.1"
```

Tambien ejecutaremos `Jsdoc` para el README.md para que este fichero contenga la documentacion de las funciones  que tenga nuestro modulo.

Y ahora, si podemos subir el paquete con el siguiente comando: `npm publish --access=public`

[![Image from Gyazo](https://i.gyazo.com/03cc8a4a06ee3a3e1012259d3d509c33.png)](https://gyazo.com/03cc8a4a06ee3a3e1012259d3d509c33)

Así se vería en la página web:

[![Image from Gyazo](https://i.gyazo.com/414a5e12a515ae2a618a081f8da407bf.png)](https://gyazo.com/414a5e12a515ae2a618a081f8da407bf)

Por último descargaremos el paquete en nuestro proyecto y comprobaremos que este funciona correctamente. Además al hacer el install se nos añade la siguiente linea en el `json`: `"@alu0101216829/espree-logging": "^0.9.1",`.

[![Image from Gyazo](https://i.gyazo.com/eff806853e699ded7d191849f7560144.png)](https://gyazo.com/eff806853e699ded7d191849f7560144)

## Estudio de Covering

Para ello utilizaremos `c8` en lugar de `nyc` ya que este nos ha dado problemas a lo largo de las pruebas. Como se puede observar en el estudio se han añadido nuevos tests, y todos se pasan correctamente. Ademas en el estudio se observa que algunas lineas no se cubren esto es debido a que son las lineas de cuando se espera un error.

[![Image from Gyazo](https://i.gyazo.com/2be4b9ca8a5921168a0efe6b31c4063a.png)](https://gyazo.com/2be4b9ca8a5921168a0efe6b31c4063a)
