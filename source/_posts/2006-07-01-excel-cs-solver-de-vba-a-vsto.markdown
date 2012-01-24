---
date: '2006-07-01 17:29:27'
layout: post
slug: excel-cs-solver-de-vba-a-vsto
status: publish
title: Excel - CS-Solver de VBA a VSTO
wordpress_id: '712'
categories:
- excel
- office
---

Hace ya algún tiempo que escribí **CS-Solver**, estos días me he propuesto convertirlo de VBA a VSTO. 




**CS-Solver** es un add-in para Excel que escribí hace algún tiempo, ya que la calculadora que usaba tenía un sistema de resolver ecuaciones muy parecido ([**PowerOne finance**](http://www.infinitysw.com/products/poweronefinance.html)) y sin embargo resolver ecuaciones con Excel era bastante más complicado.




**CS-Solver** utiliza el método secante para encontrar las raíces de **f(x)=0** en un intervalo, este no es tan rápido como el de Newton, pero si más rápido que la bisección ya que acota el intervalo y usa la aproximación más reciente. 




A diferencia del método de Newton, no necesitamos calcular la derivada de la función en cada iteración, y que es un proceso más tedioso y requiere cálculo adicional.




Veamos ahora como funciona **CS-Solver** con un pequeño ejemplo, resolviendo la siguiente ecuación   
**2^x + 5x = 2**

![image](https://lh4.googleusercontent.com/-QfldrCzMxmU/Tx5jCFdNSdI/AAAAAAAAAFc/OGLUYKlmZcg/s288/css002.jpg)

Primero crearemos una celda **C2 **a la que daremos un nombre en este caso **X**, esta es la incógnita, en otra celda teclearemos la función, igualando el resultado a **0**, quedando la ecuación **2^x+5x-2=0**, esta celda la introduciremos como texto.

Ahora desde el menú de **CSSolver**, seleccionamos resolver ecuación 

En donde se nos pide que introduzcamos la celda que contiene la ecuación a resolver, una vez seleccionada la celda que contiene la ecuación C4 pulsamos sobre Aceptar 


![image](https://lh3.googleusercontent.com/-y5m_OFVbh6s/Tx5jCox6sjI/AAAAAAAAAFk/k3NA3E2Rh9M/s288/css004.jpg)

El resultado aparece en la celda de nombre **X **que es la incógnita a resolver.


![image](https://lh5.googleusercontent.com/-PcPRzGEIBqQ/Tx5jDC4y8mI/AAAAAAAAAFw/zupkgAHJBbE/s288/css006.jpg)


Veamos ahora otras cosas que se puede hacer con **CS-Solver**. Para ello usaremos la formula del Interés compuesto y calcularemos el Valor Futuro de un Capital invertido al 5% de interés anual durante **3 **meses.  
****

**Valor Final = Valor Actual * ( 1 + (Interés/12))^Meses**

![image](https://lh4.googleusercontent.com/-RUuLpI26qfM/Tx5jDYxcipI/AAAAAAAAAF0/sQ24BKQ-OeA/s400/css008.jpg)

Teclearemos la formula igualada a 0, **0=(vactual*(1+interes))^meses)-vfinal** para ello, nombraremos la celda **C2** como **vfinal**, **C3** como **vactual**, **C4 **como **interes** y **C5** como **meses**

Si ejecutamos **CS-Solver** y seleccionamos **B7** como la formula a resolver obtendremos el **Valor Actual**, solucionando la ecuación.

![image](https://lh3.googleusercontent.com/-q9eoJIfPHNI/Tx5jD5b9-KI/AAAAAAAAAF8/O65iLsj7T6c/s400/css010.jpg)


Bien, una vez obtenido el resultado deseamos vemos que **1012,55** no es lo esperado que necesitamos obtener al final de nuestra inversión un total de **1125**, así que queremos ver cuanto necesitamos ingresar ahora para poder retirar dentro de 3 meses **1125**

Para ello, nos basta con eliminar el contenido de la celda **C3**, (**vactual**) e indicar en la celda **C2 **el importe que deseamos obtener. Nuestra hoja de excel quedaría así.

![image](https://lh6.googleusercontent.com/-iFxRtl3rCJM/Tx5jEAmbvxI/AAAAAAAAAGA/DQag_6CYMzg/s400/css012.jpg)

Bien, no tenemos que tocar nada más tan solo seleccionar **CS-So**lver e indicar la formula que ya habíamos escrito, **B7**

![image](https://lh3.googleusercontent.com/-GNhduDPB4Ac/Tx5jEuiGGQI/AAAAAAAAAGM/BfZdjM4n-0k/s400/css014.jpg)

**CS-Solver** comprueba cual es la incógnita de la ecuación, vactual en este caso y comprueba que no tiene valor, así que su objetivo es resolver dicha incógnita.


**CS-Solver** incorpora también una nueva formula llamada **CSSolver** con la cual podemos tener en una celda el valor de la incógnita de la ecuación, de este modo se puede tener en una celda **=CSSolver(“2*x*sin(x)-5”)** para la ecuación **2*x*sin(x)=5** e incluso podemos vincular la formula con una celda **=CSSolver(A5)**

