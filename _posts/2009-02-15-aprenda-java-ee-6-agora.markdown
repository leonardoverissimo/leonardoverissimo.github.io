---
layout: post
title: Aprenda Java EE 6, agora!
tags:
- Java
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
Quando fiquei craque no Java, a versão do "Enterprise Edition" era ainda 1.4. Eu aprendi a versão 5, logo depois do lançamento do Glassfish. Mas agora estou fazendo algo diferente, não vou esperar a versão Java EE 6 sair, vou aprender as coisas tão logo as novidades apareçam.

E você também pode fazer isso. Pegue [uma versão promoted do Glassfish no site deles](http://download.java.net/glassfish/v3/promoted/), que, assim como o [Lost](http://abc.go.com/primetime/lost/index?pn=index), tem sempre uma versão nova nas quartas à noite, que você baixa na quinta. A última versão, que estou usando nesta semana, é a b36.

Baixe a última versão disponível e descompacte num lugar que você achar apropriado. No Eclipse, na hora de instalar o servidor, basta escolher a opção "Glassfish v3 Prelude" (se você não usava Glassfish, na tela de novo servidor há uma opção de baixar os adaptadores para outros servidores). Pronto!

Eu vou fazer uma aplicação de exemplo, que é uma agenda de consultório médico. É babaca, mas tem uma regras óbvias que não dá pra ser tratado com o velho CRUD.

Vamos lá. Crie um "Dynamic Web Project" cujo Runtime seja o "Glassfish v3 Prelude", vou chamá-lo de **AgendaMedica**. Não sei como vocês lidam com testes no Eclipse, mas costumo criar um novo projeto Java que referencia o projeto que estou interessado em testar. No caso, criarei também o projeto Java **AgendaMedicaTeste**. Após criado, dê &lt;Alt&gt; + &lt;Enter&gt; sobre o projeto, clique em "Java Build Path", vá na aba "Projects" e adicione o projeto AgendaMedica.

No projeto AgendaMedicaTeste, eu já criei um método de testes pra realizar a consulta:

{% highlight java %}
package br.com.objectzilla.agendamedica;

import java.util.Calendar;

import org.junit.Assert;
import org.junit.Test;

public class AgendamentoTeste {
	
	@Test
	public void pacienteMarcaHoraComMedico() {
		// um paciente...
		Paciente paciente = new Paciente();
		paciente.setNome("Maria");
		
		// um médico...
		Medico medico = new Medico();
		medico.setNome("Dr. Gregory House");
		
		// ambos agendam um horário
		Agendamento agendamento = new AgendamentoImpl();
		Calendar horario = Calendar.getInstance();
		horario.set(2009, Calendar.MARCH, 5, 17, 00, 00);
		
		agendamento.marcaConsulta(medico, paciente, horario);
		
		// agora, médico tem consulta com paciente marcada às 5
		Paciente pacienteMarcado = medico.consulta(horario);
		Assert.assertEquals(paciente.getNome(), pacienteMarcado.getNome());
	}
}
{% endhighlight %}

Beleza, mas o código não compila, né? Ora, não seja por isso. Vá criando as classes clicando nos erros com o botão direito do mouse, só não se esqueça de colocar as classes no projeto AgendaMedica. No caso, Agendamento é uma interface e o restante são classes.

Bom, tente rodar o script de teste (clique com botão direito sobre a classe e dê Run As... > JUnit Test). Vai dar erro, óbvio! Por isso, convido a você a escrever código nas classes até o script rodar com sucesso. Não se preocupe, se você não conseguir ou não tiver paciência, no final do post eu mostro o link onde você pode baixar o código.

Agora, vamos transformar a interface Agendamento em um Stateless Session Bean. Não precisa nem mover pra um projeto próprio, pois agora os EJBs podem ser "deployados" em aplicações WAR. Basta adicionar @Local na interface (a v3 ainda não possui suporte a @Remote), assim:

{% highlight java %}
@Local
public interface Agendamento {
	// ...
}
{% endhighlight %}

E adicionar @Stateless na classe que implementa:

{% highlight java %}
@Stateless
public class AgendamentoImpl implements Agendamento {
	// ...
}
{% endhighlight %}

Pronto, faça o deploy do projeto web no servidor Glassfish v3 e veja o seguinte log aparecer:

<pre>
INFO: Bound Java:Global name [business view] : java:global/AgendaMedica/AgendamentoImpl#br.com.objectzilla.agendamedica.Agendamento
INFO: Bound Java:Global name [single business view] : java:global/AgendaMedica/AgendamentoImpl
</pre>

Repare em como é realizado o _binding_ com o JNDI. A partir da versão 6, não será mais uma implementação específica de contêiner. Todos os servidores de aplicações, se quiserem ser homologados para Java EE 6, deverão dispor seus EJBs, em contextos JNDI, de acordo com o seguinte padrão:

**java:global[/&lt;application-name&gt;]/&lt;module-name&gt;/&lt;bean-name&gt;#&lt;interface-name&gt;**

Onde:

**&lt;application-name&gt;** é opcional, refere-se ao nome do pacote EAR (não tem no nosso caso).

**&lt;module-name&gt;** é o módulo onde está o EJB _AgendaMedica_.

**&lt;bean-name&gt;** é o nome do bean _AgendamentoImpl_.

**&lt;bean-name&gt;** é o nome do bean _AgendamentoImpl_.

**&lt;interface-name&gt;** é o nome completo da interface que implementa o bean _br.com.objectzilla.agendamedica.Agendamento_ .

O leitor atento deve ter reparado: está sendo feito **dois _bindings_** para o EJB! Isso também faz parte do padrão. Se o bean implementar apenas uma interface, um _alias_ será criado onde o nome da interface não aparece.

Já que criamos o Session Bean, seria interessante se usássemos, não é não? Vamos fazer um Servlet, mas não precisa tomar o susto de configurar o wer.xml, pois basta criar um classe que herde de HttpServlet e adicionar uma anotação de acordo com o seguinte código:

{% highlight java %}
@WebServlet(value="/agendamento")
public class AgendamentoServlet extends HttpServlet {
	@Override
	public void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
			IOException {
	}
}
{% endhighlight %}

Após o deploy, basta digitar http://localhost:8080/AgendaMedica/agendamento e você vai ver... uma tela em branco. Normal, a gente não fez nada mesmo. Agora, vamos preencher o doGet() com uma chamada ao método de marcarConsulta() de agendamento. Eu também injetei uma instância de EJB.


{% highlight java %}
@WebServlet(value="/agendamento")
public class AgendamentoServlet extends HttpServlet {
	@EJB
	private Agendamento agendamento;
	@Override
	public void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
			IOException {
		String nomeMedico = req.getParameter("medico");
		String nomePaciente = req.getParameter("paciente");
		int hora = Integer.parseInt(req.getParameter("hora"));
		// criando médico
		Medico medico = new Medico();
		medico.setNome(nomeMedico);
		// criando paciente
		Paciente paciente = new Paciente();
		paciente.setNome(nomePaciente);
		// criando horário
		Calendar horario = Calendar.getInstance();
		horario.set(Calendar.HOUR, hora);
		horario.set(Calendar.MINUTE, 0);
		horario.set(Calendar.SECOND, 0);
		// chamando método de agendamento
		agendamento.marcaConsulta(medico, paciente, horario);
		PrintWriter pw = resp.getWriter();
		pw.format("Eu juro que marquei consulta para o paciente %s, com o médico %s, na hora %d.",
				medico.getNome(), paciente.getNome(), horario.get(Calendar.HOUR));
	}
}
{% endhighlight %}

Se você rodar http://localhost:8080/AgendaMedica/agendamento?medico=Jose&paciente=Adolfo&hora=14, vai ver que a aplicação mostra uma mensagem na tela que não diz se foi, ou não foi, feito algo. Precisaria de uma tela de consulta, mas só dá pra fazer isso se nossa aplicação persistisse os objetos entre as requisições, o que atualmente não ocorre.

Vou fazer isso no próximo post, onde apresentarei mais novos recursos e novas funcionalidades na aplicação pra que ela finalmente funcione de verdade.

Quem quiser, pode ir alterando a aplicação por conta própria, deixei no repositório [GitHub](https://github.com/) em <http://github.com/leonardoverissimo/javaee-6-application/tree/master>.

