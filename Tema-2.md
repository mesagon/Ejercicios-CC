### Ejercicio 1: Darse de alta en algún servicio PaaS tal como Heroku, zeit, BlueMix u OpenShift.

Me he dado de alta en [Heroku](https://id.heroku.com/login).

### Ejercicio 2: Crear una aplicación en OpenShift o en algún otro PaaS en el que se haya dado uno de alta. Realizar un despliegue de prueba usando alguno de los ejemplos incluidos con el PaaS.

Vamos a crear y desplegar la aplicación de prueba con python que nos proporciona Heroku en su [guía](https://devcenter.heroku.com/articles/getting-started-with-python) de inicialización. Para ello abrimos un terminal y en primer lugar clonamos el repositorio que contiene el código de la aplicación de prueba y accedemos al repositiorio clonado.

~~~
$ git clone https://github.com/heroku/python-getting-started.git
$ cd python-getting-started
~~~

Una vez hecho esto procedemos a desplegar la aplicación. Para ello creamos en primer lugar la aplicación en Heroku desde la terminal.

~~~
$ heroku create
~~~

Esto preparará a Heroku para recibir el código fuente de la aplicación de muestra. Tras esto, se nos habrá dado una url con la cual podemos acceder a la aplicación creada y también se ha creado un git remoto llamado heroku asociado con el repositorio local (python-getting-started en este caso). Dicho esto, podemos desplegar la aplicación utilizando git.

~~~
$ git push heroku master
~~~

Tras esto, nuestra aplicación se abrá desplegado. Para asegurarnos de que hay al menos una instancia de la aplicación ejecutándose ejecutamos lo siguiente.

~~~
$ heroku ps:scale web=1
~~~

Finalmente accedemos a la aplicación a través de su url y veremos una página que nos indicará que esto es una aplicación de muestra de Heroku con python.

### Ejercicio 3: Realizar una app en express (o el lenguaje y marco elegido) que incluya variables como en el caso anterior.

En primer lugar debemos de crear el esqueleto de la aplicación con express.

~~~
$ express app_ej3
~~~

Eso creará una carpeta app_ej3 en el directorio actual con el esqueleto de la aplicación. A continuación accedemos a la carpeta e instalamos las dependencias.

~~~
$ cd app_ej3
$ npm install
~~~

Una vez hecho esto, la aplicación se encuentra en la carpeta app.js. La modificamos para crear nuestra aplicación, cuyo código es el siguiente.

 ~~~
 var express = require('express');
 var app = express();

 // Vector para almacenar nombres.
 var nombres = new Array;

 // Operación PUT para introducir un nuevo nombre en el
 // vector.
 app.put('/nombre/:nomb', function (req, res) {

 	// Almacenar el nombre en el vector.
 	nombres.push(req.params.nomb);  
 	res.send("Nombre almacenado");
 });

 // Operación GET para consultar los nombres introducidos
 // hasta el momento.
 app.get('/nombres', function(req,res){

 	// Responder al cliente con los nombres del vector.
 	res.send(nombres);
 });

 app.listen(app.get('port'), function () {
   console.log("Aplicacion ejecutandose en localhost:" + app.get('port'));
 });
 ~~~

En este caso, el cliente podrá enviarle a la aplicación nombres a través de una operación PUT y esta los irá almecenando en un vector. Además mediante una operación GET el cliente también podrá consultar los nombres almacenados por la aplicación hasta el momento.

### Ejercicio 4: Crear pruebas para las diferentes rutas de la aplicación.

En primer lugar debemos de instalar el módulo supertest en el directorio de trabajo de la aplicación.

~~~
$ npm install supertest --save-dev
~~~

A continuación debemos de exportar la aplicación añadiendo al fichero app.js lo siguiente.

~~~
module.exports = app;
~~~

Ahora creamos la carpeta test en el directorio de trabajo de la aplicación para almacenar los tests. Dentro de esta carpeta creamos el fichero test.js que será el que ejecute los tests sobre la aplicación.

~~~
$ mkdir test
$ cd test
~~~

El fichero test.js tiene el siguiente código.

~~~
var request = require('supertest'),
app = require('../app.js');

describe( "PUT nombre", function() {
	it('should create', function (done) {
	request(app)
		.put('/nombre/Jesus')
		.expect('Content-Type', 'text/html; charset=utf-8')
		.expect(200,done);
	});
});

describe( "GET nombres", function() {
	it('should create', function (done) {
	request(app)
		.get('/nombres')
		.expect('Content-Type', /json/)
		.expect(200,done);
	});
});
~~~

Una vez creados los tests los ejecutamos con el comando

~~~
$ mocha test/test.js
~~~

### Ejercicio 5: Instalar y echar a andar tu primera aplicación en Heroku.

Vamos a desplegar la aplicación de ejemplo de Heroku con node. Para ello, en primer lugar clonamos el repositorio de dicha aplicación y nos situamos en él.

~~~
$ git clone git@github.com:heroku/node-js-getting-started.git
$ cd node-js-getting-started
~~~

A continuación, dentro del directorio creamos la aplicación en Heroku, la cual estará asociada al directorio en el que nos encontramos.

~~~
$ heroku apps:create --region eu appej4
~~~

Una vez hecho esto, se creará la aplicación en heroku y nos asignarán una URL para acceder a la misma (En este caso, https://appej4.herokuapp.com/). Ahora, desplegamos la aplicación haciendo un push desde el directorio actual.

~~~
$ git push heroku master
~~~

Con esto, al acceder a la URL anterior, podremos ver la aplicación de ejemplo desplegada.

### Ejercicio 6: Usar como base la aplicación de ejemplo de heroku y combinarla con la aplicación en node que se ha creado anteriormente. Probarla de forma local con foreman. Al final de cada modificación, los tests tendrán que funcionar correctamente; cuando se pasen los tests, se puede volver a desplegar en heroku.

Para poder desplegar de una forma más sencilla nuestra aplicación anteriormente creada en Heroku, podemos combinarla con la aplicación ejemplo de Heroku desplegada en el ejercicio 5. De esta forma, el fichero index.js de la aplicación quedaría como sigue.

~~~
const express = require('express')
const path = require('path')
const PORT = process.env.PORT || 5000
const app = express()

// Vector para almacenar nombres.
var nombres = new Array;

// Operación PUT para introducir un nuevo nombre en el
// vector.
app.put('/nombre/:nomb', function (req, res) {

	// Almacenar el nombre en el vector.
	nombres.push(req.params.nomb);  
	res.send("Nombre almacenado");
});

// Operación GET para consultar los nombres introducidos
// hasta el momento.
app.get('/nombres', function(req,res){

	// Responder al cliente con los nombres del vector.
	res.send(nombres);
});

app
  .use(express.static(path.join(__dirname, 'public')))
  .set('views', path.join(__dirname, 'views'))
  .set('view engine', 'ejs')
  .get('/', (req, res) => res.render('pages/index'))
  .listen(PORT, () => console.log(`Listening on ${ PORT }`))

module.exports = app
~~~

A continuación, el tests que habíamos creado anteriormente para nuestra aplicación debemos de ponerlo en la carpeta test (hay que crearla) en la aplicación de ejemplo.
El código del test es el siguiente (cambia la ruta para lanzar la aplicación).

~~~
var request = require('supertest'),
app = require('../index.js');

describe( "PUT nombre", function() {
	it('should create', function (done) {
	request(app)
		.put('/nombre/Jesus')
		.expect('Content-Type', 'text/html; charset=utf-8')
		.expect(200,done);
	});
});

describe( "GET nombres", function() {
	it('should create', function (done) {
	request(app)
		.get('/nombres')
		.expect('Content-Type', /json/)
		.expect(200,done);
	});
});
~~~

En este caso, he tenido problemas para instalar foreman, por lo tanto, se ha probado la aplicación de forma local con npm. Para ello nos situamos en el directorio local de la aplicación y ejecutamos en la línea de órdenes los siguientes comandos.

~~~
$ npm test
$ npm start
~~~

Con $ npm test pasamos los tests a la aplicación, mientras que con $ npm start la ejecutamos.

### Ejercicio 7: Haz alguna modificación a tu aplicación en node.js para Heroku, sin olvidar añadir los tests para la nueva funcionalidad, y configura el despliegue automático a Heroku usando Snap CI o alguno de los otros servicios, como Codeship, mencionados en StackOverflow.

Se ha añadido la funcionalidad de borrar mediante un PUT un nombre existente en el vector. De esta forma, el código de index.js quedaría como sigue.

~~~
const express = require('express')
const path = require('path')
const PORT = process.env.PORT || 5000
const app = express()

// Vector para almacenar nombres.
var nombres = new Array;

// Operación PUT para introducir un nuevo nombre en el
// vector.
app.put('/nombre/:nomb', function (req, res) {

	// Almacenar el nombre en el vector.
	nombres.push(req.params.nomb);  
	res.send("Nombre almacenado");
});

// Operación PUT para eleiminar un nombre en el
// vector.
app.put('/borrar/:nomb', function (req, res) {

	// Si el nombre se encuentra en el vector,
	// lo eliminamos, en caso contrario no hacemos
	// nada.
	var index = nombres.indexOf(req.params.nomb);

	if (index > -1){

		nombres.splice(index,1);
		res.send("\nNombre eliminado");

	}
	else{

		res.send("\nEl elemento no existe");
	}
});

// Operación GET para consultar los nombres introducidos
// hasta el momento.
app.get('/nombres', function(req,res){

	// Responder al cliente con los nombres del vector.
	res.send(nombres);
});

app
  .use(express.static(path.join(__dirname, 'public')))
  .set('views', path.join(__dirname, 'views'))
  .set('view engine', 'ejs')
  .get('/', (req, res) => res.render('pages/index'))
  .listen(PORT, () => console.log(`Listening on ${ PORT }`))

module.exports = app
~~~

También se ha creado un test para probar la nueva funcionalidad.

~~~
var request = require('supertest'),
app = require('../index.js');

describe( "PUT nombre", function() {
	it('should create', function (done) {
	request(app)
		.put('/nombre/Jesus')
		.expect('Content-Type', 'text/html; charset=utf-8')
		.expect(200,done);
	});
});

describe( "PUT borrar nombre", function() {
	it('should create', function (done) {
	request(app)
		.put('/borrar/Jesus')
		.expect('Content-Type', 'text/html; charset=utf-8')
		.expect(200,done);
	});
});

describe( "GET nombres", function() {
	it('should create', function (done) {
	request(app)
		.get('/nombres')
		.expect('Content-Type', /json/)
		.expect(200,done);
	});
});
~~~

Por último se ha configurado codeship para realizar el despliege de nuestra aplicación cada vez que hacemos un push hacia github con [este tutorial](https://blog.codeship.com/heroku-github-nodejs-deployment/).

### Ejercicio 8: Preparar la aplicación con la que se ha venido trabajando hasta este momento para ejecutarse en un PaaS, el que se haya elegido.

Esto ya se ha hecho en el ejercicio 7.
