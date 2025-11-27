# CREACIÓN DE API REST EN PYTHON

## ¿QUÉ ES UNA API?

Una API (Interfaz de Programación de Aplicaciones) es un conjunto de reglas y protocolos que permiten el intercambio de datos entre dos sistemas informáticos, de una manera segura y estandarizada.

Es decir, actua como un intermediario que define la forma en que una aplicación puede solicitar, recibir y enviar información a otra.

Su principal utilidad es simplificar el desarrollo de aplicaciones, ya que permiten reutilizar código e integrar servicios existentes con facilidad, sin necesidad de crear todo desde cero.

![Flujo API](../img/apiflow.png)

La principal forma de comunicarnos con una API es mediante un navegador web, usando el protocolo HTTP. Es decir, introduciremos una dirección URL que nos permitirá acceder a los recursos que proporciona la API. Las partes que diferenciamos en esta dirección son:

* Protocolo: http:// o https://
* Dominio: nombre del host en el que está alojada la API
* Endpoint: ubicación lógica específica a la que se envía la solicitud.

Los diferentes métodos utilizados para interactuar con una API son:

* **GET**: para obtener información.
* **POST**: para enviar información.
* **PUT**: para actualizar información.
* **DELETE**: para eliminar información.

## PRINCIPALES FRAMEWORKS PARA APIs

Al crear aplicaciones web en Python, hay tres opciones de frameworks que destacan sobre el resto: **Django**, **Flask** y **FastAPI**. Cada uno aporta fortalezas, limitaciones y contextos de uso distintos, como vemos en la siguiente tabla:

<figure markdown="span">
  ![Frameworks](https://miro.medium.com/v2/resize:fit:720/format:webp/1*LOZgruzs6i82_D2oMYG4bw.png){ width="500" }
  <figcaption>Frameworks Python</figcaption>
</figure>

<!-- ![Frameworks Python](https://miro.medium.com/v2/resize:fit:720/format:webp/1*LOZgruzs6i82_D2oMYG4bw.png){width="500"}-->

| CARACTERÍSTICAS     | DJANGO | FLASK | FASTAPI|
| -------------------:| :------| :-----| :------|
| **Definición**      | Framework web de alto nivel y full stack, que viene con todas las funciones y componentes necesarios para un uso completo desde el principio. | Microframework ligero y minimalista, de facil aprendizaje y que apuesta por la felxibilidad. | Framework moderno y de alto rendimiento pensado para construir APIs con validación, y generación de documentación automática. |
| **Uso Recomenado**      |  Aplicaciones web grandes con base de datos, panel de administración y varias funcionalidades listas para usar, como un e-commerce, CMS o ERP. | APIs pequeñas o medianas, prototipos y proyectos de aprendizaje donde se busque libertad tecnológica. | REST APIs y microservicios, aplicaciones en tiempo real como chat o IoT y despliegue de modelos de machine learning e IA cuando el rendimiento es crítico. |
| **Desventajas**     |  Es más pesado y menos flexible al imponer convenciones y no tan rápido cuando se trata de APIs de alto rendimiento. | Al ser tan ligero, suele ser necesario instalar librerias externas para integrar ciertas funcionalidades, lo que en proyectos grandes puede ser algo tedioso. | Al ser más joven que los dos anteriores, puede que su comunidad no sea tan extensa y no está orientado a aplicaciones monolíticas. |

*[Fuente: Q2B](https://www.q2bstudio.com/nuestro-blog/18435/django-flask-y-fastapi-cual-elegir)*

### **FastAPI**

Es importante mencionar que FastAPI solo puede usarse con Python 3.6 o superior. Para utilizarlo debemos instalar dos librerías: fastapi y uvicorn.

**Uvicorn** es una librería de Python que actúa como servidor web asíncorno (ASGI) de alto rendimiento y se utiliza para ejecutar aplicaciones web y APIs, como las creadas con FastAPI. Se usa **desde la línea de comandos** para levantar el servidor, por ejemplo, si tu archivo se llama *"main.py"* y tu aplicación se llama *"app"*, ejecutaríamos: `uvicorn main:app --reload`.

    pip install fastapi
    pip install uvicorn

#### Mi primera API con FastAPI

    from fastapi import FastAPI
    
    app = FastAPI()
    
    @app.get("/mi-primer-api")
    def hello():
        return {"Hello world from FastAPI!"}

<!-- Se pone un signo de interrogación (?) en una llamada a una API para indicar el comienzo de los parámetros de consulta, que son datos adicionales que se envían al servidor para especificar la solicitud. Estos parámetros le dicen a la API qué información específica obtener o qué acción realizar, funcionando como una extensión de la URL.

Función: Separar la ruta principal del endpoint de los datos adicionales (parámetros) que se están enviando.
Estructura: Después del signo de interrogación (\(?\)), se incluyen pares clave-valor (por ejemplo, $clave=valor$).
Separador de parámetros: Si hay múltiples parámetros, se separan entre sí con un símbolo de y comercial (&).
Ejemplo: En una URL como $https://ejemplo.com/api/usuarios?id=123&estado=activo$, el signo de interrogación marca el inicio de los parámetros $id=123$ y $estado=activo$, que especifican que se busca el usuario con el ID 123 y cuyo estado es activo. 
-->

*[Fuente: Ander Fernandez](https://anderfernandez.com/blog/como-crear-api-en-python/)*
