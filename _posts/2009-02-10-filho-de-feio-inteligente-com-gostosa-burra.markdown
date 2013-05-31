---
layout: post
title: Filho de feio inteligente com gostosa burra
tags:
- dinâmica
- estática
- Groovy
- Python
- Sem categoria
- tipagem
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
Seguinte, na segunda-feira (09/fev), eu vi um PDF sobre uma [experimentação de tipagem estática em Python](http://wiki.jvmlangsummit.com/pdf/28_Siek_gradual.pdf). Não que eu esteja duvidando da capacidade dos caras que fizeram a apresentação, até porque, mexer com linguagens não é para qualquer um, mas sei lá, o fato de muita gente ter dado um "up" no Reddit implica que muita gente sonha com uma linguagem cuja tipagem é tanto estática quanto dinâmica.

Isso me preocupa, sabia? A impressão que dá é que as pessoas sonham com uma espécie de uma linguagem meio-termo, que acabasse com as guerras nerdísticas e que fosse a única que reinaria soberana sobre todos as outras. E uma que fosse estática e dinâmica, _ao mesmo tempo_, seria a candidata ideal. Infelizmente, tenho que te contar a triste realidade: tipagem estática e dinâmica são antagônicas, tipo água e vinho.

Você já programou em C++? Então, quem conhece o troço sabe que existem vários paradigmas conflitantes entre si. Um paradigma é a programação procedural com conteineres STL, outro é a programação orientada a objetos e outro é programação por templates. Todos bem diferentes, e cuja mistureba só pode ser feita com bastante cuidado por gente que não tem amor à própria vida. Sim, é realmente complicado programar em templates e depois tentar usar OO, é complicado usar o paradigma C quando boa parte do código já está com a sintaxe do C++. E eu desconfio que a mesma coisa pode ocorrer com essas linguagens que misturam tipagem estática e tipagem dinâmica. Vamos fazem uma experimentação com Groovy? Primeiro: o que acontece quando compilamos (com groovyc) o seguinte trecho?

{% highlight groovy %}
// recebe um objeto de tipo indefinido
def imprima(valor) {
    println valor
}

// objeto de tipo "String" numa referência de tipo "String"
String msg = "Olá"

// objeto de tipo "String" passado para parâmetro sem tipo
imprima msg
{% endhighlight %}

Sucesso, é perfeitamente possível passar um objeto qualquer numa referência sem tipo.

Mas, e se invertermos, assim?

{% highlight groovy %}
// recebe um objeto de tipo "String"
def imprima(String valor) {
    println valor
}

// objeto de tipo "String" numa referência sem tipo
msg = "Olá"

// objeto de tipo "String" passado para parâmetro de tipo "String"
imprima msg
{% endhighlight %}

Sucesso também, o compilador não reclama quando uma referência String recebe um objeto de uma referência sem tipo. E se chutarmos o balde, definindo a função imprime() recebendo um int?

{% highlight groovy %}
// recebe um objeto de tipo "int"
def imprima(int valor) {
    println valor
}

// objeto de tipo "String" numa referência de tipo "String"
String msg = "Olá"

// objeto de tipo "int" passado para parâmetro de tipo "String"
imprima msg
{% endhighlight %}

Acha que dá erro de compilação? Dá nada! Compila **com sucesso** também! Não existem casos específicos para isso. Em Groovy, os tipos não são decididos em tempos de compilação. Erros de tipos só são descobertos quando você _executa o código_ com tipagem divergente.

Não tenho nada contra linguagens de tipagens estáticas, nem de dinâmicas. Só acho que cada um deve estar no seu quadrado. Muito se fala que, com linguagens "mistas", como Groovy, se obtém os benefícios da tipagem estática com os benefícios da tipagem dinâmica. Será mesmo? Vamos pensar: Em minha opinião, a vantagem da tipagem estática é a possibilidade de reduzir erros causados por atribuições errôneas, impossibilitanto até mesmo e existência do código executável; a desvantagem é que os tipos estáticos causam um engessamento da aplicação (a famosa dúvida: "como trocar a classe, uma vez que as referências são da classe antiga?"), e cuja solução (criação de classes abstratas e de interfaces) precisa ser feita no começo do desenvolvimento.

A vantagem da tipagem dinâmica (de novo, em minha opinião) é possibilitar um código mais flexível pois, como as referências não se comprometem com tipos, uma nova classe com os mesmos contratos de uma antiga sempre será substituível por esta. A desvantagem é que, principalmente com APIs públicas ou com equipes dispersas, torna-se mais difícil definir um contrato dos métodos e classes.

Veja, Groovy não tem _aquela vantagem_ de uma linguagem estática. Se você retornar, em uma função, um valor de tipo diferente, é possível quebrar o código em algum outro lugar e você nem perceber. Significa que precisará de uma grande suíte (e de uma forte cultura) de testes unitários, como qualquer linguagem dinâmica.

E também não tem aquela vantagem de uma linguagem dinâmica. Tipos não são garantias de que métodos obedeçam estritamente um contrato. Em linguagens "puramente" dinâmicas, como não há tipos nas referências, é permitido uma certa flexibilidade do que se pode ou não pode aceitar. Exemplo, em Ruby, poderia ter métodos que recebem valores válidos, não importando seu tipo.

{% highlight ruby %}
# classe hipotética Triângulo
# válido
t = Triangulo.new :tipo=>:retangulo

# válido também, por que não?
t = Triangulo.new "tipo"=>"retangulo"

# válido
t.hipotenusa 10

# válido, por que não?
t.hipotenusa "10"

# opa! erro! O método lança exceção!
t.cateto_oposto -4
{% endhighlight %}

Tá vendo, os métodos podem tanto receber String quanto números, o importante é o valor resultante ser um número natural. O pensamento da tipagem limita essas espertezas, pois fica-se o tempo todo imaginando: "Não! Esse método tem que receber uma 'classe X'!" (Pior que o indivíduo se esquece até da diferença entre objeto e classe.). Como Groovy não vai incentivar ninguém a abandonar os tipos em tudo quanto é canto, também não se receberá os benefícios das linguagens dinâmicas.

Então, pra amarrar com o título acima. Sou da opinião de que Groovy é ruim... não, corrigindo... de que Groovy **é a pior linguagem do mundo** porque, ao unir o feio inteligente (a tipagem estática) com a gostosa burra (a tipagem dinâmica), não se deu origem a um filho bonito e inteligente, como era de se esperar.

Foi pior, o filho é feio e burro.
