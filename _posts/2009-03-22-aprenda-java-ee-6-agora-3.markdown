---
layout: post
title: Aprenda Java EE 6, agora! (3)
tags:
- Sem categoria
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
Seguindo o [post anterior](http://www.objectzilla.com.br/2009/02/24/aprenda-java-ee-6-agora-2/), continuaremos a aplicação de AgendaMedica. Lembra do que faltava? Precisávamos de um conceito de Consulta, que não havia no nosso modelo. E precisávamos que a consulta tivesse uma característica melhor do mundo real, como, por exemplo, não permitir duas consultas no mesmo horário e não permitir consultas fora do horário do médico.
<!--more-->

Como faríamos isso? Bom, primeiro haveria a classe Consulta, que só existirá a partir do relacionamento entre Paciente e Medico.
                                           
Mas antes vou subdividir as minhas classes atuais em pacotes, assim:

<pre>
.
|-- aplicacao
|   |-- AgendaFormServlet.java
|   |-- Agendamento.java
|   |-- AgendamentoServlet.java
|   `-- ConsultaServlet.java
|-- dominio
|   |-- AgendamentoImpl.java
|   |-- Medico.java
|   |-- MedicoNaoEncontradoException.java
|   |-- MedicoRepositorio.java
|   |-- Paciente.java
|   |-- PacienteNaoEncontradoException.java
|   `-- PacienteRepositorio.java
`-- persistencia
    |-- MedicoDAO.java
    `-- PacienteDAO.java
</pre>

Se desse jeito é uma boa idéia ou não, eu não sei. Mas sei que a famosa subdivisão por _patterns_ que se encontra por aí não tem o menor sentido. Pacotes deveriam ser divididos sobre a regra: _"alta coesão e baixo acoplamento"_, assim como é com classes, arquivos e métodos. A subdivisão por _patterns_ parece ser o contrário disso: tem **baixa coesão** porque um determinado domínio está espalhado entre os pacotes _to_, _dto_, _entity_, _sessionbean_ etc.; e tem **alto acoplamento** porque sob o mesmo pacote está uma classe sobre Veiculo e outra sobre Arvore. Tudo bem que é importante a subdivisão por camadas, mas não vai achar que os TOs estão em uma camada própria!

Dando prosseguimento, vamos criar uma nova classe de teste para a criação da classe Consulta. Sabemos que essa classe não existe sem Paciente e Medico, portanto iremos colocar essas classes no construtor. Não só isso, Consulta não existe sem um horário inicial e a duração dela, e portanto também estará no construtor. 

Mas vamos definir uma regra clara: só é possível marcar consulta em horário disponível. Nesse primeiro teste, como não foi setado um horário, então deve dar erro sempre que marcar uma consulta.

{% highlight java %}
@Test
public void consultaNaoDisponivel() {
	Paciente p = new Paciente();
	p.setNome("Leonardo");
	Medico m = new Medico();
	m.setNome("Dr. Meredith Grey");
	// não informo o período de consulta 
	Calendar c = Calendar.getInstance();
	c.set(2009, Calendar.MARCH, 01, 15, 00, 00);
	try {
		new Consulta(p, m, c.getTime(), 60);
		Assert.fail("Esperava-se o lançamento de exceçao");
	} catch (ConsultaNaoDisponivel e) {	
	}
	Assert.fail();
}
{% endhighlight %}

Gere o construtor e a classe de exceção (eu herdei de **IllegalStateException**) para o compilador não reclamar. Após rodar o teste, vai acusar um erro. Faça o teste rodar adicionando a seguinte instrução no construtor:

{% highlight java %}
public Consulta(Paciente paciente, Medico medico, Date inicio, int duracaoMinutos) {
	throw new ConsultaNaoDisponivel();
}
{% endhighlight %}

Não é o que se espera, mas faz, por enquanto, o teste rodar.

Agora, qual seria um caso onde realmente a consulta aconteceria com sucesso? Seria quando o consulta estivesse dentro de um horário disponível pelo médico. Minha idéia é que médico teria seus horários disponíves (classe HorarioDisponivel) com dia da semana, horário inicial e horário final. A consulta só ocorreria quando a consulta batesse com pelo menos um desses intervalos. 

{% highlight java %}
@Test
public void consultaDisponivel() {
	Paciente p = new Paciente();
	p.setNome("Leonardo");
	Medico m = new Medico();
	m.setNome("Dr. Meredith Grey");
	m.adicioneDisponibilidade(new HorarioDisponivel(DiaSemana.SEGUNDA, 13, 00, 19, 00));
	// não informo o período de consulta 
	Calendar c = Calendar.getInstance();
	c.set(2009, Calendar.MARCH, 02, 15, 00, 00);
	Consulta consulta = new Consulta(p, m, c.getTime(), 60);
	SimpleDateFormat sdf = new SimpleDateFormat("ddMMyyyyhhmmss");
	//
	Assert.assertEquals(p, consulta.getPaciente());
	Assert.assertEquals(m, consulta.getMedico());
	Assert.assertEquals("02032009150000", sdf.format(consulta.getInicio()));
	Assert.assertEquals("02032009160000", sdf.format(consulta.getFim()));
}
{% endhighlight %}

Depois de acertar os problemas de compilação, rode o teste. O construtor de consulta irá lançar exceção, falhando o teste. Claro, agora é hora de acertarmos este construtor para que não lance a exceção nesse segundo caso de teste.

A nova classe de consulta ficou assim:

{% highlight java %}
public class Consulta implements Serializable {

	public Consulta(Paciente paciente, Medico medico, Date inicio, int duracaoMinutos) {
		Set<HorarioDisponivel> disponibilidades = medico.getDisponibilidades();
		
		Calendar inicioConsulta = Calendar.getInstance();
		inicioConsulta.setTime(inicio);
		
		Calendar finalConsulta = Calendar.getInstance();
		finalConsulta.setTime(inicio);
		finalConsulta.add(Calendar.MINUTE, duracaoMinutos);
		
		boolean disponivel = false;

		for (HorarioDisponivel disponibilidade : disponibilidades) {
			
			if (inicioConsulta.get(Calendar.DAY_OF_WEEK) == disponibilidade.diaSemana.ordinal() + 1) {
				Calendar limiteInicial = Calendar.getInstance();
				// o dia seria igual à data marcada, apenas para não complicar
				limiteInicial.setTime(inicio);

				limiteInicial.set(Calendar.HOUR_OF_DAY, disponibilidade.horaInicial);
				limiteInicial.set(Calendar.MINUTE, disponibilidade.minutoInicial);
				limiteInicial.set(Calendar.SECOND, 0);

				Calendar limiteFinal = Calendar.getInstance();
				// o dia seria igual à data marcada, apenas para não complicar
				limiteFinal.setTime(inicio);
				// se hora final é menor que hora inicial, assumimos que a hora
				// final refere-se ao dia seguinte. Exemplo: inicial: 22 e
				// final: 04 significa um período que começa às 10 da noite de
				// um dia e vai até às 4 da manhã do dia seguinte
				if (disponibilidade.horaFinal < disponibilidade.horaInicial) {
					limiteFinal.add(Calendar.DAY_OF_MONTH, 1);
				}
				
				limiteFinal.set(Calendar.HOUR_OF_DAY, disponibilidade.horaFinal);
				limiteFinal.set(Calendar.MINUTE, disponibilidade.minutoFinal);
				limiteFinal.set(Calendar.SECOND, 0);
				
				// verifica se a data está no intervalo
				if (inicioConsulta.compareTo(limiteInicial) >= 0 && finalConsulta.compareTo(limiteFinal) <= 0) {
					disponivel = true;
					break;
				}
			}
		}

		if (disponivel) {
			this.paciente = paciente;
			this.medico = medico;
			this.inicio = inicioConsulta.getTime();
			this.fim = finalConsulta.getTime();
		} else {
			throw new ConsultaNaoDisponivel();
		}
	}

  // getters omitidos
  	
	private Paciente paciente;
	private Medico medico;
	private Date inicio;
	private Date fim;

}
{% endhighlight %}

E ainda precisei alterar a classe do médico para adicionar os seguintes métodos e atributos:

{% highlight java %}
public class Medico implements Serializable {

  // ...	
	public void adicioneDisponibilidade(HorarioDisponivel horarioDisponivel) {
		disponibilidades.add(horarioDisponivel);	
	}
	
	public Set<HorarioDisponivel> getDisponibilidades() {
		return Collections.unmodifiableSet(disponibilidades);
	}

  //...	
	private Set<HorarioDisponivel> disponibilidades = new HashSet<HorarioDisponivel>();
}
{% endhighlight %}

Com isso, a classe HorarioDisponivel ficou assim:

{% highlight java %}
public class HorarioDisponivel {
	public enum DiaSemana {DOMINGO, SEGUNDA, TERCA, QUARTA, QUINTA, SEXTA, SABADO}

	public HorarioDisponivel(DiaSemana diaSemana, int horaInicial, int minutoInicial,
			int horaFinal, int minutoFinal) {
		this.diaSemana = diaSemana;
		this.horaInicial = horaInicial;
		this.minutoInicial = minutoInicial;
		this.horaFinal = horaFinal;
		this.minutoFinal = minutoFinal;
	}

	DiaSemana diaSemana;
	int horaInicial;
	int minutoInicial;
	int horaFinal;
	int minutoFinal;
}
{% endhighlight %}

Hora de refatorar, três problemas à vista: um é o fato do construtor da Consulta estar acessando demais as propriedades de HorarioDisponivel, indício que boa parte dessa lógica poderia estar nessa classe; outro é o fato do médico ter **duas** formas de realizar consulta, a antiga através dos métodos _consulta(Calendar)_ e _consulta(Paciente, Calendar)_ e a nova que foi feita agora, ou seja, o antigo precisa ser removido; por último é o fato da aplicação ainda salvar o objeto Médico na hora de criar uma consulta.

Ao primeiro problema, criei um método _isDentroHorario()_ dentro de HorarioDisponivel, contendo o "miolo" do laço _for_. Assim, o construtor de Consulta foi reduzido a:

{% highlight java %}
Set<HorarioDisponivel> disponibilidades = medico.getDisponibilidades();
		
		Calendar inicioConsulta = Calendar.getInstance();
		inicioConsulta.setTime(inicio);
		
		Calendar finalConsulta = Calendar.getInstance();
		finalConsulta.setTime(inicio);
		finalConsulta.add(Calendar.MINUTE, duracaoMinutos);
		
		boolean disponivel = false;

		for (HorarioDisponivel disponibilidade : disponibilidades) {
			
			if (disponibilidade.isDentroHorario(inicioConsulta, finalConsulta)) {
				disponivel = true;
				break;
			}
		}

		if (disponivel) {
			this.paciente = paciente;
			this.medico = medico;
			this.inicio = inicioConsulta.getTime();
			this.fim = finalConsulta.getTime();
		} else {
			throw new ConsultaNaoDisponivel();
		}
{% endhighlight %}

O teste rodou com sucesso.

Ao segundo problema, retirei os dois métodos _consulta()_ e adicionei o novo método _marcaConsulta()_, assim:

{% highlight java %}
public void marcaConsulta(Paciente paciente, Date horario) {
		Consulta novaConsulta = new Consulta(paciente, this, horario, 60);
		if (getConsultas() == null) {
			setConsultas(new HashSet<Consulta>());
		}
		getConsultas().add(novaConsulta);
	}
{% endhighlight %}

É claro que isso teve impacto em alguns casos de testes. Mas o código final não será mostrado aqui.

Ao terceiro problema, movi o método

{% highlight java %}
public interface MedicoRepositorio {
	// ...
	void salvaConsultaMedico(Medico medico);
}
{% endhighlight %}

para a nova classe ConsultaRepositorio, agora com novo nome:

{% highlight java %}
public interface ConsultaRepositorio {
	
	void marca(Consulta consulta);
}
{% endhighlight %}

Resolvi problemas de compilação. O método marcaConsulta de AgendamentoImpl ficou assim (não mostrei, mas foi adicionado mais um bean do EJB):

{% highlight java %}
	public void marcaConsulta(long medicoId, long pacienteId, Date horario)
			throws PacienteNaoEncontradoException, MedicoNaoEncontradoException {

		Paciente paciente = pacienteRepositorio.getPaciente(pacienteId);
		Medico medico = medicoRepositorio.getMedico(medicoId);
		
		consultaRepositorio.marca(new Consulta(paciente, medico, horario, 60));
	}
{% endhighlight %}

Resolvi outros problemas de compilação também; e criei também um "DAO", à semelhança dos outros, que também é um Singleton:

{% highlight java %}
@Singleton
public class ConsultaDAO implements ConsultaRepositorio {
	private List<Consulta> consultas = new ArrayList<Consulta>();
	@Override
	public void marca(Consulta consulta) {
		consultas.add(consulta);
	}
}
{% endhighlight %}

Descobri também que ninguém chama o método marcaConsulta() de Medico. Vou apagá-lo. Vou rodar o teste e... opa! Deu erro! No método pacienteMarcaHoraComMedico(), é feita a verificação de consulta através da propriedade "lista de consultas" de médico. Porém, por não estar persistido de verdade, essa relação acaba não existindo.

{% highlight java %}
try {
	agendamento.marcaConsulta(medicoId, pacienteId, horario.getTime());
	
	// verifica se os métodos definidos no mock foram realmente chamados
	context.assertIsSatisfied();
	
	// agora, médico tem consulta com paciente marcada às 5
	Assert.assertEquals(1, medicoRetornado.getConsultas().size());
	Consulta consulta = medicoRetornado.getConsultas().iterator().next(); // <-- ocorre erro aqui
	Assert.assertEquals(pacienteRetornado.getNome(), consulta.getPaciente().getNome());
	
} catch (PacienteNaoEncontradoException e) {
	Assert.fail();
} catch (MedicoNaoEncontradoException e) {
	Assert.fail();
}
{% endhighlight %}

Vamos mudar um pouco e retornar o registro obtido através do DAO de Consulta. Assim: lá em cima, eu criei uma classe falsa (tá, o nome correto é classe interna anônima) de ConsultaRepositorio:

{% highlight java %}
final List<Consulta> consultaRetornada = new LinkedList<Consulta>();
		
// ajustando o repositório de médico
final MedicoRepositorio medicoRep = context.mock(MedicoRepositorio.class);
final ConsultaRepositorio consultaRep = new ConsultaRepositorio() {
	@Override
	public void marca(Consulta consulta) {
		consultaRetornada.add(consulta);
	}
};
{% endhighlight %}

e o código com problemas fica assim:

{% highlight java %}
try {
	agendamento.marcaConsulta(medicoId, pacienteId, horario.getTime());
	
	// verifica se os métodos definidos no mock foram realmente chamados
	context.assertIsSatisfied();
	
	// agora, médico tem consulta com paciente marcada às 5
	Consulta consulta = consultaRetornada.get(0);
	Assert.assertEquals(pacienteRetornado.getNome(), consulta.getPaciente().getNome());
	
} catch (PacienteNaoEncontradoException e) {
	Assert.fail();
} catch (MedicoNaoEncontradoException e) {
	Assert.fail();
}
{% endhighlight %}

Sucesso.

Ufa! Vamos continuar com o problema do agendamento. O que acontece quando são marcadas duas consultas ao mesmo tempo? Deveria dar erro, certo? Mas, se rodarmos esse novo teste:

{% highlight java %}
@Test
public void duasConsultasAoMesmoTempo() {
	medico.adicioneDisponibilidade(HorarioDisponivel.getInstance(DiaSemana.SEGUNDA, 8, 00, 17, 00));
	// a primeira consulta está ok
	Consulta consulta = new Consulta(paciente, medico, horarioConsulta, 60);
	SimpleDateFormat sdf = new SimpleDateFormat("ddMMyyyyHHmmss");
	Assert.assertEquals(paciente, consulta.getPaciente());
	Assert.assertEquals(medico, consulta.getMedico());
	Assert.assertEquals("02032009150000", sdf.format(consulta.getInicio()));
	Assert.assertEquals("02032009160000", sdf.format(consulta.getFim()));
	// a segunda consulta é no mesmo horário
	Paciente paciente2 = new Paciente();
	paciente2.setNome("Maria");
	try {
		new Consulta(paciente2, medico, horarioConsulta, 60);
		Assert.fail("Realizada segunda consulta por engano");
	} catch (ConsultaNaoDisponivel e) {
	}
{% endhighlight %}

Vai dar erro. Pois o sistema atual não restringe duas consultas.

Para resolver isso, precisamos que, ao marcar uma nova consulta, verifica-se primeiro a existência de um registro no mesmo horário antes de começar. Porém, não acho prudente fazer isso no construtor da entidade, delegarei isso para o repositório.

{% highlight java %}
public interface ConsultaRepositorio {
	
	void marca(Consulta consulta);
	
	boolean existeConsultaPara(Medico medico, Date horario, int duracaoMinutos);
}
{% endhighlight %}

Mas também, não dá pra contar que esse método sempre será chamado ao criar uma consulta. Imaginei uma factory de objetos que retornaria a Consulta e que também chamaria esse método do DAO antes de devolver o objeto. Qualquer um faria esse objeto como Singleton ou algo parecido, farei ele como um Session Bean, mas (pra ficar diferente) farei sem interface, coisa que agora será possível, assim:

{% highlight java %}
package br.com.objectzilla.agendamedica.dominio;

import java.util.Date;

import javax.ejb.EJB;
import javax.ejb.Stateless;

@Stateless
public class ConsultaFactory {

	public Consulta novaConsulta(Paciente paciente, Medico medico, Date inicio, int duracaoMinutos) {
		boolean agendado = consultaRepositorio.existeConsultaPara(medico, inicio, duracaoMinutos);
		if (agendado) {
			throw new ConsultaNaoDisponivel();
		}
		
		return new Consulta(paciente, medico, inicio, duracaoMinutos);
	}
	
	@EJB
	public void setConsultaRepositorio(ConsultaRepositorio consultaRepositorio) {
		this.consultaRepositorio = consultaRepositorio;
	}

	private ConsultaRepositorio consultaRepositorio;
}
{% endhighlight %}

Legal, mas o teste se complica, já que a decisão por existir ou não a consulta cabe à camada de dados, e acabou ficando assim:

{% highlight java %}
@Test
public void duasConsultasAoMesmoTempo() {
	medico.adicioneDisponibilidade(HorarioDisponivel.getInstance(DiaSemana.SEGUNDA, 8, 00, 17, 00));
	ConsultaFactory consultaFactory = new ConsultaFactory();
	Mockery context = new JUnit4Mockery();
	final ConsultaRepositorio consultaRep =  context.mock(ConsultaRepositorio.class);
	context.checking(new Expectations() {{ "{{" }}
		one(consultaRep).existeConsultaPara(medico, horarioConsulta, 60); will(returnValue(false));
		one(consultaRep).existeConsultaPara(medico, horarioConsulta, 60); will(returnValue(true));
	}});
	consultaFactory.setConsultaRepositorio(consultaRep);
	// a primeira consulta está ok
	Consulta consulta = consultaFactory.novaConsulta(paciente, medico, horarioConsulta, 60);
	SimpleDateFormat sdf = new SimpleDateFormat("ddMMyyyyHHmmss");
	Assert.assertEquals(paciente, consulta.getPaciente());
	Assert.assertEquals(medico, consulta.getMedico());
	Assert.assertEquals("02032009150000", sdf.format(consulta.getInicio()));
	Assert.assertEquals("02032009160000", sdf.format(consulta.getFim()));
	// a segunda consulta é no mesmo horário
	Paciente paciente2 = new Paciente();
	paciente2.setNome("Maria");
	try {
		consultaFactory.novaConsulta(paciente2, medico, horarioConsulta, 60);
		Assert.fail("Realizada segunda consulta por engano");
	} catch (ConsultaNaoDisponivel e) {
	}
}
{% endhighlight %}

Sempre fico com uma pulga na orelha quando uso mocks para testes. Afinal, estou testando o meu código (como se espera) ou o mock?  Bom, não consigo pensar em algo melhor. Vou deixar assim, que está rodando com sucesso.

Dessa vez, irei usar o JavaServer Faces 2.0 como camada visual, e vou ignorar aqueles Servlets que estavam antes.

Não irei mostrar todas as páginas, mas gostaria de mostrar como é para fazer um managed bean. É assim agora:

{% highlight java %}
@ManagedBean(name="consulta")
@SessionScoped
public class ConsultaManagedBean {
  // ...
}
{% endhighlight %}

Nada de XML, você define o seu managed bean através de anotação. E o escopo é definido através de anotações @SessionScoped, @ViewScoped (um novo escopo que dura enquanto existir a página), @RequestScoped e @ApplicationScoped.

Uma outra coisa é que não é necessário declarar o destino da página em faces-config.xml todas as vezes. Por exemplo, se você definir um retorno com uma String, assim:

{% highlight java %}
public String novo() {
		// preenche combo de médicos
		List<Medico> listaMedico = medicoRepositorio.todosMedicos();
		
		medicosItens = new LinkedHashMap<String, Long>();
		for (Medico m : listaMedico) {
			medicosItens.put(m.getNome(), m.getId());
			
		}
		
		// preenche combo de pacientes
		List<Paciente> listaPaciente = pacienteRepositorio.todosPacientes();
		
		pacientesItens = new LinkedHashMap<String, Long>();
		for (Paciente p : listaPaciente) {
			pacientesItens.put(p.getNome(), p.getId());
		}
		
		return "formNovaConsulta";
	}
{% endhighlight %}

E não definir essa String na navegações, será exibido a página cujo nome é essa String. Nesse caso acima, será exibido à página "formNovaConsulta.xhtml".

Bom, é isso. Eu sei que demorei demaaais para mostrar essa nova parte. Mas ficou muito complicado, e percebi que fugi muito o tópico. Nas próximas séries sobre Java EE 6, não utilizarei esse exemplo, focando apenas nas novas features. Até.

