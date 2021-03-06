---
layout: post
title: 'Você sabe o que é: Closure?'
tags:
- Java
- Ruby
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
Existem três propostas de closures em Java, [BGGA](http://www.javac.info/), [CICE](http://docs.google.com/View?docid=k73_1ggr36h)/[ARM](http://docs.google.com/View?docid=dffxznxr_1nmsqkz) e [FCM](http://docs.google.com/View?docid=ddhp95vd_0f7mcns). A primera proposta é a mais forte mas também a mais criticada, e, ao que parece, [não haverá espaço para closures no Java 7](http://hamletdarcy.blogspot.com/2008/12/java-7-update-from-mark-reinhold-at.html). Mas o que é closure?<!--more-->

Primeira coisa: em linguagens que suportam closures, funções são tratadas como dados, ou seja, tem a mesma semântica que um inteiro, uma string, um objeto... Isso é diferente de linguagens onde closures não existem, como Java, em que a função é definida estaticamente, e de lá não sai. Na verdade, no nível da máquina, tanto dados quando procedimentos são representados por um endereço de uma região de memória, são as linguagens sem closures que abstraem variáveis e funções como sendo coisas distintas.

Em linguagens procedimentais, como C, existe o conceito chamado **ponteiro de função** que, basicamente, é a obtenção de uma referência de uma função pra ser executada em um tempo posterior. O uso de ponteiro de função é bastante baixo nível, e é utilizado bastante, por exemplo, no Linux, já que os módulos, os callbacks e os próprios processos são executados dinamicamente, de acordo com a vontade de seus usuários.

Porém, pra muitos, a simples referência de uma função não se caracteriza uma closure. Principalmente em linguagens orientadas a objetos, closure é associada à captura de um contexto onde a closure é definida. A função closure captura sempre as referências do objeto e do método que a criou pois a qualquer momento pode chamar uma de suas variáveis ou de seus métodos, ou seja, ela "fecha" o contexto (daí o nome closure, ou "fechamento"). Parece esquisito, mas na prática, essa característica faz com que você sinta que o escopo "é o de menos" e que possa usar os métodos e variáveis disponíveis, sem se preocupar depois se o contexto vai desaparecer ou não. É como se, depois de declarada a função,  houvesse uma variável invisível que tivesse a referência de tudo o que está externo e que, portanto, poderia usá-lo enquanto a função existir. Pense, por exemplo, nas **inner class** do Java, você simplesmente não consegue chamar qualquer variável do objeto mais externo, a menos que esteja marcado final. Porém, em linguagens com closures, não existe uma restrição como essa.

Closures, por serem também um dado, costumam ser transportados de uma variável à outra, passando por vários contextos diferentes.

Vamos a um exemplo: na linguagem Ruby, eu poderia definir uma função e associá-la a uma variável:
{% highlight ruby %}
quadrado = lambda do |x|
  x * x
end
{% endhighlight %}
Por ser um dado, então atribuí-la a uma outra variável é possível:
{% highlight ruby %}
funcaoMisteriosa = quadrado
{% endhighlight %}
E tanto "quadrado" quanto "funcaoMisteriosa" possuem a referência à mesma função. E ambos são capazes de avaliar a função, veja:
{% highlight ruby %}
puts quadrado.call(4)    # 16
puts funçãoMisteriosa.call(4)  # 16
{% endhighlight %}
Isso por si só não é uma closure. Como dito antes, a linguagem C possui a capacidade de transportar ponteiros de funções, e é apenas isso que eu estou fazendo. Eu poderia, em Ruby, utilizar variáveis e métodos do contexto mais externo, mesmo que este não estivesse visível no momento em que a função fosse realmente executado. Como exemplo, imagine que uma função é declarada em uma classe, e que fosse possível passar pra um outro objeto:

{% highlight ruby %}
# essa classe só tem um método que vai
# executar o bloco recebido
class Bar
  def execute(bloco)
    puts bloco.call(10)
  end
end

class Foo
  def initialize(atributo)
    @atributo = atributo
  end

  def teste()
    # criei uma função e associei à variável funcaoQualquer
    # repare que a função tem uma instância de uma variável pertencente
    # a esse objeto (atributo)

    blocoQualquer = lambda do |x|
      @atributo + x
    end
    # criei um objeto...
    bar = Bar.new
    # ...e passei a variável contendo a função
    bar.execute(blocoQualquer)
  end
end

# crio o objeto passando 5 como attributo
foo = Foo.new(5)
# chamo teste, que vai criar um bloco e chamar execute de um outro objeto,
# que por sua vez, vai imprimir uma soma (o resultado é 15). Repare que foi
# utilizado, na sua execução, uma variável (@atributo) que o objeto do tipo
# Bar não conhece, mas que está no contexto do bloco passado (houve a closure).
foo.teste
{% endhighlight %}

Foi possível chamar a variável @atributo porque, ao declarar o bloco "blocoQualquer" foi feita a captura do contexto em volta; como, por exemplo as variáveis do método Foo#teste e os atributos de instância do objeto Foo.

Cada linguagem tem seu jeito de chamar uma closure. Por isso, nos próximos posts, vou dar exemplos em algumas delas. Aguarde.

UPDATE:
Essa série sobre Closures continua [aqui](http://www.objectzilla.com.br/2009/01/07/closure-em-cpp/), e [aqui](http://www.objectzilla.com.br/2009/01/14/closure-em-java/).
