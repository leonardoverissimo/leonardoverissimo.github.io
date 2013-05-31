---
layout: post
title: Go With the Flow
tags:
- go
- go lang
- objetos
- paralelismo
- Sem categoria
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
A Google tem um poderoso canal de comunicação com os nerds de todo o planeta. Tudo que sai da cabeça desses engenheiros é motivo de burburinho em todo o lugar, até que desapareça tempos depois.

A mais recente invenção dos caras é a [GO](http://golang.org/), uma linguagem de programação de sistemas, compilável, com coletor de lixo e primitivas para concorrência. Em poucos dias, causou vários comentários que, obviamente, se arrefeceram uma semana depois. Mas não vamos deixar essa ausência de comentários abalar este post onde, com apenas meus poucos dias de estudo, mostrarei meus pitacos com relações a aspectos da linguagem que mais se destacam dos demais. (Não mostrarei como montar o ambiente, nem como fazer compilação. Isso você encontra no Google.)<!--more-->

## Sintaxe
A sintaxe é ligeiramente igual às famílias da linguagem C (como Java, C++ ou C#), com a diferença que os tipos vem depois do nome da variável ou da função. Um exemplo:

{% highlight go %}
package main

import "fmt";

// constante, tipo é implícito
const Pi = 3.1415

/* 
Métodos começam com "func", seguido do nome,
dos parâmetros e depois o retorno.
No parâmetro, tipo "int" vem depois no nome "v"
*/
func doubleIt(v int) int {
	return v * 2;
}

// o começo de um programa é o método main do pacote main
func main() {
	// variáveis começam com var, seguido de nome e tipo
	var number1 int = 8;
	// mas também pode ser assim
	var number2 = 10;
	// ou assim
	number3 := 12;

	fmt.Printf("Number1: %d\n", doubleIt(number1));
	fmt.Printf("Number2: %d\n", doubleIt(number2));
	fmt.Printf("Number3: %d\n", doubleIt(number3));
}
{% endhighlight %}

Pode parecer muito revolucionário, mas não é! Dê uma olhada em [Haskell](http://www.haskell.org/) ou [Erlang](http://www.erlang.org/) e depois a gente conversa.
## Sem herança nem sobrecarga, e muito diferente
Em Go, existe a noção de objetos, mas não do jeito tradicional, como visto por aí. Se bobear, alguns nem a chamem de orientado a objetos, afinal.

Primeiro, que não existe herança: todos os objetos não fazem parte de uma classe pai e nem há a possibilidade de classes filhas. Como consequência, a composição de objetos é a única forma possível de se trabalhar. E o mais interessante é que, sem heranças, uma grande Caixa de Pandora desaparece, porque:

- **os métodos e atributos _protected_ vão embora:** só existe visibilidade pública e privada ao pacote, e seu discernimento é feito da seguinte forma: quando um atributo ou método começa em letra maiúscula, é visível a todo mundo; quando minúscula, é protegido (mais sobre isso daqui a pouco).

- Nada de atributo **virtual** que tanto enche quem programa em C# ou C++. Nada de depender do _“worst-case”_ de outras linguagens que assumem que todo _dispatch_ de métodos 	é dinâmico. Em Go, por objetos não terem filhos, todo _dispatch_ de métodos é estático, a menos que você esteja usando uma referência à interface (mais sobre isso depois).

Além do mais, diferente de Java e mais parecido com Python, o atributo **this** não é implícito. E não, não fica no primeiro parâmetro do método, mas num lugar especial, antes do nome do método. Também não há a noção de um escopo de classe onde todos os métodos ficam dentro. Em Go, primeiro é criada uma **struct**, e fora dela, os métodos – lembra um pouco Javascript.

Pra não ficarem boiando, um exemplo de uma classe _Time_ simplificada, que exibe horas e minutos:

{% highlight go %}
package mytime

import "fmt"

// os atributos de uma classe ficam numa struct
type Time struct {
	hour uint;
	minute uint;
}

// método "solto" que cria um Time.
// repare que, por iniciar em letra minúscula,
// só codigos sobre o pacote mytime podem acessá-lo
func newTime(hour uint, minute uint) *Time {
	// um jeito de iniciar uma struct
	t := Time {0, 0};
	// minutos - overflow em hora
	t.minute = minute % 60;
	t.hour = minute / 60;
	// horas - ignora overflow
	t.hour += hour % 24;

	return &t;
}

// método de fábrica
// não existe construtor de objetos
// letra maiúscula indica que é método público
func Create(hour uint, minute uint) *Time {
	return newTime(hour, minute);
}

// aqui é que fica interessante
// este método torna-se membro de Time ao declará-lo como receptor
// (declaração antes do nome do método)
func (t *Time) AddMinute(min uint) {
	totalMin := t.minute + min;
	t.minute = totalMin % 60;
	
	t.AddHour(totalMin / 60);
}

func (t *Time) AddHour(hour uint) {
	totalHour := t.hour + hour;
	t.hour = totalHour % 24;		
}

func (t * Time) Increment() {
	t.AddMinute(1);
}

// os métodos de print chamam esse método
func (time *Time) String() string {
	return fmt.Sprintf("%02d:%02d", time.hour, time.minute);
}
{% endhighlight %}

Ah, ia me esquecendo! As interfaces! Um exemplo que eu precisei fazer é criar um tipo standardInput que lê do _File Descriptor_ 0 (zero) (em Unix, 0 é _sdtin_, 1 é _stdout_ e 2 é _stderr_), porque não existe um método _scanf_, como em C.

{% highlight go %}
// tipo necessário para o reader do bufio,
// que aceita qualquer classe com interface io.Reader		
type standardInput struct {

}

func (si *standardInput) Read(p []uint8) (n int, err os.Error) {
	r, e := syscall.Read(0, p);
	if e != 0 {
		err = os.Errno(e);
	}
	return int(r), err;		
}
{% endhighlight %}

Não tem nada de **implements** aí, mas o tipo já implementa a interface [io.Reader](http://golang.org/pkg/io/#Reader), apenas porque aquele realiza o mesmo método deste. Isso tanto é verdade que quando é usado como parâmetro para [bufio.NewReader](http://golang.org/pkg/bufio/#Reader.NewReader), não ocorre problemas.
## Com paralelismo mais fácil, o design muda
**_Goroutines_** é o jeito da linguagem implementar paralelismo. Não chega a ser tão "na unha" quanto a [Thread](http://java.sun.com/javase/6/docs/api/java/lang/Thread.html) em Java ou a [pthread](http://www.yolinux.com/TUTORIALS/LinuxTutorialPosixThreads.html) em C/Posix, mas não é tão alto nível quanto o modelo de atores implementado em Erlang ou Scala.

Uma _goroutine_ é um método que roda em _background_, e que não avisa ninguém quando termina a execução. Não existe uma comunicação entre o chamador e o chamado, a não ser com o uso de _channels_: uma estrutura síncrona e bidirecional para troca de mensagens. Como comparação, parece igual ao C, onde é possível dar um _fork_ no processo (criar _goroutine_), e depois fazer comunicação entre si através de _pipes (channels)_. Porém, com Go é muito mais fácil e flexível.

_Goroutine_ lembra apenas vagamente o modelo de **_coroutines_** em Python (com seu _yield_, _next_ e _send_), ou o modelo de **Fibers** do Ruby 1.9. Nesses, a diferença é que não existem _channels_, porque os métodos de envio e recebimentos de mensagens fazem parte da referência da _coroutine_.

Acredito que, apesar de _goroutine_ estar associado à otimização de aplicativos que rodam em múltiplos _cores_, a grande vantagem pode ser uma melhor modularização das aplicações. Já vi idéias de se usar uma corrotina para separar códigos com estados mutáveis dos não-mutáveis, mas isso eu discutiria num outro post. Por hora, resolvi fazer uma aplicação interativa que pergunta ao usuário como manipular a hora (classe definida anteriormente). Resolvi fazer uma _goroutine_ que isola o _input-output_ do restante da aplicação. Sim, com certeza, não vai melhorar a execução paralela, mas modulariza melhor. Primeiro, a parte do _input-output_ no código abaixo:

{% highlight go %} 
package iocontrol

import (
	"bufio";
	"syscall";
	"fmt";
	"os";
	"strings";
	"strconv";
)


type Query struct {
	What string;
	HowMany uint;
}

// tipo necessário para o reader do bufio,
// que aceita qualquer classe com interface io.Reader		
type standardInput struct {

}

func (si *standardInput) Read(p []uint8) (n int, err os.Error) {
	r, e := syscall.Read(0, p);
	if e != 0 {
		err = os.Errno(e);
	}
	return int(r), err;		
}

// converte uma string para o tipo Query
func ToQuery(line string) *Query {

	// separando a linha em dois valores
	arr := strings.Split(line, ":", 2);
	
	var what string = "";
	if (len(arr) >= 1) {
		// o primeiro é uma string sem espaços em volta
		what = strings.TrimSpace(arr[0]);
	}

	// não havendo uma string com dois pontos no meio,
	// o array tem tamanho 1
	var howMany uint = 0;
	if (len(arr) == 2) {
		// o segundo é um número
		var err os.Error;
		howMany, err = strconv.Atoui(
			strings.TrimSpace(arr[1]));
		
		if (err != nil) {
			fmt.Println(err);
		}
	}
	// criando a estrutura Query
	return &Query{what, howMany};
}

func IOControl(channel chan string) {
	for {
		// obtendo do channel a resposta a ser exibida
		// na tela
		output := <-channel;
		
		fmt.Println(output);
		
		// lendo a linha de comando
		reader := bufio.NewReader(&standardInput{});
		line, err := reader.ReadString('\n');
		if err == nil {
			channel <- line;
		} else {
			channel <- "error:0";
		}
	}
}
{% endhighlight %}

O código que vai virar uma _goroutine_ é o último aí em cima, o **IOControl**. No código principal abaixo, será criado o _channel_ e a _goroutine_ para execução.

{% highlight go %}
package main

import (
	"fmt";
	"./mytime";
	"./iocontrol";
)

func main() {
	
	// criando um channel
	var io = make(chan string);
	// e criando uma goroutine
	go iocontrol.IOControl(io);

	time := mytime.Create(0, 0);

	fmt.Println("Possible commands:\n"
		"hour : n\n"
		"minute : n\n"
		"tick\n"
		"exit\n");
	// mandando o time em String para o channel "io"
	io <- time.String();

	loop: for {
		// obtendo do channel uma string do próximo pedido
		query := iocontrol.ToQuery(<-io);
		
		// "switch" em Go não requer "break"
		switch query.What {
		case "hour":
			time.AddHour(query.HowMany);
		case "minute":
			time.AddMinute(query.HowMany);
		case "tick":
			time.Increment();
		case "error":
			fmt.Println("an error occurred");
		case "exit":
			break loop;
		}

		io <- time.String();
	}
}
{% endhighlight %}

Executando-o, será perguntado como e quanto a acrescentar na hora atual, até que você digite _exit_.

Preciso dizer também que não é difícil cometer erros na programação paralela. Eu por exemplo, esqueci de por o conteúdo de **IOControl** dentro de um laço infinito. E aí, com a _thread_ morta após a primeira rodada de resposta e pergunta, o _channel_ foi bloqueado na escrita sem ninguém pra ler. Com isso, a aplicação em entrou em _deadlock_, havendo a interrupção do programa com o erro na tela. Ou seja, não é tão desgraçado quanto Thread, mas fique atento.
## Sem exceção, literalmente
Talvez isso decepcione alguns, mas a linguagem Go não tem _exceptions_. Ainda não digeri bem essa notícia, mas existem coisas que fazem com que essa linguagem não caia na armadilha do C.

Uma é que o _“finally”_ pode ser substituído pelo **defer**, que indica o comando que precisa ser invocado incondicionalmente antes da saída do método. Outra, é que, como em Go é possível haver múltiplos retornos, criam-se métodos com dois retornos, o primeiro sendo o conteúdo normal e o segundo o código de erro (se houver).  Assim:

{% highlight go %}

package main

import (
	"os";
	"io";
	"flag";
	"fmt";
)

func data(name string) (string, os.Error) {
	var content string = "";

	f, err := os.Open(name, os.O_RDONLY, 0);
	if (err == nil) {
		defer f.Close();
		var rawContent []byte = nil;
		rawContent, err = io.ReadAll(f);
		content = fmt.Sprintf("%s", rawContent);
	}
	return content, err;
}

func main() {
	flag.Parse();
	
	filename := flag.Arg(0);

	content, err := data(filename);
	
	if (err == nil) {
		fmt.Println(content);
	} else {
		fmt.Println(err);
	}
}
{% endhighlight %}

O compilador impede que método com dois retornos seja associada a apenas uma variável (mesmo com tipo implícito). Então, supõe-se que o programador não tenha como esquecer de tratar os erros.

## Conclusão
Go não é uma linguagem inovadora, mas pode ser um refresco para quem precisa de uma linguagem que tenha um contato próximo à máquina, e não quer depender de C ou C++. Nitidamente, é uma linguagem muito crua, e muitos anos de desenvolvimento precisam acontecer. Mas tem potencial.

P.S.: "Go With the Flow" é um rock do _Queens of the Stone Age_.
