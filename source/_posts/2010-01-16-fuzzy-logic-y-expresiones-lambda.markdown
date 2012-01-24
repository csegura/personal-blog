---
date: '2010-01-16 01:33:00'
layout: post
slug: fuzzy-logic-y-expresiones-lambda
status: publish
title: Fuzzy Logic y expresiones lambda
categories:
- net
---

Para un proyecto que estoy realizando, he realizado una prueba de concepto probado a aplicar lógica difusa con el fin de realizar ciertas acciones a partir de una serie de datos imprecisos.
  
A la hora de expresar las reglas encargadas de modelar nuestro sistema de lógica difusa, (IF-THEN) la mayoría de los sistemas que he visto requerían de un pequeño y simple análizador sintáctico para interpretar las reglas, en este punto me he preguntado si en vez de un análizador sintáctico, podríamos usar expresiones lambda y una interfaz fluida.  
  
**FuzzySystem (improvisado)**  
  
De modo que he construido un pequeño motor de lógica difusa que tiene la particularidad de usar expresiones lambda y una interfaz fluida para definir el conjunto de reglas.  
Este pequeño motor incorpora un par de funciones miembro, una de tipo Trapezoide y otra Triangular, básicamente su uso es el siguiente.  
Definimos el sistema, añadimos las variables con sus estados y funciones miembro…


{% codeblock lang:csharp %}
public class FuzzyTest
{
    private readonly FuzzySystem _engine = new FuzzySystem();

    public FuzzyTest()
    {
        FuzzyVariable varA = new FuzzyVariable("A");
        varA.Memberships.Add(new TrapezoidMembership("Cerrado", 0, 0, 5, 5));
        varA.Memberships.Add(new TrapezoidMembership("Medio", 4, 4, 6, 6));
        varA.Memberships.Add(new TrapezoidMembership("Abierto", 7, 7, 9, 9));
        _engine.Variables.Add(varA);

        FuzzyVariable varB = new FuzzyVariable("B");
        varB.Memberships.Add(new TrapezoidMembership("Cerrado", 0, 0, 5, 5));
        varB.Memberships.Add(new TrapezoidMembership("Medio", 4, 4, 6, 6));
        varB.Memberships.Add(new TrapezoidMembership("Abierto", 7, 7, 9, 9));
        _engine.Variables.Add(varB);

        FuzzyVariable varR = new FuzzyVariable("R");
        varR.Memberships.Add(new TrapezoidMembership("Frio", 0, 0, 3, 3));
        varR.Memberships.Add(new TrapezoidMembership("Templado", 3, 3, 6, 6));
        varR.Memberships.Add(new TrapezoidMembership("Caliente", 7, 7, 9, 9));
        _engine.Variables.Add(varR);
{% endcodeblock %}

**La definición de reglas mediante expresiones Lambda**  
  
El beneficio en este punto es que el sistema no requiere del análizador sintáctico para evaluar las expresiones si no que será el CLR quien se encargue de ello.  
Para llevar a cabo esta evaluación de las reglas he definido una clase _FuzzyExpresion _que tiene definidos los operadores && y || para así como el true y el false para poder evaluar las reglas en consecuencia.


{% codeblock lang:csharp %}
public class FuzzyExpression
{
    private readonly FuzzyVariable _variable;
    private string _state;
    private double? _value;

    public double Value
    {
        get
        {
            if (_value.HasValue)
                return _value.Value;
            _value = _variable.Fuzzify(State);
            return _value.Value;
        }
    }

    public string State
    {
        get { return _state; }
    }
  
    public FuzzyExpression(FuzzySystem engine, string literal)
    {
        _variable = engine.Variables[literal];
        
        if (_variable == null)
        {
            throw new FuzzyException("variable not found.");
        }
    }

    public FuzzyExpression(double value)
    {
        _value = value;
    }

    public FuzzyExpression Is(string state)
    {
        _state = state;
        _value = _variable.Fuzzify(_state);
        return this;
    }

    public FuzzyExpression Set(string newState)
    {
        _state = newState;
        return this;
    }

    public static FuzzyExpression operator |(FuzzyExpression a, FuzzyExpression b)
    {
        return new FuzzyExpression(Math.Max(a.Value, b.Value));
    }

    public static FuzzyExpression operator &(FuzzyExpression a, FuzzyExpression b)
    {
        return new FuzzyExpression(Math.Min(a.Value, b.Value));
    }

    public static bool operator false(FuzzyExpression a)
    {
        return 0 > a.Value;
    }

    public static bool operator true(FuzzyExpression a)
    {
        return a.Value > 0;
    }
}
{% endcodeblock %}


Las reglas en la lógica difusa sirven para combinar las distintas proposiciones, no son reglas excesivamente complicadas, y son del tipo IF-THEN, básicamente hay 4 reglas (Implicación Conjunción, Disyunción y negación) que se corresponden con:


- Implicación IF variable==estado THEN variable = estado  
- Conjunción, equivalente al AND, si dos proposiciones son ciertas simultáneamente   
- Disyunción, cualquiera de las dos proposiciones es cierta OR  
- Negación, invierte la proposición


Finalmente podemos definir las reglas de este modo


{% codeblock lang:csharp %}
_engine.AddRule(x =>
                x.Variable("A").Is("Abierto") &&
                x.Variable("B").Is("Abierto"))
                .Then(x => x.Variable("R").Set("Caliente"));

_engine.AddRule(x =>
                x.Variable("A").Is("Abierto") &&
                (x.Variable("B").Is("Cerrado") || x.Variable("B").Is("Medio")))
                .Then(x => x.Variable("R").Set("Templado"));

_engine.AddRule(x =>
                x.Variable("B").Is("Abierto") &&
                (x.Variable("A").Is("Cerrado") || x.Variable("A").Is("Medio")))
                .Then(x => x.Variable("R").Set("Templado"));

_engine.AddRule(x =>
                x.Variable("A").Is("Cerrado") &&
                x.Variable("B").Is("Cerrado"))
                .Then(x => x.Variable("R").Set("Frio")); 
{% endcodeblock %}


**¿Como funciona?**


Una regla tiene una condición (IF) y una consecuencia (THEN), la condición es una función que recibe el sistema y debe devolver una FuzzyExpresion, (_Func<FuzzySystem,FuzzyExpression>_), esto es parte de la magia ya que una expresión condicional es siempre reducida a una única expresión y finalmente a un único valor.  
Y curiosamente la consecuencia de la regla, es exactamente igual, solo que en este caso la expresión no devuelve nada porque la consecuencia es una acción, aqui hay otro pequeño truco que es que la propia regla contiene los dos elementos separados (condición y consecuencia) fijaros en que:


{% codeblock lang:csharp %}
_engine.AddRule(x =>
	x.Variable("A").Is("Abierto") &&
    x.Variable("B").Is("Abierto"))
{% endcodeblock %}


Esta devolviendo una regla y el método _Then_ es aplicado sobre la regla, y cuando se evalúa la regla si la condición es cierta se evalua la consecuencia, que lo que hace es únicamente cambiar el estado (ver el método _Set _de _FuzzyExpression_)  


{% codeblock lang:csharp %}    
public class FuzzyRule
{
   private readonly FuzzySystem _engine;
   private readonly Func _condition;
   private Func _then;


   public double Value { get; set; }

   public FuzzySystem Engine
   {
       get { return _engine; }
   }

   public FuzzyRule(FuzzySystem engine, Func condition)
   {
       _engine = engine;
       _condition = condition;
   }

   public FuzzyExpression Eval()
   {
       Value = _condition(Engine).Value;

       if (Value > 0)
       {
           return _then(Engine);
       }

       return null;
   }

   public void Then(Func then)
   {
       _then = then;
   }     
}
{% endcodeblock %}
  
**Defuzzyficando**


Para evaluar nuestro sistema ..

{% codeblock lang:csharp %}
        for (int a = 0; a < 10; a++)
    {
        for (int b = 0; b < 10; b++)
        {
            _engine.Variables["A"].InputValue = a;
            _engine.Variables["B"].InputValue = b;
            _engine.Consecuent = "R";

            var r = _engine.Defuzzy();

            Console.WriteLine(string.Format("A {0} - B {1} = {2,-6} [{3,-10}][{4,-10}] = {5}", 
                a, 
                b, 
                r, 
                _engine.GetVariableState("A",a),
                _engine.GetVariableState("B",b),
                _engine.GetVariableState("R", r)));
        }
    }
}
{% endcodeblock %}


**Mejoras de diseño **

Hay una cosa que no me gusta mucho y es el hecho de que mi clase FuzzyExpression, tiene tanto el método Is como el método Set y esto habría que dividirlo en dos para no llevar a errores a la hora de la usar la interfaz fluida.

**Errores de diseño **

Pero no todo es de color de rosa, cuando planifique el sistema, pensé en la posibilidad de guardar las expresiones lambda como texto y después compilarlas usando CodeDom, bien tengo que deciros que cuando me he puesto a ello me he llevado la desagradable sorpresa de que CodeDom no soporta expresiones lambda, de modo que mi gozo en un pozo.

Y digo esto porque es de vital importancia el poder mantener las reglas fuera del código, bien un archivo de configuración, un archivo XML, lo que sea… de este modo podemos realizar ajustes sin tener que recompilar todo …   
****

**Para una futura version **

Completar estas cositas …



