# control-de-h-bitos
proyecto de habit tracker

- BACKEND

- instale las dependencias EXPRESS, MONGOOSE, CORS y DOTENV
- Modelo SERVER
* dotenv.config() → carga variables de entorno (como la URL de Mongo).
* app.use(cors()) → habilita comunicación con el frontend.
* app.use(express.json()) → transforma el cuerpo del request a JSON automáticamente.
* mongoose.connect() → conecta la app con la base de datos MongoDB.
* app.get("/ping") → endpoint de prueba (si funciona, sabemos que el server está vivo).
* app.listen(PORT) → arranca el servidor en el puerto especificado.

- Creo el modelo USER
* required: true → indica que ese campo es obligatorio.
* unique: true → MongoDB se asegura de que no haya dos usuarios con el mismo email.
* timestamps: true → agrega automáticamente createdAt y updatedAt. Muy útil para estadísticas o logs.
* Usamos Mongoose porque nos permite trabajar con MongoDB usando objetos de JS y nos da validación fácil.

- Creo modelo userRoutes
- creo los endpoints /register y /login
* bcrypt → encripta la contraseña antes de guardarla, para que nadie pueda leerla si roban la DB.
* jwt.sign() → crea un token que el frontend puede usar para identificar al usuario en futuras peticiones.
* expiresIn → el token caduca después de un tiempo (7 días en este ejemplo).
* Respondemos con token + datos básicos del usuario, nunca la contraseña.

- Creo middleware auth
* Este middleware se ejecuta antes de cualquier ruta protegida.
* Toma el token del header Authorization: Bearer TOKEN.
* Si es válido, agrega req.user con la info del usuario y deja continuar.
* Si no, devuelve error 401/403.

- Creo modelo Habit
* userId → vincula el hábito con un usuario específico.
* frecuencia → limita los valores posibles (enum).
* completadoHoy → para estadísticas diarias.
* timestamps → guarda createdAt y updatedAt.

- Creo habitRoute
* Todas las rutas usan verifyToken, por lo que solo usuarios logueados pueden acceder.
* Los métodos find, findOneAndUpdate, findOneAndDelete siempre filtran por userId → un usuario no puede tocar hábitos de otro.
* CRUD completo: Create, Read, Update, Delete.

- Implemento concepto de ESTADÍSTICA a los hábitos
* Queremos poder calcular, por ejemplo:
* Hoy: cuántos hábitos completó el usuario.
* Últimos 7 días: cuántos hábitos completados cada día.
* Frecuencia: hábitos diarios vs semanales.
* Para eso, agrego un campo history en el modelo Habit que guarde cada vez que se completa un hábito.
* history → guarda un arreglo de objetos {date, completed}
* Cada vez que un usuario completa un hábito, agregamos un registro con la fecha actual.
* Esto permite calcular estadísticas históricas sin modificar los datos originales.

- Implemento concepto de ESTADÍSTICA a la ruta de hábitos
* Normalizamos la fecha a medianoche para evitar problemas con horas distintas.
* Si el hábito ya fue completado hoy, devolvemos error 400.
* Agregamos {date: today, completed: true} al arreglo history.
* today → cuenta cuántos hábitos completó el usuario hoy.
* last7Days → array con cantidad de hábitos completados cada día (7 días).
* Esto servirá en React para mostrar gráficos de progreso.
