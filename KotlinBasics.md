# Ciclo de vida de una aplicacion de Kotlin:

- onCreate(): es lo que sucede antes de que la app se muestre en la pantalla. Aquí es donde se inicializan las variables y se configuran las activities.
- onStart(): se llama a este método JUSTO DESPUÉS de que se haya creado la app, pero TODAVÍA NO ES VISIBLE para el usuario.
- onResume(): aquí es donde sucede todo lo que el usuario ve en la pantalla y la mayor parte de sus interaciones con la app.
- onPause(): aquí la app ha perdido el enfoque pero sigue siendo visible, por lo que es un buen lugar para detener las animaciones y liberar recursos.
- onStop(): Sucede cuando se minimiza la app. la activity ya no es visible para el usuario, por lo que es un buen lugar para liberar recursos, hacer guardados, etc.
- onDestroy(): justo antes de destruir la activity.
- onRestart(): cuando se reanuda justo después de destruirla. Al onRestart le sigue un onStart.



### Ejemplo del ciclo de vida:
► Lanzar la aplicación: Se llama a onCreate(), seguido de onStart() y onResume(). La
aplicación ahora está en el estado "Running".

► Minimizar la aplicación: Se llama a onPause() y luego a onStop(). La Activity ahora
está en el estado "Stopped".

► Volver a la aplicación: Se llama a onRestart(), seguido de onStart() y onResume().
La Activity vuelve al estado "Running".

► Cerrar la aplicación: Se llama a onPause(), onStop() y finalmente onDestroy().


# El ciclo de vida en detalle:

## Constructor de nuestro activity

En la primera parte definiremso todas las propiedades que vayamos a usar, esto es:
-Componentes visuales.
-Atributos que necesitemos.
-Companion objects.

El resultado sería este:

````kotlin

class MainActivity : AppCompatActivity() {

    //Creacion de un companion object que es accesible desde todas las activities
    companion object{
        //Creo IMC_KEY para asignar el valor del extra en el intent
        const val IMC_KEY = "IMC_RESULT"
    }

    //Creamos las variables privadas con INICIACIÓN TARDÍA para recoger los elementos visuales
    private lateinit var  viewMale: CardView
    private lateinit var  tvHeight: TextView
    private lateinit var  rsHeight: RangeSlider
    private lateinit var  btnSubWeight: FloatingActionButton
    private lateinit var  btnImcCalculator: Button


    //Creamos los atributos necesarios para la logica de la Activity
    private var isMaleSelected :Boolean = true
    private var currentAge: Int = 23
    private var currentHeight: Double = 1.2

    // A continuación iría el oncreate()

````


## onCreate()
Este método se llama cuando la Activity se está creando. Para asegurar su correcto funcionamiento, haremos que reciba como parametro savedInstanceState: Bundle?. El savedInstances hará que los datos de la Activity permanezcan incluso después del onDestroy o al girar la pantalla.


````kotlin

    override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            enableEdgeToEdge()
            setContentView(R.layout.activity_main)

            // A continuación, llamamos a tres métodos que deben estar SIEMPRE
            //Para iniciar los componentes visuales
            initComponents()

            //Para creamos los listenners de los eventos
            initListeners()

            //Configuraciones visuales de los componentes
            initUI()
    }

````

## initComponents()

Este es un método que vamos a crear nosotros. Arriba ya declaramos las variables, pero aún no están inicliazadas. Es ahora esn este momento que tenemos que inicializarlas:

initComponentes(){
        viewMale = findViewById(R.id.cvMale)
        tvHeight = findViewById(R.id.tvHeight)
        rsHeight = findViewById(R.id.rsHeight)
        btnSubWeight = findViewById(R.id.btnSubWeight)
        btnImcCalculator = findViewById(R.id.btnImcCalculator)
}

## initListeners()

Ya que tenemos los elementos visuales, tenemos que implementar la activación que van a tener. Existen múltiples formas de detectar esta activación, y estas son algunas:

**Más usuales**

- setOnClickListener() : simplemente detecta cuando se ha pulsado.
- addOnChangeListener() : este detecta cambios de estado en el valor del componente.

**Ocasionales**

- setOnLongClickListener() : detecta cuando se mantiene pulsado.
- setOnTouchListener() : detecta cuando se toca el componente o hay algún deslizamiento.
- setOnFocusChangeListener() : cuando se pierde el foco, por ejemplo, un input deja de estar seleccionado.
- addTextChangedListener() : detecta cambios en un input de texto.
  
````kotlin
    //Generamos los listenners de los eventos que necesitamos
        private fun initListeners() {
            this.viewMale.setOnClickListener{
                if(!this.isMaleSelected) {
                    changeGender()
                }
                setGenderColor()
            }


            //Evento de cambio en la propiedad del slider
            this.rsHeight.addOnChangeListener {_, value, _ ->
                //Pasamos a metros
                this.currentHeight = (value / 100.0).toDouble()
                //Definimos el formato
                val df = DecimalFormat("#.##")
                val result = df.format(value)
                //Mostramos el resultado
                tvHeight.text = result + " cm"
            }
        }
````

## initUI()

Finalmente, tenemos que configurar los elementos visuales. Esto puede ser cambiar el color de un botón, cambiar el texto de un TextView, etc.

````kotlin
    //Configuramos los elementos visuales
    private fun initUI() {

        // Llamamos a los métodos que van a hacer la configuración visual
        setGenderColor()
        this.setAge()

        // Sería posible implementar la lógica directamente, como vemos aquí abajo, aunque no es recomendable
        rsHeight.value = (currentHeight * 100).toFloat()
        tvHeight.text = (currentHeight * 100).toString() + " cm"
    }
````

En realidad, como vemos, el initUI solo debería llamar a otros métodos que inicialicen los elementos visuales. En este caso, setGenderColor() y setAge()...  De hecho, si nos vamos al método setAge, comprobaremos que realmente su lógica podría resolverse inclusod entro del initUI().

````kotlin
    private fun setAge() {
        this.tvAge.text = this.currentAge.toString()
    }
````


## Navegación hacia la siguiente actividad:

Para la navegación, fijaremos un intent de la clase Intent. Este intent llevará como parámetros la activity actual y la activity a la que queremos navegar. 

Sin embargo, **antes de ejecutar** el intent, será necesario añadir todo lo que queramos llevarnos de una pantalla a otra. En este caso, a través del **COMPANION** que definimos al principo -esa será la clave- e irá con un valor. 


````kotlin

   //Funcion para navegar a la siguiente activity
    private fun navigateToResult(resultIMC: Double) {
        //Creamos el objeto intent
        val intent = Intent(this, ResultIMCActivity::class.java)

        //Añadimos el extra necesario para pasar el resultado del IMC
        intent.putExtra(IMC_KEY, resultIMC)

        //Navegamos a la siguiente activity
        this.startActivity(intent)

    }

````

## onResume()

Si en algún momento quisiéramos volver hacia atrás, es posible que quisiéramos recuperar el valor de todos los campos tal y como estaban. Para ello, llamaremos a este método.

````kotlin
    override fun onResume() {
        super.onResume()
        //Recuperamos el valor de la altura
        rsHeight.value = (currentHeight * 100).toFloat()
        tvHeight.text = (currentHeight * 100).toString() + " cm"
    }
````

## Navegación a la siguietne Activity


```kotlin

class ResultIMCActivity : AppCompatActivity() {

    //Creamos las variables privadas con iniciacion tardia para recoger los elementos visuales
    private lateinit var tvResult: TextView
    private lateinit var tvIMC: TextView
    private lateinit var tvDescription: TextView
    private lateinit var  btnReCalculateIMC: Button


    // savedInstanceState:bundle es lo que permite recuperar datos si se gira la pantalla
    // o si la actividad es destruida en un momento. Si es la primera vez que se crea, el valor es nulo.
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_result_imcactivity)
        }

        val resultIMC = intent.extras?.getDouble(IMC_KEY)?: -1.0 //si no obtenemos el extra le damos valor -1

        initComponents()
        initListenners()
        initUI(resultIMC)

    }

```

Aquí no nos detendremos mucho, pero sí que prestaremos atención a lo que sucede dentro del initUI()

## Acceso a la careptas RES de la app

En cualquier momento de la activity, podemos acceder a los valores guardados en REs. Hasta ahora lo hemos hecho con findViewById(R.id.nombreValor). Pero en realidad podemos hacerlo para otros valores. Observemos la función initUi:

```kotlin

 private fun initUI(resultIMC: Double) {
        this.tvIMC.text = resultIMC.toString()
        when(resultIMC){
            in 0.00..18.50 -> { //Bajo Peso
                this.tvResult.setTextColor(ContextCompat.getColor(this,R.color.peso_bajo))
                this.tvResult.text = "Bajo peso"
                this.tvDescription.text = "Tu peso esta por debajo de lo optimo"
            }

            in 18.51..24.99 -> { //Peso Normal
                this.tvResult.setTextColor(ContextCompat.getColor(this,R.color.peso_normal))
                this.tvResult.text = "Normal"
                this.tvDescription.text = "Peso Saludable"
            }

```

Dentro de esta función podemos ver varias cosas:

1. ContextCompat.getColor(this, R.color.peso_bajo) : con esto accedemos a los colores que hemos definido en la carpeta res/values/colors.xml.
2. this.tvResult.setTextColor() Nos va a cambiar el color del texto, ¿a qué color? justamente al que acabamos de recuperar.
3. El resto del when es simplemente para cambiar el texto de la descripción y del resultado.