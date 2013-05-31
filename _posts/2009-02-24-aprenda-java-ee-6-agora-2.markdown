---
layout: post
title: Aprenda Java EE 6, agora! (2)
tags:
- Java
- Sem categoria
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
Dessa vez, estarei utilizando a versão b37 do Glassfish v3. Por causa disso, você vai ter que fazer alterações à mão no plugin do Eclipse para o servidor Glassfish. Na versão b36 havia as pastas web e ejb dentro de module. Agora, todos os jars estão dentro de module, sem distinção de pastas.

Primeiro, feche o Eclipse. Iremos copiar o plugin do Glassfish e descompactá-lo, para alterar o seu arquivo de configurações. No Linux eu fiz assim, supondo que ECLIPSE_HOME seja uma variável onde aponta para a pasta do Eclipse:<!--more-->

{% highlight bash %}
$ mkdir -p ~/plugin # cria pasta plugin dentro de seuhome

$ cd $ECLIPSE_HOME/plugins #entra na pasta plugins dentro de eclipse

$ cp com.sun.enterprise.jst.server.sunappsrv_1.0.16.jar ~/plugin/ # copia o plugin do Glassfish para sua pasta

$ cd ~/plugin/ # vá para a sua pasta...

$ unzip com.sun.enterprise.jst.server.sunappsrv_1.0.16.jar -d plugin_glassfish # e extrai o conteúdo do jar dentro da subpasta plugin_glassfish

$ cd plugin_glassfish/serverdef/ #entre na pasta serverdef (do conteúdo recém extraído)

$ vim glassfishv3preludeserverdef.xml # edite este arquivo (não precisa ser, necessariamente, o vim)
{% endhighlight %}

Você vai encontrar isso na linha 184:
{% highlight xml %}
<archive path="${sunappserver.rootdirectory}/modules/web/jsf-impl.jar" />
<archive path="${sunappserver.rootdirectory}/modules/web/jstl-impl.jar" />
{% endhighlight %}
Troque para:
{% highlight xml %}
<archive path="${sunappserver.rootdirectory}/modules/jsf-impl.jar" />
<archive path="${sunappserver.rootdirectory}/modules/jstl-impl.jar" />
{% endhighlight %}

Agora vamos reempacotar o jar e devolvê-lo pro Eclipse:

{% highlight bash %}
$ cd ~/plugin # volta onde está a cópia do plugin do Eclipse

$ mv com.sun.enterprise.jst.server.sunappsrv_1.0.16.jar com.sun.enterprise.jst.server.sunappsrv_1.0.16.jar__old # mude o nome

$ cd plugin_glassfish/ # entre na pasta com conteúdo "dezipado"

$ zip -r ../com.sun.enterprise.jst.server.sunappsrv_1.0.16.jar ./ # e faça o zip de novo com o mesmo nome do jar

$ cd .. # volte um nível

$ cp com.sun.enterprise.jst.server.sunappsrv_1.0.16.jar $ECLIPSE_HOME/plugins/ # e copie o novo jar para a pasta de plugins do Eclipse
{% endhighlight %}

Ao abrir o Eclipse de novo, na versão b37, o projeto não sentirá mais falta de bibliotecas. Voltando...

No [post anterior](http://www.objectzilla.com.br/2009/02/15/aprenda-java-ee-6-agora/), fizemos uma aplicação já utilizando os recursos do EJB Lite. Porém, faltava alguma coisa para persistir os objetos.

No teste, instanciava-se ali mesmo os objetos necessários e jogava-os no método marcaConsulta(). Porém, num caso real, os objetos precisariam existir muito antes, armazenados num meio persistente (na maioria das vezes, em um banco de dados relacional). A prática corrente é encapsular essa obtenção de objetos do meio persistente através do DAO (Data Access Object, ou objeto de acesso aos dados). Nessa aplicação, seguiremos a prática de separar a interface da implementação, e chamarei a interface de **.&#42;Repository** e a classe que a implementa de **.&#42;DAO**. Usarei EJB Lite também para a implementação de persistência.

Aquele primeiro teste será modificado para buscar os objetos do repositório. Estaremos utilizando o JMock para simular o DAO, pois, além de não termos uma fonte de dados real (por enquanto), testes que dependem de recursos externos perdem a confiabilidade. Melhor fazer um stub e ter certeza que traz o resultado esperado

{% highlight java %}
package br.com.objectzilla.agendamedica;

import java.util.Calendar;

import org.jmock.Expectations;
import org.jmock.Mockery;
import org.jmock.integration.junit4.JUnit4Mockery;
import org.junit.Assert;
import org.junit.Test;

public class AgendamentoTeste {
	
	@Test
	public void pacienteMarcaHoraComMedico() {
		
		Mockery context = new JUnit4Mockery();

		// ajustando o repositório de paciente
		final PacienteRepositorio pacienteRep =  context.mock(PacienteRepositorio.class);
		
		context.checking(new Expectations() {{ "{{" }}
			Paciente paciente = new Paciente();
			paciente.setNome("Maria");
			
			oneOf (pacienteRep).getPaciente(32L); will(returnValue(paciente));
		}});
		
		// ajustando o repositório de médico
		final MedicoRepositorio medicoRep = context.mock(MedicoRepositorio.class);
		
		context.checking(new Expectations() {{ "{{" }}
			Medico medico = new Medico();
			medico.setNome("Dr. Gregory House");
			
			oneOf (medicoRep).getMedico(4L); will(returnValue(medico));
		}});
		
		// um paciente...
		Paciente paciente = pacienteRep.getPaciente(32L);
		
		// um médico...
		Medico medico = medicoRep.getMedico(4L);
		
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


Tanto MedicoRepositorio quanto PacienteRepositorio são interfaces (mais tarde, criarei as implementações). Ao criá-las, o teste já é executado com sucesso.

Agora chegou a hora de refatorar. Repare que o teste, do jeito que está, é imbecil. Os mocks apenas complicaram a obtenção de objetos e, se deixasse como antes, seria a mesma coisa! Mas eu fiz essa alteração para um próximo passo: os repositórios não serão mais chamados pelo meu teste, mas pelo método marcaConsulta(), que receberia apenas os ids de médico e paciente. E como consequência, os mocks servirão para indicar que os métodos dos repositórios realmente são chamados. Veja como fica:

{% highlight java %}
		// ajustando o repositório de paciente (omitido, pois é como era antes)
		// ajustando o repositório de médico (omitido, pois é como era antes)
		
		// id de paciente e médico
		long pacienteId = 32L;
		long medicoId = 4L;
		
		// cria um agendamento, passando os repositórios como parâmetro
		Agendamento agendamento;
		{
			AgendamentoImpl impl = new AgendamentoImpl();
			impl.setPacienteRepositorio(pacienteRep);
			impl.setMedicoRepositorio(medicoRep);
			agendamento = impl;
		}
		
		// cria um horário para a consulta
		Calendar horario = Calendar.getInstance();
		horario.set(2009, Calendar.MARCH, 5, 17, 00, 00);
		
		agendamento.marcaConsulta(medicoId, pacienteId, horario);
		
		// verifica se os métodos definidos no mock foram realmente chamados
		context.assertIsSatisfied();
		
		// agora, médico tem consulta com paciente marcada às 5
		Paciente pacienteMarcado = medicoRetornado.consulta(horario);
		Assert.assertEquals(pacienteRetornado.getNome(), pacienteMarcado.getNome());

{% endhighlight %}

Tente executar até que fique com sucesso. Se quiser, dentro do método marcaConsulta(), ao invés de pegar os objetos dentro dos repositórios, crie novos objetos Medico e Paciente. Vai dar erro, porque foi adicionado no teste a instrução _context.assertIsSatisfied()_, que garante que os métodos definidos no mock sejam realmente chamados por apenas uma vez (e dê erro caso contrário).

Existe ainda mais uma oportunidade de teste: o que acontece quando eu uso um identificador para um objeto que não existe? Deveria lançar exceção, mas, atualmente, a assinatura do método dos repositórios não possui qualquer sinalização disso. Farei também um teste para o caso de não achar objetos. Mas nesse caso, não mostrarei aqui, estando apenas no [meu GitHub](http://github.com/leonardoverissimo/javaee-6-application/tree/master).

Chegou a hora da verdade, como fazer os façades existentes se tornarem Session Beans? Simples, com anotações! Primeiro, as interfaces de repositório:

{% highlight java %}
@Local
public interface MedicoRepositorio {
	Medico getMedico(long id) throws MedicoNaoEncontradoException;
}
{% endhighlight %}

{% highlight java %}
@Local
public interface PacienteRepositorio {
	Paciente getPaciente(long id) throws PacienteNaoEncontradoException;
}
{% endhighlight %}

A implementação do Session Bean Agendamento possui agora as dependências com os repositórios:

{% highlight java %}
@Stateless
public class AgendamentoImpl implements Agendamento {
	@Override
	public void marcaConsulta(long medicoId, long pacienteId, Calendar horario)
			throws PacienteNaoEncontradoException, MedicoNaoEncontradoException {
		Paciente paciente = pacienteRepositorio.getPaciente(pacienteId);
		Medico medico = medicoRepositorio.getMedico(medicoId);
		medico.consulta(paciente, horario);
		medicoRepositorio.salvaConsultaMedico(medico);
	}
	@EJB
	public void setPacienteRepositorio(PacienteRepositorio pacienteRep) {
		pacienteRepositorio = pacienteRep;
	}
	@EJB
	public void setMedicoRepositorio(MedicoRepositorio medicoRep) {
		medicoRepositorio = medicoRep;
	}
	private PacienteRepositorio pacienteRepositorio;
	private MedicoRepositorio medicoRepositorio;
}
{% endhighlight %}

Na implementação dos repositórios, ainda não colocarei a persistência com o banco de dados. Ao invés disso, deixarei os objetos em memória. Numa situação como essa, esse lugar da memória deveria ser o único em toda a aplicação, ou seja, _singleton_. Até agora, _singletons_ deveriam ser feitos à mão; na próxima versão, será possível usar EJB, bastando uma anotação, veja:

{% highlight java %}
@Singleton
public class MedicoDAO implements MedicoRepositorio {
  //...
}
{% endhighlight %}

Isso é ainda uma demonstração, quando eu usar um banco de dados de verdade, voltarei a usar **@Stateless**. De fato, não é porque existe **@Singleton** que você sempre deve usá-lo porque, com essa anotação, as chamadas aos métodos serão serializadas (não ocorre execução em paralelo), reduzindo sensivelmente a escalabilidade. Portanto, use como último recurso. O código completo do DAO, você vê no [meu GitHub](http://github.com/leonardoverissimo/javaee-6-application/tree/master).



Acabou? Nããããããooo! Vamos imaginar como seria uma tela de agendamento:
- Mostra-se uma lista de médicos, uma lista de pacientes e pede-se um horário de consulta.
- O usuário marca a consulta.
- Mostra-se a consulta marcada.

Três coisas que falta nos DAOs, uma busca por lista de médicos, outra para lista de pacientes, e um método que salva a consulta. Farei os três métodos:

{% highlight java %}
@Local
public interface PacienteRepositorio {
	Paciente getPaciente(long id) throws PacienteNaoEncontradoException;
	List<Paciente> todosPacientes();
}
{% endhighlight %}

{% highlight java %}
@Local
public interface MedicoRepositorio {
	Medico getMedico(long id) throws MedicoNaoEncontradoException;
	List<Medico> todosMedicos();
	void salvaConsultaMedico(Medico medico);
}
{% endhighlight %}

No teste, adicionarei também a exigência do método de _marcarConsulta()_ salvar a consulta do médico. Mas, pra variar, não vou mostrar. (Se você estiver atento, faltou o conceito de Consulta, que está escondido como um atributo do médico. Numa próxima oportunidade, antes de adicionar qualquer outra funcionalidade, criarei esse tipo.)

Vamos às páginas web! Usarei Servlets e JSPs. A primeira página pega todos os pacientes e todos os médicos e exibe uma tela para digitar o horário da consulta. Seguindo o MVC, temos o Servlet agindo como Controller:

{% highlight java %}
@WebServlet(value="/agendaForm")
public class AgendaFormServlet extends HttpServlet {
	private static final long serialVersionUID = -7874706688461756994L;
	@Override
	public void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
			IOException {
		// obtém médicos
		List<Medico> medicos = medicoRepositorio.todosMedicos();
		req.setAttribute("medicos", medicos);
		// obtém pacientes
		List<Paciente> pacientes = pacienteRepositorio.todosPacientes();
		req.setAttribute("pacientes", pacientes);
		// passa para uma página jsp
		req.getRequestDispatcher("/WEB-INF/agendaForm.jsp").forward(req, resp);
	}
	@EJB
	private MedicoRepositorio medicoRepositorio;
	@EJB
	private PacienteRepositorio pacienteRepositorio;
}
{% endhighlight %}

Não se deveria haver duas chamadas ao model dentro de um controller. Mas nesse caso, não consigo imaginar uma situação melhor. Acho muito burocrático fazer um método que retorna um DTO contendo as listas de Medico e Paciente; além do nome do método resultante não ter uma boa abstração (ou você acha que **listaMedicoEPaciente()** é um nome bom?). E não consigo achar um jeito de usar apenas um objeto raiz, onde os outros seriam obtidos através deste; a relação entre Medico e Paciente ainda vai ser criada, ela não existe no momento de carregar o formulário. Portanto, deixarei assim mesmo.

A view não possui nada de novo em relação às versões anteriores:

{% highlight jsp %}
	<jsp:useBean id="medicos" scope="request" type="java.util.List"/>
	<jsp:useBean id="pacientes" scope="request" type="java.util.List"/>
	
	<form method="POST" action="marcaConsulta">
		<p>Selecione o m&eacute;dico:
		<select size="1" name="medicoId">
			<option value="0">M&eacute;dicos</option>
			<c:forEach items="${medicos}" var="medico">
			<option value="${medico.id}">${medico.nome}</option>
			</c:forEach>
		</select>
		</p>
		<p>Selecione o paciente:
		<select size="1" name="pacienteId">
			<option value="0">Pacientes</option>
			<c:forEach items="${pacientes}" var="paciente">
			<option value="${paciente.id}">${paciente.nome}</option>
			</c:forEach>
		</select>
		</p>
		Dia: <input type="text" name="dia" size="10"><br/>
		Hora: <input type="text" name="hora" size="10"><br/>
		
		<p><input type="submit" value="Marcar Consulta"></p>
	</form>
{% endhighlight %}

O conteúdo será enviado à ação marcaConsulta, que será feita através da alteração do antigo Servlet AgendamentoServlet:

{% highlight java %}
package br.com.objectzilla.agendamedica;

import java.io.IOException;
import java.io.PrintWriter;
import java.text.DateFormat;
import java.util.Calendar;
import java.util.Date;

import javax.ejb.EJB;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet(value = "/marcaConsulta")
public class AgendamentoServlet extends HttpServlet {

	private static final long serialVersionUID = -1218896541620785877L;
	
	@EJB
	private Agendamento agendamento;

	@Override
	public void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
			IOException {

		try {
			long medicoId = Long.parseLong(req.getParameter("medicoId"));
			long pacienteId = Long.parseLong(req.getParameter("pacienteId"));

			DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.SHORT,
					DateFormat.SHORT, req.getLocale());

			String horario = req.getParameter("dia") + " " + req.getParameter("hora");
			Date date = dateFormat.parse(horario);

			Calendar cal = Calendar.getInstance(req.getLocale());
			cal.setTime(date);

			// chamando método de agendamento
			agendamento.marcaConsulta(medicoId, pacienteId, cal);

			resp.sendRedirect(resp.encodeRedirectURL("consulta?medicoId=" + medicoId
					+ "&horario=" + horario));

		} catch (Exception e) {
			PrintWriter out = resp.getWriter();
			out.print("Você fez alguma coisa de errado!");
		}
	}
}
{% endhighlight %}

Dessa vez, apaguei o **doGet()** e coloquei a ação de salvar a consulta em um **doPost()**. Realizei um redirect em um outro Servlet que faz a consulta de horários.

Servlet:
{% highlight java %}
package br.com.objectzilla.agendamedica;

import java.io.IOException;
import java.io.PrintWriter;
import java.text.DateFormat;
import java.util.Calendar;
import java.util.Date;

import javax.ejb.EJB;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet(value="/consulta")
public class ConsultaServlet extends HttpServlet {

	private static final long serialVersionUID = 8084899967543431263L;

	@Override
	public void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException,
			IOException {
		
		try {
			long medicoId = Long.parseLong(req.getParameter("medicoId"));
			
			Medico medico = medicoRep.getMedico(medicoId);
			
			DateFormat df = DateFormat.getDateTimeInstance(DateFormat.SHORT, DateFormat.SHORT, req.getLocale());
			Date d = df.parse(req.getParameter("horario"));
			
			Calendar horario = Calendar.getInstance(req.getLocale());
			horario.setTime(d);
			req.setAttribute("horario", horario);
			
			Paciente paciente = medico.consulta(horario);
			
			if (paciente == null) {
				paciente = new Paciente();
				paciente.setNome("(ninguém)");
			}
			
			req.setAttribute("paciente", paciente);
			
			req.getRequestDispatcher("/WEB-INF/consulta.jsp").forward(req, resp);
			
		} catch (Exception e) {
			PrintWriter out = resp.getWriter();
			out.println("Você fez alguma coisa de errado!<br/><br/>");
			e.printStackTrace(out);
		}
	}
	
	@EJB
	private MedicoRepositorio medicoRep;
}
{% endhighlight %}

E JSP:
{% highlight jsp %}
	<jsp:useBean id="paciente" scope="request" type="br.com.objectzilla.agendamedica.Paciente"/>
	<jsp:useBean id="horario" scope="request" type="java.util.Calendar"/>
	
	<p>Data: <fmt:formatDate value="${horario.time}" type="both" dateStyle="long" timeStyle="medium"/></p>
	
	<p>Paciente: ${paciente.nome}</p>
	
	[De novo](agendaForm)
{% endhighlight %}

Você pode agora digitar http://localhost:8080/AgendaMedica/agendaForm no seu browser e inserir uma consulta qualquer. Ao fazer a submissão, uma nova página será exibida, cuja URL tem parâmetros de _medicoId_ e _horario_. Mude esses parâmetros e você verá que, em horários que você não marcou horário, aparece _(ninguém)_ como paciente.

Faltam coisas ainda. A agenda permite horários esdrúxulos, como agendamento no passado, agendamento fora do horário comercial, agendamento de dois pacientes com um minuto de intervalo, e sobreposição de horário. Precisamos resolver todas essas validações. E outra, precisamos criar a classe Consulta, pois não faz sentido temos que mudar Médico quando um paciente quer marcar um dia. E seria bem melhor também, fazer as páginas web com algum framework. JSF 2.0 seria uma ótima pedida.

Mas tudo isso num próximo post.
