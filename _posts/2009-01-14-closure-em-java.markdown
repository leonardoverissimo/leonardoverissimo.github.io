---
layout: post
title: 'Closure: em Java (?)'
tags:
- Sem categoria
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
_Esse post é uma continuação da série [Closure](http://www.objectzilla.com.br/2009/01/04/voce-sabe-o-que-e-closure/), que também já falou de [C++](http://www.objectzilla.com.br/2009/01/07/closure-em-cpp/)._

Java tem closure? Como assim? Bom, a linguagem tem mais alguns truques que dá pra se aproximar melhor de closure do que C++, mas ainda é bem distante de linguagens onde closures é uma coisa natural (tanto é que os javeiros mal usam essa sintaxe). Enfim, e ainda por cima, precisarei utilizar a principal muleta das linguagens chinfrins: **design patterns**.
<!--more-->
Vamos lá? A idéia é a mesma, tenho um Observer (minha classe **SimpleObserver**, que não tem suporte a múltiplas instâncias de "observados", nem usa generics) que é assim:

{% highlight java %}
package closure;

public class SimpleObserver {
    public void registry(FunctionObject callback) {
        this.callback = callback;
    }
    
    public void warn() {
        if (callback != null) {
            callback.call();
        }
    }
    
    private FunctionObject callback;
}
{% endhighlight %}

Bom, eu tenho um método **registry()** que vai receber um objeto do tipo **FunctionObject** (ainda não apareceu, vou mostrar daqui a pouco). Você lembra do pattern Command? Então, estou utilizando aqui.

O outro método é **warn()**. O exemplo em C++ (e nos próximos que virão) está como notify(), mas não posso usar essa assinatura em Java porque Object já tem um método com esse nome e é reservado para uso de sincronização de threads. Enfim, se registry() é responsável por receber um **FunctionObject**, o método warn() irá executar o objeto do tipo FunctionObject previamente recebido.

Mas você deve estar se perguntando: quem é FunctionObject? Apenas uma interface! Para implementarmos o pattern Command:

{% highlight java %}
package closure;

public interface FunctionObject {
    
    void call();
}
{% endhighlight %}

Ou seja, a o objeto SimpleObserver está pouco se lixando para a classe do objeto recebido, o importante é que este implemente **FunctionObject**, que só tem um método: call().

Agora, a classe que cria a closure, como em C++, chama-se **Event**:

{% highlight java %}
package closure;

import java.util.Map;

public class Event {
    public Event(long value) {
        this.instanceValue = value;
    }
    
    public FunctionObject createClosure(final Map<String, Long> param) {
        final long localValue = param.get("value") + 5;
        
        return new FunctionObject() {
            public void call() {
                param.put("value", param.get("value") + 6);
                instanceValue += localValue;
            }
        };
    }
    
    public long getInstanceValue() {
        return instanceValue;
    }
    
    private long instanceValue;
}
{% endhighlight %}

É parecido com o exemplo do C++, porém beeem mais simples. Agora, o método **createClosure()** vai retornar o nosso command: o objeto que implementa FunctionObject. O truque: estou usando inner class, ou seja, uma classe dentro da outra, pois essa é a única maneira em Java de uma classe ter referência do contexto mais externo. É esquisito, eu declaro um método, que dentro dela declara uma classe anônima (implementando FunctionObject), que dentro dela declara um outro método. Neste, estou usando referências que estão do lado de fora, como _param_, _localValue_ e _instanceValue_.

Porém um adendo, reparou que eu estou usando referências **final**? Por que isso? Bem, closures em Java não é perfeito. O que a classe anônima faz é copiar todas as referências externas e tomar para si próprio. Agora pense, eu tenho duas referências apontando para o objeto de tipo Map, uma quem "segura" é o objeto anônimo, outro é o objeto Event. Por ter duas referências, eu não posso mudá-las! Porque senão um objeto vai apontar para um valor e o outro, para o outro! Preciso impedir isso, colocando final. Aliás, o compilador vai encher o saco se eu não colocar, então não teria alternativa mesmo.

O restante dos métodos é um construtor que recebe um valor e um método que verifica esse valor, nada de especial.

Chegou a hora de bater tudo isso no liquidificador e ver como é que fica num main, assim:

{% highlight java %}
package closure;

import java.util.Map;
import java.util.TreeMap;

public class Main {
    public static void main(String[] args) {
        SimpleObserver observer = new SimpleObserver();
        
        Event event = new Event(10);
        
        Map<String, Long> parameter = new TreeMap<String, Long>();
        parameter.put("value", 8L);
        
        observer.registry( event.createClosure(parameter) );
        
        observer.warn();
        
        System.out.println("Instance value = " + event.getInstanceValue());
        System.out.println("Parameter      = " + parameter.get("value"));
    }
}
{% endhighlight %}

A rotina é bem boba. Crio o objeto evento. Depois, peço a este uma closure que será registrado no objeto observer. Quando dou um "warn" no observer, este ativa a closure, disparando alterações de maneira invisível no conteúdo do Map parameter e da variável de instância do objeto evento.

É isso, eu comecei com C++, melhorei um pouco com Java, e vou melhorar um pouco mais nos próximos posts. Até.
