1. Punto de partida y nombre de la copia.
Este proyecto es una copia del TP 06, conservada en un repositorio independiente con el nombre tp-07-salas-mongodb.
Se mantienen las funcionalidades anteriores:
    •	Inicio de la aplicación Express.
    •	Consulta del estado mediante /estado.
    •	Listado de reservas desde la aplicación web.
    •	Formulario de creación de reservas.
    •	Validación de formularios y respuestas HTTP.
    •	Recursos estáticos, CSS, vistas y respuesta para rutas inexistentes.


2. Versiones utilizadas y confirmación del uso de Atlas Free (M0).
Tecnología	        Versión o configuración
Node.js	            [Completar con node --version]
npm	                [Completar con npm --version]
Express	            [Completar con la versión instalada]
Mongoose	        [Completar con la versión instalada]
MongoDB Atlas	    Clúster Free (M0)
Base de datos	    reservas_salas_u7
Módulos de Node.js	CommonJS

Las versiones de las dependencias se encuentran registradas en package.json y package-lock.json.


3. Pasos desde una descarga nueva: instalar dependencias, preparar el clúster gratuito, usuario e IP permitida, copiar la plantilla de entorno y ejecutar.

3.1. Descargar el proyecto
Clonar el repositorio y acceder a su directorio:
git clone URL_DEL_REPOSITORIO
cd tp-07-salas-mongodb
Reemplazar URL_DEL_REPOSITORIO por la URL real del repositorio de GitHub.

3.2. Instalar dependencias
Desde la raíz del proyecto, ejecutar:
npm install
Este comando instala las dependencias declaradas en package.json y permite generar o actualizar package-lock.json cuando corresponde.
Mongoose es la dependencia utilizada para definir el esquema, validar las reservas e interactuar con MongoDB.

3.3. Preparar MongoDB Atlas
    1.	Iniciar sesión en MongoDB Atlas.
    2.	Crear un proyecto o utilizar uno existente.
    3.	Crear un clúster de tipo Free (M0).
    4.	Crear un usuario de base de datos con permisos de lectura y escritura.
    5.	Agregar la dirección IP desde la cual se ejecutará la aplicación a la lista de acceso de red (Network Access).
    6.	Obtener la cadena de conexión para una aplicación Node.js desde la opción de conexión del clúster.
    7.	Conservar la URI en forma privada y utilizarla en el archivo .env.

La conexión de este trabajo apunta a una base de datos propia llamada reservas_salas_u7.
La base y la colección pueden crearse automáticamente cuando se guarda el primer documento, siempre que el usuario tenga permisos suficientes.

3.4. Configurar las variables de entorno
Copiar el archivo de ejemplo:
cp .env.example .env
En Windows, también se puede copiar el archivo desde el explorador de archivos.
Editar .env y reemplazar los marcadores por los valores reales proporcionados por Atlas:
PORT=3000
NODE_ENV=development
MONGODB_URI=mongodb+srv://USUARIO:CLAVE@HOST/reservas_salas_u7?retryWrites=true&w=majority
La URI anterior es únicamente un ejemplo: USUARIO, CLAVE y HOST deben sustituirse por los datos reales del clúster. Se deben conservar las opciones de conexión proporcionadas por Atlas y utilizar el nombre de base de datos reservas_salas_u7.
Si la contraseña contiene caracteres reservados para las URI, deben codificarse correctamente según las reglas de conexión de MongoDB.
Seguridad: el archivo .env contiene información privada y no debe subirse a GitHub. El archivo .env.example sí se versiona porque contiene únicamente marcadores y valores de ejemplo.



4. Diagnóstico DNS y resolutores usados, si hubo ajustes; comandos exactos de conexión, creación, listado, calidad y arranque web.

La configuración general se encuentra en src/config/entorno.js. Allí se centralizan las variables del entorno, el puerto HTTP y la lectura de la URI mediante MONGODB_URI.
La conexión se implementa en src/config/base-datos.js, que utiliza Mongoose para conectarse a MongoDB Atlas.
Antes de establecer la conexión, se configura la resolución DNS de Node.js mediante:
dns.setServers(["8.8.8.8", "1.1.1.1"]);
Este ajuste utiliza resolutores DNS públicos para resolver nombres de dominio. Puede ayudar ante determinados problemas de resolución SRV o TXT relacionados con conexiones mongodb+srv://.
El cambio afecta la resolución DNS utilizada por Node.js; no modifica la configuración de red del sistema operativo, no corrige errores de autenticación y no autoriza direcciones IP en Atlas. Si la red requiere otros resolutores o funciona correctamente con los predeterminados, se debe utilizar la configuración acordada con el docente.
La URI se obtiene de las variables de entorno y no se escribe directamente en el modelo ni en los scripts. Si falta la variable o la URI tiene un prefijo incorrecto, el programa debe informar el problema de configuración y terminar con un código de salida distinto de cero.
Los errores de conexión deben permitir distinguir, cuando corresponda, el nombre del error, el código DNS y la operación de resolución, sin exponer credenciales ni imprimir la URI privada.

Comprobar la conexión
Ejecutar desde la raíz:
node --env-file=.env scripts/comprobar-conexion.js

El programa espera a que la conexión se establezca antes de confirmar el éxito. No crea reservas y cierra la conexión tanto si termina correctamente como si ocurre un error.
Resultado observado: [Completar con el resultado real].



5. Mapa de archivos y explicación de esquema, modelo y colección.
La organización principal del laboratorio es la siguiente:

tp-07-salas-mongodb/
├── src/
│   ├── config/
│   │   ├── entorno.js
│   │   └── base-datos.js
│   ├── modelos/
│   │   └── reserva.js
│   ├── index.js
│   ├── app.js
│   └── ...archivos conservados del TP 06
├── scripts/
│   ├── comprobar-conexion.js
│   ├── crear-reserva.js
│   └── listar-reservas.js
├── .env.example
├── .gitignore
├── eslint.config.js
├── package.json
├── package-lock.json
└── README.md
La estructura representa las responsabilidades principales; los demás archivos de la aplicación web se conservan según la organización original del TP 06.


6. Campos y reglas, incluyendo límites que todavía no se controlan.

Campo	    Tipo	Reglas
estudiante	String	Obligatorio y sin espacios en los extremos.
email	    String	Obligatorio, con trim y presencia de @.
sala	    String	Obligatorio. Valores permitidos: Sala Norte, Sala Sur o Sala Multimedia.
fecha	    String	Obligatoria y no vacía después de aplicar trim.
turno	    String	Obligatorio. Valores permitidos: Mañana, Tarde o Noche.
personas	Number	Obligatorio, entero entre 1 y 6.

Las restricciones se implementan mediante las opciones de esquema de Mongoose:
•	required para campos obligatorios.
•	trim para eliminar espacios en los extremos de los textos.
•	enum para restringir los valores de sala y turno.
•	match: /@/ para comprobar la presencia de @ en el email.
•	min y max para limitar la cantidad de personas.
•	validate con Number.isInteger para exigir un número entero.


7. Matriz de pruebas con resultados y evidencia de persistencia entre procesos.

La persistencia se comprueba mediante una secuencia reproducible sobre la base reservas_salas_u7.
Procedimiento
    1.	Ejecutar el programa de listado y registrar la cantidad inicial.
    2.	Ejecutar el programa de creación y guardar el _id confirmado.
    3.	Esperar a que termine el proceso de creación.
    4.	Iniciar un nuevo proceso de Node.js ejecutando el programa de listado.
    5.	Comprobar que aparece el mismo _id y que la cantidad aumentó en uno.
    6.	Ejecutar nuevamente el listado sin realizar otras escrituras.
    7.	Comprobar que el _id continúa presente y que la cantidad no cambió.


Registro de resultados
Ensayo	                                Cantidad de reservas	Resultado observado
Listado inicial	                        [Completar]	            [Completar]
Después de crear una reserva	        [Completar]	            [Completar]
Listado desde un proceso nuevo	        [Completar]	            [Completar]
Segundo listado sin nuevas escrituras	[Completar]	            [Completar]

Matriz de pruebas
Completar los resultados y las cantidades con los valores observados durante la ejecución. No se deben inventar resultados ni eliminar registros para conseguir una cantidad determinada.
Para cada prueba inválida, registrar la cantidad de documentos antes y después. La cantidad debe permanecer igual cuando el guardado es rechazado.
Las pruebas de validación se realizan modificando temporalmente el objeto ficticio de scripts/crear-reserva.js, de a un campo por vez. Al finalizar, se debe restaurar el objeto válido.
Para la prueba de autenticación se conserva la URI correcta en privado y se utiliza temporalmente una contraseña incorrecta en .env, sin modificar la contraseña del usuario en Atlas. Luego se restaura la URI válida y se repite la comprobación de conexión.
Si MONGODB_URI también está definida en la terminal, se debe revisar su prioridad respecto de .env para asegurarse de que el ensayo utilice el valor esperado.
No se eliminan documentos para fabricar una colección vacía. Si la base ya contiene registros, se documenta esa condición y se utiliza otra base de laboratorio autorizada, si está disponible.


8. Explicación de por qué el sitio continúa en memoria y qué se abordará en la unidad 8.
El proyecto mantiene dos comportamientos independientes:
    •	Aplicación web: utiliza el servicio en memoria heredado del TP 06. Sus rutas y controladores conservan el funcionamiento anterior.
    •	Laboratorio de persistencia: utiliza Mongoose y MongoDB Atlas desde programas de terminal para crear y consultar documentos persistentes.

Esta separación permite comprobar la conexión, el esquema, las validaciones y la persistencia sin introducir cambios funcionales en el servidor HTTP existente.
En la unidad 8 se abordará la integración de la persistencia con la aplicación, incluyendo los cambios necesarios para que el servicio web utilice operaciones asíncronas y trabaje con los documentos almacenados en MongoDB, según las consignas de esa unidad.


9. Confirmación de que no se publican .env, credenciales ni datos personales.

El repositorio incluye el código fuente, los archivos de configuración de ejemplo, las dependencias declaradas, el archivo de bloqueo y la documentación.
No se deben versionar:
    •	.env.
    •	Credenciales de MongoDB Atlas.
    •	URI de conexión con usuario o contraseña reales.
    •	node_modules.
    •	Otros archivos que contengan secretos o datos personales.
