---
layout: post
title: Nem tudo em Java são flores
tags:
- C++
- Java
status: publish
type: post
published: true
meta:
  _edit_last: '1790462'
---

Tem gente que é muito fã da linguagem Java. Tem gente que além de ser muito fã, considera-a  perfeita, linda e maravilhosa. Tem gente que além de ser muito fã e considerá-la perfeita, linda e maravilhosa, seria capaz de comê-la se fosse mulher.

Se você se enquadra entre os itens acima, amigo, nem tudo em Java são flores! Não sei se você sabe, mas algumas coisas em Java são cabeludas, mais até do que outras linguagens, sejam elas mais novas ou mais velhas que Java.

Um exemplo é o mecanismo de template do C++ que em Java é chamado de generic. Quando o Java 5 veio muita gente olhou é disse: "Ah! Agora Java tem template que-nem C++!" Peraí, essa pessoa disse "que-nem"? "QUE-NEM"?! Ora! Vai dormir! Nunca desconfiou porque em Java o mecanismo não é chamado de template? É porque generics é pior, é mais fraco, é mais "for dummies" que template! Se você não acredita, vou te mostrar.
<!--more-->
Isso aqui é um código em C++ usando template, chamei-o de generic_builder.h

{% highlight cpp %}
#ifndef _GENERIC_BUILDER_H
#define	_GENERIC_BUILDER_H

namespace objectzilla {

    template<typename TargetObject>
    class GenericBuilder {

        private:
            TargetObject* instance;

        public:
            GenericBuilder() : instance() {
                instance = new TargetObject();
            }

            TargetObject&amp; operator*() {
                return *(operator->());
            }

            TargetObject* operator->() {
                return instance;
            }

            ~GenericBuilder() {
                delete instance;
            }

    };
}
#endif	/* _GENERIC_BUILDER_H */
{% endhighlight %}

Essa classe possui um ponteiro para o typename TargetObject, que pode ser qualquer coisa, um int, um vector, uma classe sua...

Seu construtor aloca um novo objeto, existem também os métodos que sobrecarregam o operador * e -&gt; para retornar o ponteiro de TargetObject, além de um destrutor (não seria mais apropriado "destruidor"?).

Tenho também uma classe POJO, se é que podemos chamar assim, da classe MyData, no arquivo my_data.h.

{% highlight cpp %}
#ifndef _MY_DATA_H
#define	_MY_DATA_H

namespace objectzilla {

    class MyData {
        private:
            int value;

        public:
            MyData() : value(11) {}
            int getValue() const {return value;}
            void setValue(int value) {MyData::value = value;}

    };
}

#endif	/* _MY_DATA_H */
{% endhighlight %}

É besta. Só armazena um valor, que por padrão é 11 (não me pergunte o porquê), e seu getter e setter.

Agora vamos criar um método main e jogar tudo no liquidificador.

{% highlight cpp %}
#include <iostream>

#include "generic_builder.h"
#include "my_data.h"

using namespace std;

int main(int argc, char** argv) {

    objectzilla::GenericBuilder<objectzilla::MyData> gb;

    // Aos desavisados, existe uma sobrecarga no operador-> na classe
    // GenericBuilder que devolve um objeto, e &Atilde;&copy; nesse objeto que eu estou
    // invocando o m&Atilde;&copy;todo getValue(), n&Atilde;&pound;o no objeto GenericBuilder
    cout << gb->getValue() << endl;

}
{% endhighlight %}

Esse trecho vai criar um GenericBuilder com MyData como typename, e vai invocar o método operator-&gt;() para trazer o objeto MyData e invocar nele o método getValue().

"Grande merda!", disse o fanzoca do Java, "depois do Java 5 podemos fazer essa mesma coisa!". Pode mesmo?
Então vamos lá. É a sua chance de mostrar todo o poderio de Java.

"Deixa comigo! Aqui está a classe GenericBuilder feita em Java!"

{% highlight java %}
package com.wordpress.objectzilla.template;

public class GenericBuilder<TargetObject> {

    private TargetObject instance;

    public GenericBuilder() {
        instance = new TargetObject();
    }

    public TargetObject getInstance() {
        return instance;
    }
}
{% endhighlight %}

Hum! Parabéns, fanzoca do Java! Bem parecido com o meu exemplo em C++. Mas talvez você deveria ter passado um javac antes de se humilhar na frente de todo o mundo, porque esse código NÃO COMPILA!

(Platéia) "Ohhhhhh!"

É isso mesmo. Tá vendo a linha 8 onde você dá um new em TargetObject? Ela não vai compilar pois, segundo a especificação atual, você não pode instanciar um tipo genérico, pode apenas criar suas referências. Isso, apesar de em C++ você poder fazer gato e sapato com o seu typename.

Poderíamos tentar alguma gambiarra? Usar algo com reflection? Vamos tentar trocar o construtor para:

{% highlight java %}
    public GenericBuilder()
            throws InstantiationException, IllegalAccessException {
        instance = TargetObject.class.newInstance();
    }
{% endhighlight %}

Ainda assim não vai compilar. Apesar da construção "TargetObject.class" estar correta, você não pode fazer nada com esse retorno. Estranho não?

O que compila é assim.

{% highlight java %}
   public GenericBuilder(Class<TargetObject> aClass)
            throws InstantiationException, IllegalAccessException {
        instance = aClass.newInstance(); // ah! agora sim compila
    }
[{% endhighlight %}

Eu tenho que passar no construtor uma instância de Class do mesmo objeto que eu estou definindo como generic, um outro diferente não passaria da compilação por causa da limitação da instância de Class que tem que ser generic de TargetObject.

E aí eu teria a classe Main abaixo (a classe MyData em java foi omitido):

{% highlight java %}
package com.wordpress.objectzilla.template;

public class Main {

    public static void main(String[] args) {
        try {
            GenericBuilder<MyData> gb = new GenericBuilder<MyData>(MyData.class);

            System.out.println(gb.getInstance().getValue()); 

        } catch (Exception ex) {
            System.err.println(ex.getMessage());
            ex.printStackTrace();
        }
    }
}
{% endhighlight %}

Faz a mesma coisa que em C++, porém ao custo de um parâmetro a mais no construtor. Óbvio que, além de redundante, é inseguro. Poderíamos ter escrito:

{% highlight java %}
	GenericBuilder<MyInterface> gb2 = new GenericBuilder<MyInterface>(MyInterface.class);
{% endhighlight %}

E não haveria problema nenhum na hora da compilação, mas haveria lançamento de exceção quando executado, porque eu estaria tentando criar uma instancia de uma interface via reflection. Em C++, numa situação de uma classe abstrata, o programa simplesmente não compilaria (tudo bem que geraria um monte de erros esquisitos, mas geraria erros).

Generics em Java é um fracasso. Parece que foi imaginado pra funcionar exclusivamente em Collections, porque é só ali que o Generics reside, em nenhum outro lugar mais.

Ninguém usa porque, além do fato de não se poder criar instâncias, não há uma vantagem de performance em Java como existe em C++. Por debaixo dos panos, não passa de um monte de referências Object que serão "casteados" numa hora apropriada.

Não que tudo em Generics seja ruim, algumas decisões foram acertadas, como a possibilidade de criar uma referencia como em "MyObj&lt;? extends MyInterface&gt;" para limitar as classes que podem ser usadas.

Mas não deveria ser desse jeito. Java é uma linguagem anacrônica, ao mesmo tempo que se tem propriedades de reflexão, não existe um jeito fácil na linguagem de manipular seu próprio código.

Generics poderia ser implementado diferente, por exemplo: quando se criasse uma instância com generic, seria criado um objeto dinamicamente que fosse a junção da classe genérica e do parâmetro desta. Seria algo próximo da programação orientada  a aspectos. E pra evitar problemas de se instanciar uma interface ou uma classe abstrata por engano, poderia criar uma nova sintaxe:

{% highlight java %}
class MyClass<+Type> {

}
{% endhighlight %}

Colocaria um sinal de "+" (mais) antes do parâmetro, para indicar que só pode usar classes concretas. Assim, se algum programador associar com uma interface ou classe abstrata, daria erro de compilação.

Se dessem todo o potencial para o Generics, encontraríamos bastante idéias para utilizá-lo. Mas infelizmente isso não ocorreu.
