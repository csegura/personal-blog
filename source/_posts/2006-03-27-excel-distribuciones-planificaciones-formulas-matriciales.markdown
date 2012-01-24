---
date: '2006-03-27 22:21:15'
layout: post
slug: excel-distribuciones-planificaciones-formulas-matriciales
status: publish
title: 'Excel - Distribuciones, planificaciones, formulas matriciales '
wordpress_id: '738'
categories:
- excel
---

No se si lo he comentado alguna vez, pero en el trabajo suelo usar mucho Excel para hacer una gran variedad de cosas, en general mucho de lo que tiene que ver con el área económico financiera (lo terminamos haciendo en Excel).   
  
Muchas de nuestras hojas de cálculo, son planes financieros, estudios de viabilidad, análisis etc. Básicamente estas hojas planifican cobros y pagos ó gastos e ingresos, la cuestión es la planificación, en Excel podemos planificar todo aquello que queramos, es sencillo podemos establecer una fila para cada previsión a realizar y en cada columna podemos poner un periodo (ene, feb, mar…), por último en cada celda (previsión / periodo) el importe de dicha previsión.  



![image](https://lh5.googleusercontent.com/-_QyX7I6Q3wA/Tx5_ezVfMpI/AAAAAAAAAHE/AaZzfV7zjS8/s400/emat1.gif)

El problema se complica un poco cuando el importe de cada previsión no tiene un periodo fijo, sino que este debe establecerse en función de otra variable. Supongamos que tenemos que realizar una previsión entre dos periodos dados (Inicio y Final) y el importe debe ser proporcional al número de periodos.


![image](https://lh5.googleusercontent.com/-431XS42269U/Tx5_ezf3IbI/AAAAAAAAAHI/jgCYaL3z3OM/s400/emat2.gif)

Como es lógico las formulas se nos van complicando, más y más en función de las condiciones que necesitamos. Con lo que terminamos creando nuestras funciones en VBA para simplificar el proceso. 


{% codeblock lang:vbnet %}
Function csPDistB(importe As Integer, periodo As Integer, inicio As Integer, fin As Integer)
  If (periodo >= inicio) And (periodo <= fin) Then
    csPDistB = importe / ((fin – inicio) + 1)
  End If
End Function
{% endcodeblock %}


Realizaría el mismo trabajo que las formulas vistas. Dándole una vuelta más podemos crear una función matricial para hacer la misma tarea y que automáticamente tome el periodo actual en función del rango en donde se encuentre.

{% codeblock lang:vbnet %}
Public Function csPDistB(importe As Variant, primero As Integer, ultimo As Integer) As Variant
  Dim i As Integer
  Dim nPeriodos As Integer
  Dim valor As Double

  ReDim a(0 To Application.Caller.Rows.Count, 0 To Application.Caller.Columns.Count) As Variant

  On Error GoTo Handler

  nPeriodos = ultimo – primero
  valor = CDbl(importe / (nPeriodos + 1))

  For i = 0 To Application.Caller.Columns.Count
      If i + 1 >= primero And i + 1 <= ultimo Then
         a(0, i) = valor
      End If
  Next

  csPDistB = a

  Exit Function
Handler:
  csPDistB = CVErr(2015)  ‘xlErrNum = 2036
End Function
{% endcodeblock %}


![image](https://lh3.googleusercontent.com/-1Pv7rPLx7xw/Tx5_e7OZKXI/AAAAAAAAAHM/PUnEmaK9XVo/s800/emat3.gif)


Por último con una pequeñas modificaciones sobre este código podemos crear funciones más complejas para nuestras planificaciones, por ejemplo distribuciones en función de una curva de porcentajes, 25%, 50% y 25% sería el 25% en el primer tercio, el 50% en el segundo tercio y el 25% en el tercer tercio del tiempo. 



{% codeblock lang:vbnet %}
Public Function csPDistCP(importe As Variant, ParamArray porcentajes()) As Variant
  Dim i As Integer
  Dim nParte As Integer
  Dim p As Integer

  ReDim a(Application.Caller.Columns.Count) As Variant

  On Error GoTo Handler

  nParte = (UBound(a) + 1) / (UBound(porcentajes) + 1)

  p = -1

  For i = 0 To UBound(a)
    If i Mod Int(nParte) = 0 Then
       If p < UBound(porcentajes) Then
          p = p + 1
       End If
    End If

    a(i) = CDbl((importe / nParte) * porcentajes(p))

  Next

  csPDistCP = a

  Exit Function

Handler:

  csPDistCP = CVErr(2015)  ‘xlErrNum = 2036

End Function
{% endcodeblock %}


Un último ejemplo en donde realizamos previsiones los periodos indicados, el importe proporcional al número de periodos.

{% codeblock lang:vbnet %}
Public Function csPDistP(importe As Variant, ParamArray periodos()) As Variant
  Dim i As Integer
  Dim nPeriodos As Integer
  Dim valor As Double

  ReDim a(Application.Caller.Columns.Count) As Variant

  On Error GoTo Handler

  nPeriodos = UBound(periodos)

  valor = CDbl(importe / (nPeriodos + 1))

  For i = 0 To nPeriodos
      a(periodos(i)) = valor
  Next

  csPDistP = a

  Exit Function

Handler:

  csPDistP = CVErr(2015)  ‘xlErrNum = 2036

End Function
{% endcodeblock %}
