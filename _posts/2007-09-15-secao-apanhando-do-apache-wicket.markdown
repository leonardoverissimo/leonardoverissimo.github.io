---
layout: post
title: 'Seção: Apanhando do... Apache Wicket'
tags:
- Apache Software Foundation
- Apanhando do
- Java
- Wicket
status: publish
type: post
published: true
meta:
  _edit_last: '1790462'
---

Existem duas verdades sobre frameworks Java:

1- Framework Java é que-nem espaço do Gmail, mais de 2900 e  aumentando.

2- Não importa o quão cuidadoso você é na codificação. A primeira tela da sua aplicação é o do seu servidor com todos aqueles stacktraces.

Wicket, é sim, mais um daqueles frameworks que seu chefe só aprovaria colocar no sistema da empresa se estiver bêbado. E AINDA! Assim como todos frameworks Java existentes, a primeira aplicação sua nesse novo framework não vai funcionar.

Mas depois desse susto inicial você vai gostar dele. Sabe por que? <!--more-->Imagine JavaServer Faces, o JSF para os íntimos: ele é um framework orientado a componentes, certo? Esses componentes são definidos em uma página JSP, e uma árvore de componententes é gerada no servidor, enquanto o cliente tem uma árvore de componentes em HTML normal. Bom, beleza!

Agora imagine o Google Web Toolkit, também chamado de GWT para os íntimos. Imagine mesmo! Vamo lá, eu espero você.

Wait...

Wait...

Wait...

Wait...

Wait...

Wait...

E aí, imaginou? Não?

Nem eu! Eu não conheço esse framework! Merece até uma "Seção: Apanhando do... Google Web Toolkit" mais tarde. Mas o que eu ouvi falar dele é que é tipo um Swing, você programa em Java e um HTML com JavaScript será gerado.

Perceberam a diferença?

JSF -&gt; Escreve um monte de scripts, gera um monte de objetos Java.

GWT -&gt; Escreve um monte de objetos Java, gera um monte de scripts. (objetos? Não seria "classe" não, jumento? Eu sei, mas é que aí perde o significado do trocadilho)

Bom, em Wicket você manipula do dois. (DOIS! Ah não, f**eu!) Calma! É mais simples do pode parecer. Primeiro, o designer escreve as páginas HTML naquelas ferramentas de fresco deles, o programador só tem que adicionar atributos como "wicket:id", que-nem o JSF Templating tem o "jsfc", mas mais fácil ainda! Mas o programador não tem só moleza! Para cada HTML, existe uma classe Java que herda de WebPage que manipula os componentes da página HTML marcados com algum atributo Wicket.

O legal é que não existe "overlap" (traduzindo para o português: um em cima do outro e traduzindo para o brasileiro: putaria). O HTML é responsável pela estrutura e apresentação da página e a classe Java filha de WebPage é responsável pela parte lógica, e só.

Bom, eu fui ver se existia um plugin para NetBeans - eu sei que isso não existe, mas não custa tentar né? - e encontrei! Quando eu vi a página principal, eu vi uma citação que deveria servir de exemplo para toda a comunidade internacional, e porque não dizer: IN-TER-PLA-NE-TÁ-RIA, de código aberto. A página é [essa](https://nbwicketsupport.dev.java.net/), e eu vou traduzir o que eles disseram:

> "Trabalhamos para o NetBeans em Praga, República Checa. Então, se você não quer contribuir com código, pode, ao invés disso, contribuir com o dinheiro da cerveja... porque a cerveja checa* é a melhor do mundo e gostaríamos de continuar a fazer essa generosa contribuição à industria checa de cervejas, quando não estamos contribuindo com código para este projeto."

N. do T.: os marketeiros brasileiros sabem desde a idade da pedra que a cerveja checa é a melhor do mundo. Senão, porque então eles sempre colocam uma checa nos comerciais de cerveja?

Bom, mas não peguei o plugin, tava em alpha, e se bobear nem serve no NetBeans 6. Vamos fazer tudo à mão mesmo. Baixe o [zip do Wicket](http://www.apache.org/dyn/closer.cgi/wicket/1.3.0-beta3/).

Ah! Outra! Você vai ter que baixar o [zip do SLF4J](http://www.slf4j.org/download.html), pois o Wicket o exige e não vem junto com ele. O SLF4J é mais uma daquelas idéias filha-da-p*** que o pessoal da comunidade Java costuma ter. Primeiro, a ASF criou o Log4J; a Sun, invejosa, criou um outro Logger "padrão" no seu JDK; aí a ASF viu que ia ficar uma zona e criou o Commons Logging; aí um maldito, que não sei o que que tinha na cabeça, criou essa joça aí: a Simple Loggin Facade For Java, tipo um façade pra vários Loggers existentes. Bom, ao criador dessa biblioteca: queime no mármore do inferno!

Eu coloquei no projeto os jars slf4j-api-1.4.3.jar, obrigatório, e slf4j-jdk14-1.4.3, que serve de façade pro Logger do JDK. Existem outros façades pro Log4J e até um no-op! Além disso, eu coloquei o wicket-1.3.0-beta3.jar, por motivos óbvios.

Vamos ao nosso HelloWorld babaca.

Você quer uma aplicação, certo? Por onde começa? Por uma página JSP? Não em Wicket, onde a sua aplicação é uma classe Java que herda de WebApplication, como abaixo:

{% highlight java %}
package com.wordpress.objectzilla.wicket.helloworld;

import org.apache.wicket.protocol.http.WebApplication;

public class HelloWorldApplication extends WebApplication {

    public HelloWorldApplication() {
    }

    public Class getHomePage() {
        return HelloWorld.class;
    }
}
{% endhighlight %}

Essa classe acima sobrescreve apenas um método: getHomePage(), que, como o nome indica, cospe qual é a página inicial, que definimos que será a classe HelloWorld. Simples assim.

Aí precisamos definir como será a página inicial. Lembra-se que eu falei que tem a página HTML e a classe Java? Pois é olha primeiro o arquivo HelloWorld.html (ATENCÃO! Essa página HTML deve estar no mesmo diretório onde estará HelloWorld.class):

{% highlight html %}
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <title>Hello World</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  </head>
  <body>
      
          Hello World! Agora &eacute; <span wicket:id="now">a mesma hora de ontem</span>.
      
  </body>
</html>
{% endhighlight %}

Ele é uma página normal como qualquer outra, exceto pelo atributo wicket:id da tag span. Haverá na classe HelloWorld um componente com id "now" e que substituirá o conteúdo da tag com esse mesmo id. Isso significa que, o que eu escrevi lá em cima: "a mesma hora de ontem" não será exibido, mas sim um valor definido pela classe HelloWorld. Isso significa também que poderiamos ter escrito: &lt;span wicket:id="now"/&gt;, que daria na mesma. E isso significa também que você poderia ter escrito &lt;span wicket:id="now"&gt;Não é mole não! O meu chefe dá a bunda pro patrão!&lt;/span&gt; e ninguém do seu trabalho iria desconfiar.

O código HelloWorld.java fica assim:

{% highlight java %}
package com.wordpress.objectzilla.wicket.helloworld;

import java.util.Date;
import org.apache.wicket.markup.html.WebPage;
import org.apache.wicket.markup.html.basic.Label;

public class HelloWorld extends WebPage {

    public HelloWorld() {
        add(new Label("now", new Date().toString()));
    }
}
{% endhighlight %}

No caso, basta herdar de WebPage e chamar o método do pai add() para adicionar um componente. No caso adicionamos Label, cujo id é "now" (lembra do id "now" dá página?), e cujo conteúdo é o String da hora atual. Quando renderiza, o Wicket pega o conteúdo do Label e coloca-o na página.

Legal não? É porque você ainda não configurou o arquivo web.xml. Em linhas gerais é o seguinte: apague qualquer tag welcome-file-list, não precisa. Coloque um filtro com: o nome "HelloWorldApplication" (põe o que você quiser) e a classe org.apache.wicket.protocol.http.WicketFilter, também precisa de um parâmetro inicial pra propriedade "applicationClassName", cujo valor é a classe da sua aplicação, no nosso caso é "com.wordpress.objectzilla.wicket.helloworld.HelloWorldApplication". O mapeamento é pro caminho "/*". Pra quem não entendeu o que eu disse, aí vai o código:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
    <filter>
        <filter-name>HelloWorldApplication</filter-name>
        <filter-class>org.apache.wicket.protocol.http.WicketFilter</filter-class>
        <init-param>
<param-name>applicationClassName</param-name>
<param-value>com.wordpress.objectzilla.wicket.helloworld.HelloWorldApplication</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>HelloWorldApplication</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <session-config>
        <session-timeout>
            30
        </session-timeout>
    </session-config>
</web-app>

{% endhighlight %}

E nada mais! É puro convenção-sobre-configuração!

Uma coisa importante a se observar é que o esse xml está diferente dos exemplos da página do Wicket. Lá é a versão 1.2 e não 1.3. O que aconteceu é que o Wicket não era um projeto da ASF, e quando entrou pra Apache, estes, pra variar, tiveram a brilhante idéia de transformar todos os pacotes que começavam em "wicket" para "org.apache.wicket". Nem passou pela cabeça deles que os projetos já existentes passariam a não compilar. Além disso, trocaram a configuração inicial de servlet para filter. Segundo eles, ainda existe um servlet no 1.3 que serve por questões de compatibilidade (tipo, maneira de dizer, né?), mas que já é depreciado. Na realidade não é nem isso, porque eu tentei usar o servlet no começo, e deu um ClassNotFoundException. Acho que depreciado pra eles é: a classe sumiu!

Bom, HelloWorld é muito bonitinho, mas não enche estômago de ninguém. Numa próxima oportunidade mostrarei validação, sessão e ajax no Wicket.

Até.
