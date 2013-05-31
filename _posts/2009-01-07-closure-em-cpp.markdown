---
layout: post
title: 'Closure: em C++ (?)'
tags:
- C++
- Java
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
*Esse post é uma continuação da série [Closure](http://www.objectzilla.com.br/2009/01/04/voce-sabe-o-que-e-closure/)*.

C++ tem closure? Como assim? Viajou, né? Bom, vou *tentar* fazer algo, só não sei se vai dar certo. A minha idéia é fazer uma classe **Observer** onde é possível registrar uma closure. Quando for chamado a função **notify()** do objeto dessa classe, a closure será chamada e esta terá inclusive referencia de um contexto que não existe pro objeto Observer.<!--more-->

Primeiro, criei a classe **SimpleObserver** (pois só armazena uma closure por vez e não se aproveita de template). Todo o conteúdo do fonte está em um arquivo hpp (simple_observer.hpp):
{% highlight cpp %}
#ifndef _SIMPLE_OBSERVER_H_
#define _SIMPLE_OBSERVER_H_

#include <boost/function.hpp>

class SimpleObserver {
    
    public:
        /* armazena uma função sem parâmetros que retorna void */
        void registry(const boost::function<void (void)>& cb)     { callback = cb; }
        /* invoca a função previamente armazenada */
        void notify()                                             { if (callback) callback();    }  
        
    private:
        boost::function<void (void)> callback;
};

#endif
{% endhighlight %}

Simples, não? Um método pra armazenar uma função, outro pra chamá-lo. Repare que eu usei uma classe do Boost, **function**, que serve para guardar funções de qualquer tipo. No caso limitei para retorno void e nada de parâmetro de entrada. Outra coisa, pra quem não conhece C++, a expressão **if(callback)** parece estranho. O que ocorre é que nessa linguagem, é possível um objeto ser implicitamente convertido pra outro tipo (contanto que o programador escreva a função de conversão, claro!), e, no nosso caso, a biblioteca converte o objeto function pra um outro do tipo bool. "Mas pra que?", você pensa. Bom, é comum um objeto inválido ser sempre convertido como false, e um válido como true; e nesse caso, o objeto é inválido se o objeto function não apontar pra nenhum ponteiro de função.

Agora, vou mostrar a classe que vai criar uma closure, a **Event**, em dois arquivos, a event.hpp

{% highlight cpp %}
#ifndef _EVENT_H_
#define _EVENT_H_

#include <map>
#include <boost/function.hpp>

class Event {

    public:
        /* construtor que recebe um número como argumento */
        Event(const long& value);
        
        /* retorna uma função, que leva em consideração o valor de entrada
           e a própria instância */
        boost::function<void (void)> create_closure(std::map<std::string, long>& parameter_value);
        
        long instance_value() { return this->inst_value; }
        
    private:
        void callback_function(std::map<std::string, long>* param, long local);
        
    private:
        long inst_value;
};

#endif
{% endhighlight %}

É uma classe que tem um construtor que recebe um valor (daqui a pouco você vai ver que esse valor é atribuído à única variável de instância da classe). A função **create_closure()** retorna uma "function" recebendo um map. A função **instance_value()** simplesmente mostra o valor da variável de instância. Existe uma função privada, a **callback_function** que usarei como ponteiro da minha closure.

Vamos à event.cpp:
{% highlight cpp %}
#include "event.hpp"
#include <boost/bind.hpp>

Event::Event(const long& value)
    : inst_value(value) {}

boost::function<void (void)>
Event::create_closure(std::map<std::string, long>& parameter) {

    long local_value = parameter["value"] + 5;
    
    return boost::bind(&Event::callback_function, this, &parameter, local_value);
    
}

void 
Event::callback_function(std::map<std::string, long>* param, long local) {
    (*param)["value"] += 6;
    this->inst_value += local;
}
{% endhighlight %}

A função create_closure() faz uma soma qualquer, criando uma variável local. Depois é usado o **bind()**, também do Boost, para reduzir uma função com três parâmetros (o "this" mais os dois parâmetros "normais") para uma função com zero parâmetros. Essa função, a **callback_function**, não faz nada mais do que um cálculo.

Pra finalizar, criei o arquivo main.cpp com a função que chama todo mundo:

{% highlight cpp %}
#include "simple_observer.hpp"
#include "event.hpp"

#include <iostream>

using namespace std;

int main(int argc, char** argv) {
    
    SimpleObserver observer;
    
    Event event(10);
    
    std::map<std::string, long> parameter;
    parameter["value"] = 8;
    
    observer.registry( event.create_closure(parameter) );
    
    observer.notify();
    
    cout << "Instance value = " << event.instance_value() << endl;
    cout << "Parameter      = " << parameter["value"] << endl;
    
}
{% endhighlight %}

Eu tenho o Ubuntu, e usei o apt-get pra pegar uma versão da biblioteca Boost e, obviamente, o g++. Como o gerenciador de pacotes vai colocar os HPPs num lugar visível, basta apenas executar o comando:
{% highlight bash %}
 c++ -O3 event.cpp main.cpp -o closure
{% endhighlight %}

Lá na método main, eu chamei tudo mundo, crio o observer, crio o objeto event, e peço a este um closure a ser passado para o observer. Quando o observer for notificado, a instância de event é alterado, assim como o parâmetro passado.

Mas repare uma coisa, eu menti para vocês! C++ não possui suporte a closures! Lá no método create_closure(), tive que passar o "contexto" (parâmetros, variáveis locais, a própria referência this) explicitamente com o "bind". E por não ter garbage collector, não há nem garantia de que os objetos passados com bind realmente exista na sua execução. Portanto, nem tente fazer algo parecido.

Eu comecei com C++ porque é a "pior implementação de closures" das linguagens que eu conheço. Mas não se preocupe, [em um próximo post](http://www.objectzilla.com.br/2009/01/14/closure-em-java/), vou mostrar closure em uma outra linguagem, só que dessa vez, de verdade!
