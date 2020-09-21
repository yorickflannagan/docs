MQX - Utilitários para IBM MQSeries e web services [Wiki](http://gitlab.corerj.caixa/REUSO/mqx/wikis/home).
Instalação do MQ Client para linux: https://www.ibm.com/support/knowledgecenter/SSFKSJ_8.0.0/com.ibm.mq.ins.doc/q009010_.htm

Release 0.9.8
	- Inclusão de suporte à interface com o Barramento Caixa (implementaçao do método call() da interface MQXConnection).
	A seção mqxconnectors/mqxconnector/queue de cada fila deve incluir as entradas descritas a seguir:
	<!-- 
		Integração ao Barramento Caixa
		Configuração requerida para as filas de PUT (br.gov.caixa.mqx.input = false):
			put.timeout: tempo máximo (em milissegundos) de aguardo pelo envio da requisição. Default: 4000
			put.expire: tempo (em décimos de segundo) máximo de vida da requisição na fila MQ. Default: 800
		Configuração requerida para as filas de GET (br.gov.caixa.mqx.input = true):
			get.timeout: tempo máximo de aguardo (em milissegundos) pela recepção da mensagem de resposta. Default 4000
			pool.size: tamanho máximo do pool de conexões à fila de GET. Default: 5
			pool.timeout: tempo máximo (em milissegundos) de aguardo pela obtenção de uma conexão do pool. Default: 4000
	-->
	<property name="br.gov.caixa.mqx.put.timeout" value="[valor]" />
	<property name="br.gov.caixa.mqx.put.expire" value="[valor]" />
	<property name="br.gov.caixa.mqx.get.timeout" value="[valor]" />
	<property name="br.gov.caixa.mqx.pool.size" value="[valor]" />
	<property name="br.gov.caixa.mqx.pool.timeout" value="[valor]" />
	- Inclusão de bacalhau em PooledMQXConnection para lidar com múltiplos serviços com a mesma configurão. Este bacalhau será pescado, cozido e devorado o mais breve possível.
Release 0.9.9
	- Correção de bug na criação de pool de objetos MQXConnection
Release 0.9.10
	- Reimplementação do acesso a PooledMQXConnection para acesso via JNDI em web services:
		- Remoção da seção mqxconnectors do arquivo de configuração dos web services
		- Criação de um novo arquivo de configuração exclusivamente para os mqxconnectors, com a mesma estrutura
		- Inclusão da seguinte entrada na seção subsystem xmlns="urn:jboss:domain:naming do arquivo de configuração do JBoss, onde o value aponta para a localização do arquivo de configuração:
		  	<bindings>
		  		<simple name="java:global/MQXConnectors" value="/home/magut/development/sefipx/mqclient/docs/xml/config-connector.xml" />
		  	</bindings>
		- Para cada seção mqxconnectors/mqxconnector existente no arquivo de configuração, adicionar a seguinte entrada à seção bindings do arquivo de configuração do JBoss, onde [connectorName] é o nome do conector:
	  		<object-factory name="java:global/MQXConnectors/[connectorName]" module="br.gov.caixa.mqx" class="br.gov.caixa.mqx.pool.PoolFactory" />
	  	- Por exemplo:
        <subsystem xmlns="urn:jboss:domain:naming:1.4">
            <bindings>
                <simple name="java:global/MQXConnectors" value="/home/magut/development/sefipx/mqclient/docs/xml/config-connector.xml"/>
                <object-factory name="java:global/MQXConnectors/mqdes" module="br.gov.caixa.mqx" class="br.gov.caixa.mqx.pool.PoolFactory"/>
            </bindings>
            <remote-naming/>
        </subsystem>
	  	- Efetuar o lookup de uma instância de MQXConnection pelo name registrado, por exemplo:
			final InitialContext ctx = new InitialContext();
			final MQXConnection hConn = (MQXConnection) ctx.lookup("java:global/MQXConnectors/mqdes");
	  	- Prosseguir como na versão anterior.
Release 0.9.11
	- Correção de bug na interface com o barramento
Release 0.9.12
	- Atualização de br.gov.caixa.mqx.MQXConnectionImpl para suporte ao barramento Caixa
Release 1.0.0A
	- Pool de conexões ao MQ para receive() suporta ultrapassar os limites configurados
	- Inclusão de serviços de notificação e log em PooledMQXConnection
Release 1.0.0
	- Solução do problema do Woodstox tentar acessar a Internet para resolver o DTD de schema XML
	- Solução do problema de configuração do web service cliente precisar ser colocado num diretório especifico: passa admitir arquivo em qualquer lugar do file system; a configuração do web service servidor não muda
	- Correção do problema de não reconexão ao MQ Series na presença de falhas
	- Correção do bug de SingletonCursor que não permitia a recriação do prepared statement

Release 1.0.1
	- Correção de race condition em SingletonCursor

Release 1.0.2
	- Redefinição das diretivas de processamento XML
Release 1.0.3
	- Solução do bug de ClassNotFound no Soapparser.
	- Adição de verificação da XMLInputFactory, sendo obrigatória que seja Woodstox.
Release 1.0.5
	- Correção de inconformidade do header SOAPAction em WebClient
Release 1.0.6
	- Alteração de WebService para exigir obrigatoriamente um header SOAPAction válido
Release 1.0.7
    - Ajustes no Cursor
Release 1.0.8
   - Mais correções no Cursor 
Release 1.0.9
	- Correção das respostas HTTP com SOAPFault para status code 500
Release 1.0.10
	- Refactoring do cursor para garantia sob multithread
	- Correção da SOAP Fault segundo regra R10.14 da WS-i
Release 1.0.11
	- Correção do bug causado pelo valor default da chave br.gov.caixa.mqx.put.expire na configuração
Release 1.0.12
	- WebClient: captura de exceção gerada por falta de SOAP Fault com manutenção do erro original
Release 1.0.13
	- Implementação de bacalhau para contornar o problema de parser XML, que faz unescape de texto
	independente do valor da propriedade XMLInputFactory.IS_REPLACING_ENTITY_REFERENCES. Este
	comportamento foi observado tanto no Woodstox quanto no parser padrão do JDK. O assunto continua
	em estudo.
Release 1.0.14
	- Recompilação para correção de falha na gerência de configuração
Release 1.0.15
	- Correção do bug do PUT e GET na mesma fila (reason code 2039
	ou 2037)
Release 1.0.16
	- Implementação de obtenção da profundidade da fila
Release 1.0.17
	- Ajuste do queuedepth, visibilidade de friend para public e tipo de long para int

Release 2.3.0
	- Suporte à versão 8 do IBM MQSeries
	- Eliminação da necessidade de instalação do MQClient no servidor alvo
	- Suporte a UserID e senha para a conexão MQ
	- Pacote de Distribuição:
	------------------------------------|--------------------------------------------------
	Componente                          | Descrição 
	------------------------------------|--------------------------------------------------
	mqx-2.3.0.jar                       | Componente Java
	------------------------------------|--------------------------------------------------
	mqx-2.3.0-jar-with-dependencies.jar | Componente Java e suas dependências (exceto J2EE)
	------------------------------------|--------------------------------------------------
	mqx-2.3.0-sources.jar               | Código fonte do componente Java
	------------------------------------|--------------------------------------------------
	libmqx.so/mqx.dll                   | Componente JNI
	------------------------------------|--------------------------------------------------
	libmqic_r.so/mqic.dll               | Dependência da IBM MQI
	------------------------------------|--------------------------------------------------
	libmqe_r.so/mqe.dll                 | Dependência da IBM MQI
	------------------------------------|--------------------------------------------------
	- Dependências de mqx-2.3.0.jar:
	---------------------|--------|-------------------------
	Pacote               | Versão | Pacote
	---------------------|--------|-------------------------
	org.snmp4j.snmp4j    | 2.5.6  | Ver Maven
	---------------------|--------|-------------------------
	org.crypthing.things | 2.5.11 | Ver Maven
	---------------------|--------|-------------------------
	com.ibm.mq           | 8.0    | com.ibm.mq.allclient.jar
	---------------------|--------|-------------------------
	- Localização do componente libmqx.so/mqx.dll (para uso do Java)
		1. Diretório referenciado pela System.Property br.gov.caixa.mqx.Lib
		2. Qualquer diretório referenciado pela System.Property java.class.path
		3. No mesmo diretório de distribuição do pacote Java mqx-2.3.0.jar/mqx-2.3.0-jar-with-dependencies.jar
		4. Qualquer diretório referenciado pela System.Property java.library.path
	- Localização dos componentes libmqic_r.so/mqic.dll e libmqe_r.so/mqe.dll
		1. No mesmo diretório onde estiver localizado o componente libmqx.so/mqx.dll
		2. Sob Linux, no diretório /usr/lib64
		3. Sob Windows, em qualquer dos diretórios referenciados pelo %PATH%
	- Para conexões autenticadas, utilize as seguintes entradas na configuração:
	--------------------------|------------------------------------------------------------------
	Chave                     | Valor
	--------------------------|------------------------------------------------------------------
	br.gov.caixa.mqx.userid   | Sigla do usuário com permissão de acesso ao MQ Series configurado
	--------------------------|------------------------------------------------------------------
	br.gov.caixa.mqx.password | Senha (em claro) do usuário
	--------------------------|------------------------------------------------------------------
	
	
