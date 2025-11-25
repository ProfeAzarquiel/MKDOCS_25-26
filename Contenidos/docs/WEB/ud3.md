# PREPROCESADORES CSS

## INTRODUCCIÓN A SASS

### ENTORNO DE TRABAJO

**VSCODE**: Plugins Sass y Scss Intellisense.
**NODE.JS**: Para usar su gestor de paquetes NPM.
**SASS**: Para compilar nuestros ficheros SCSS a CSS.

### ¿QUE ES SASS?

Recomendaciones de uso y sentido común: [Enlace](https://sass-guidelin.es)

Página de documentación oficial SASS: [Enlace](https://sass-lang.com/documentation)

### INSTALACIÓN SASS

Previa instalación de Node.js, usamos su gestor de paquetes para ejecutar:

    npm install -g sass
    sass --version

### ELEMENTOS BÁSICOS DE SASS

#### **Variables**

Podemos definir variables donde almacenar valores, que usaremos posteriormente. Por ejemplo:

    $color-fondo:white;
    $color-texto: grey;

Tipos de datos básicos para una variable (Valores):

> - Números: 48 48px 0.5em
> - Cadenas: ‘left’ “left” left
> - Colores: rg(255, 0, 0) hsl(0, 100%, 50%), #f00 #ff0000
> - Booleanos: (true y false)
> - null
> - Mapas y Listas

Las variables pueden ser **globales** (fuera de cualquier bloque) o **locales** (definidas dentro de un bloque, por ejemplo dentro de un elemento *"section"*).

#### **Comentarios**

> - **Unilínea**: //Esto es una sola línea
> - **Multilínea**: /*Esto es multilinea*/

#### **Listas y mapas**

Las **listas** son colecciones de valores separados por comas. Similares a un array de una dimensión:

    $lista-banderas: (espana, francia, italia, alemania);

Mientras que los **mapas** son colecciones de valores del tipo "clave:valor", separados por comas:

    $mapa-paises: (
        espana: "espana.png",
        francia: "francia.png",
        italia: "italia.png",
        alemania: "alemania.png"
        );

#### **Interpolación**

#### **Anidamiento**

## COMPILACIÓN SASS A CSS

Para compilar nuestro archivo .SCSS en un archivo .CSS, el cual podamos enlazar desde una página HTML; usaremos el siguiente comando:

    sass input.scss output.css

Ese comando admite diversas opciones de personalización, una de las más comunes que usaremos, sirve para que la herramienta se quede a la espera de cambios y autocompile nuestro SCSS cada vez que lo guardemos:

    sass --watch input.scss output.css

## ESTRUCTURAS DE CONTROL EN SASS

### Condicional **`@if / @else`**

    nav{
        @if $light-theme == true{
            background-color: $navbar-background-color-light;
        } @else if $dark-theme{
            background-color: $navbar-background-color-dark;
        } @else{
            background-color: $navbar-background-color;
        }
    }

### Bucle **`@while`**

    @while $num < 6{
        figure:nth-child(#{$num}){
            background-color: nth($color-list, $num);
        }
        $num: $num + 1;
    }

### Bucle **`@for`**

    @for $i from 1 to 5{
        li:nth-of-type(#{$i}){
            color: nth($color-list2, $i);
        }
    }

### Bucle **`@each`**

    @each $c, $v in $mapa{
        div.#{$c}{
            background-image: url("img/#{$v}");
            background-repeat: no-repeat;
            background-size: contain;
        }
    }

## FUNCIONES EN SASS

Nativas:

Creadas por el usuario:

    @function anchura_col($cols, $total){
        @return percentage($cols/$total);
    }

## REGLAS Y DIRECTIVAS SASS

### @import

Permite construir una única hoja de estilos que sea la combinación de distintos archivos CSS o SCSS, denominados *"partials"*.

Los figheros partials, deberán nombrarse comenzando por guión bajo, por ejemplo, *"_colores.scss"* o *"_botones.scss"*

A la hora de compilarlos en un único CSS, no es necesario indicar ese guió bajo, por ejemplo:

    @import "colores.scss";
    @import "botones.scss";

### @extend

Nos permite construir los estilos de un elemento, usando como base los estilos de otro. Por ejemplo:

    .btn {
    border-radius: 2px;
    color: #FFF;
    padding: 5px 0;
    margin: 2px auto;
    text-align: left;
    width: 150px;
    }

    .btn-error {
        @extend .btn;  
        background-color: #F00;  
    }

    .btn-ok {
        @extend .btn;
        background-color: #0F0;
    }

Puede ser que queramos que el selector usado para el @extend, no aparezca en el CSS resultante de la compilación. Para ello podemos usar un selector **placeholder**, añadiendo el carácter % delante del nombre del selector. Es decir, en el ejemplo anterior, sustituirimos **".btn"** por **"%btn"**.

### @error / @warn / @debug

Son directivas útiles cuando estamos depurando nuestro SCSS. Nos permiten mostrar mensajes y valores de variables, funciones etc..durante el proceso de generación del CSS.

    $test: false;
    body {
        @if $test {
            @error "Mensaje de error";
            @debug "Test tiene el siguiente valor: #{$test}";
        }
        else {
            @warn "Mensaje de warning";
            @debug "Test tiene el siguiente valor: #{$test}";
        }
    }

### @at-root

La directiva @at-root permite que selectores dentro de reglas con anidamiento sean movidos a la raíz del documento. Su uso no es algo común y se puede llegar a usar con anidamientos avanzados que usan funciones de selección y selectores referentes a elementos padre.

Por ejemplo, si tenemos el siguiente código SCSS:

    .item {
        color: black;    
        @at-root #{&}is-active {
        color: blue;
        }
    }

Generará el siguiente CSS:

    .item {
    color: black;
    }
    .item.is-active {
    color: blue;
    }

### @media / @support / @keyframes

Sass también soporta las directivas `@media`, `@support` y `@keyframes` que son directivas propias de CSS y su uso es igual.

## MIXINS SASS

Nos permite definir estilos que luego voy a poder reutilizar a lo largo de mi hoja de estilos. Por ejemplo:

    @mixin centrado {
        margin: 0px auto;
    }

    header {
        @include centrado;
        background-color: black;
        color: white;
    }

Como vemos, para usar un mixim debemos usar la directiva **@include**.

### Principales usos

Los mixins, se usan generalmente en Media Queries, Prefijos relativos a los navegadores, Transformaciones CSS, Animaciones CSS, Colocación fija de elementos o Gradientes.

En el repositorio [SCSS-MIXINS-COLLECTION](https://github.com/7ninjas/scss-mixins/tree/develop) se puede ver una colección de mixins, agrupados por temática.

Por ejemplo, podemos crear mixins que incluyan selectores completos de tal manera que se pueda usar en cualquier parte de mi SCSS:

    @mixin highlighted-link {
        a {
            background-color: yellow;
            font-style: italic;
            text-decoration: none;
        }
    }

    @include highlighted-link;

Tambien puedo usar mixins dentro de otro mixin.

    @mixin centrado_menu {
        @include centrado;

        background-color: #666;
        color: white;
        height: 3rem;
    }

    .main_menu {
        @include centrado_menu();
    }

### Parámetros

También tenemos la posibilidad de que nuestros mixins reciban una serie de parámetros, que serán obligatorios u opcionales. En caso de ser opcionales tenemos que asignarles un valor por defecto en la definición del parámetro.

Un ejemplo de parametros obligatorios:

    @mixin girar($grados) {
    -webkit-transfrom: rotate(#{gradosdeg}deg);
    -ms-transform: rotate(#{gradosdeg}deg);
    transform: rotate(#{gradosdeg}deg);
    }

Otro ejemplo que incluye parámetros opcionales:

    @mixin logo($image, $radio: 10px) {
    background-image: url("#{$image}");
    background-position: center;
    border-radius: $radio;
    }

A la hora de usarlos, podemos especificar o no un valor para los paremetros opcionales:

    .logo-cuadrado {
    @include logo("img/milogocuadrado.png", 0px);
    }

## DOCUMENTACIÓN EN SASS

### ¿Qué es SassDoc?

Sistema de documentación que permite generar de manera automática, documentación de tipo profesional en HTML a partir de los comentarios de nuestros ficheros Sass.

### Instalación

Para instalarlos, debemos partir de un entorno en el que ya contemos con Nodejs, Npm y Npx (de no tener alguno de ellos, deberemos instalarlo); podemos comprobarlo con:

    node --version
    npm --version
    npx --version

Una vez comprobado esto, podemos instalar SassDoc con el siguiente comando:

    npm install sassdoc -g
    sassdoc --version

### Uso básico

Para una documentación aceptable, como mínimo, debemos comentar las **variables, las funciones y los mixins**.

Dentro de los comentarios SassDoc, existen una serie de palabras reservadas o anotaciones, que nos permitirán darle más sentido y mejor compresión a la documentación generada.

Podemos ver el detalle de cada una de ellas en la [Documentación Oficial](http://sassdoc.com/annotations/), pero la lista completa sería:

    @access     @ignore     @return
    @alias      @link       @see
    @author     @name       @since
    @content    @output     @content
    @deprecated @parameter  @throw
    @example    @property   @todo
    @group      @require    @type

Los comentarios SassDoc, comenzarán por **triple barra** y se colocarán encima de los elementos a comentar:

    /// Color de fondo de la web
    ///@group colores
    $background-color-principal: #white;

También podemos especificar al comienzo del archivo comentarios con **cuadruple barra**, los cuales se incluirán en todas las secciones de comentarios que definamos en ese fichero:

    //// @access public
    //// @author IES Azarquiel

Otro ejemplo más comlpeto, podría ser el siguiente:

    /// @group colores
    /// @parameter {color} $color - Valor del color que quiero aclarar
    /// @parameter {floar} $transparencia - Valor de la transparencia a aplicar de -1 a 1.
    /// @return {color}
    /// @example scss - Funcion Aclarar
    /// bacground-color: aclarar(#00AA00, -0.2)
    @function aclarar($color, $transparencia){
        @return adjust-color($color, $alpha: $transparencia);
    }

Una vez tengamos todos los comentarios que queramos añadir, ejecutamos el comando:

    sassdoc scss/

Esto creará una carpeta **sassdoc** dentro del directorio "scss", con toda la documentación generada y cuyo punto de entrada es el fichero `index.html`.

## INTRODUCCIÓN A GULP
