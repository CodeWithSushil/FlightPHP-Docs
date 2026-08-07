# Documentação do FlightPHP APM

Bem-vindo ao FlightPHP APM—seu coach pessoal de desempenho para aplicativos! Este guia é seu roteiro para configurar, usar e dominar o Monitoramento de Desempenho de Aplicativos (APM) com FlightPHP. Seja você caçando solicitações lentas ou apenas querendo se empolgar com gráficos de latência, nós temos tudo coberto. Vamos tornar seu aplicativo mais rápido, seus usuários mais felizes e suas sessões de depuração mais fáceis!

Veja uma [demonstração](https://flightphp-docs-apm.sky-9.com/apm/dashboard) do painel para o site Flight Docs.

![FlightPHP APM](/images/apm.png)

## Por que o APM é Importante

Imagine isto: seu aplicativo é um restaurante movimentado. Sem uma forma de rastrear quanto tempo os pedidos levam ou onde a cozinha está emperrando, você está adivinhando por que os clientes estão saindo insatisfeitos. O APM é seu sous-chef—ele observa cada etapa, desde as solicitações recebidas até as consultas de banco de dados, e sinaliza qualquer coisa que esteja te atrasando. Páginas lentas perdem usuários (estudos dizem que 53% saem se um site demora mais de 3 segundos para carregar!), e o APM ajuda você a capturar esses problemas *antes* que eles doam. É paz de espírito proativa—menos momentos de "por que isso está quebrado?" e mais vitórias de "veja como isso funciona perfeitamente!"

## Instalação

Comece com o Composer:

```bash
composer require flightphp/apm
```

Você precisará de:
- **PHP 7.4+**: Nos mantém compatíveis com distribuições Linux LTS enquanto suportamos PHP moderno.
- **[FlightPHP Core](https://github.com/flightphp/core) v3.15+**: O framework leve que estamos potencializando.

## Bancos de Dados Suportados

O FlightPHP APM atualmente suporta os seguintes bancos de dados para armazenar métricas:

- **SQLite3**: Simples, baseado em arquivo e ótimo para desenvolvimento local ou aplicativos pequenos. Opção padrão na maioria das configurações.
- **MySQL/MariaDB**: Ideal para projetos maiores ou ambientes de produção onde você precisa de armazenamento robusto e escalável.

Você pode escolher seu tipo de banco de dados durante a etapa de configuração (veja abaixo). Certifique-se de que seu ambiente PHP tenha as extensões necessárias instaladas (ex: `pdo_sqlite` ou `pdo_mysql`).

## Primeiros Passos

Aqui está seu passo a passo para a excelência do APM:

### 1. Registre o APM

Coloque isto no seu `index.php` ou em um arquivo `services.php` para começar a rastrear:

```php
use flight\apm\logger\LoggerFactory;
use flight\database\SimplePdo;
use flight\Apm;

$ApmLogger = LoggerFactory::create(__DIR__ . '/../../.runway-config.json');
$Apm = new Apm($ApmLogger);
$Apm->bindEventsToFlightInstance($app);

// If you're adding a database connection
// Prefer SimplePdo (or PdoQueryCapture from Tracy Extensions in dev).
// Enable APM query tracking via the options array (5th argument).
$pdo = new SimplePdo('mysql:host=localhost;dbname=example', 'user', 'pass', null, [
	'trackApmQueries' => true, // required to capture queries for the APM
]);
$Apm->addPdoConnection($pdo);
```

**O que está acontecendo aqui?**
- `LoggerFactory::create()` pega sua configuração (mais sobre isso em breve) e configura um logger—SQLite por padrão.
- `Apm` é a estrela—ele escuta os eventos do Flight (solicitações, rotas, erros, etc.) e coleta métricas.
- `bindEventsToFlightInstance($app)` conecta tudo ao seu aplicativo Flight.

**Dica Profissional: Amostragem**
Se seu aplicativo está ocupado, registrar *todas* as solicitações pode sobrecarregar as coisas. Use uma taxa de amostragem (0.0 a 1.0):

```php
$Apm = new Apm($ApmLogger, 0.1); // Logs 10% of requests
```

Isso mantém o desempenho ágil enquanto ainda fornece dados sólidos.

### 2. Configure-o

Execute isto para criar seu `.runway-config.json`:

```bash
php vendor/bin/runway apm:init
```

**O que isso faz?**
- Inicia um assistente perguntando de onde vêm as métricas brutas (fonte) e para onde vão os dados processados (destino).
- O padrão é SQLite—ex: `sqlite:/tmp/apm_metrics.sqlite` para fonte, outro para destino.
- Você terminará com uma configuração como:
  ```json
  {
    "apm": {
      "source_type": "sqlite",
      "source_db_dsn": "sqlite:/tmp/apm_metrics.sqlite",
      "storage_type": "sqlite",
      "dest_db_dsn": "sqlite:/tmp/apm_metrics_processed.sqlite"
    }
  }
  ```

> Este processo também perguntará se você quer executar as migrações para esta configuração. Se você está configurando isso pela primeira vez, a resposta é sim.

**Por que dois locais?**
Métricas brutas se acumulam rapidamente (pense em logs não filtrados). O worker as processa em um destino estruturado para o painel. Mantém as coisas organizadas!

### 3. Processe Métricas com o Worker

O worker transforma métricas brutas em dados prontos para o painel. Execute-o uma vez:

```bash
php vendor/bin/runway apm:worker
```

**O que ele está fazendo?**
- Lê da sua fonte (ex: `apm_metrics.sqlite`).
- Processa até 100 métricas (tamanho de lote padrão) para seu destino.
- Para quando terminar ou se não houver mais métricas.

**Mantenha-o Executando**
Para aplicativos ao vivo, você vai querer processamento contínuo. Aqui estão suas opções:

- **Modo Daemon**:
  ```bash
  php vendor/bin/runway apm:worker --daemon
  ```
  Executa para sempre, processando métricas conforme chegam. Ótimo para desenvolvimento ou configurações pequenas.

- **Crontab**:
  Adicione isto ao seu crontab (`crontab -e`):
  ```bash
  * * * * * php /path/to/project/vendor/bin/runway apm:worker
  ```
  Dispara a cada minuto—perfeito para produção.

- **Tmux/Screen**:
  Inicie uma sessão destacável:
  ```bash
  tmux new -s apm-worker
  php vendor/bin/runway apm:worker --daemon
  # Ctrl+B, then D to detach; `tmux attach -t apm-worker` to reconnect
  ```
  Mantém-o executando mesmo se você sair.

- **Ajustes Personalizados**:
  ```bash
  php vendor/bin/runway apm:worker --batch_size 50 --max_messages 1000 --timeout 300
  ```
  - `--batch_size 50`: Processa 50 métricas por vez.
  - `--max_messages 1000`: Para após 1000 métricas.
  - `--timeout 300`: Sai após 5 minutos.

**Por que se preocupar?**
Sem o worker, seu painel fica vazio. É a ponte entre logs brutos e insights acionáveis.

### 4. Inicie o Painel

Veja os sinais vitais do seu aplicativo:

```bash
php vendor/bin/runway apm:dashboard
```

**O que é isso?**
- Inicia um servidor PHP em `http://localhost:8001/apm/dashboard`.
- Mostra logs de solicitações, rotas lentas, taxas de erro e mais.

**Personalize-o**:
```bash
php vendor/bin/runway apm:dashboard --host 0.0.0.0 --port 8080 --php-path=/usr/local/bin/php
```
- `--host 0.0.0.0`: Acessível de qualquer IP (útil para visualização remota).
- `--port 8080`: Use uma porta diferente se 8001 estiver ocupada.
- `--php-path`: Aponta para o PHP se ele não estiver no seu PATH.

Acesse a URL no seu navegador e explore!

#### Modo de Produção

Para produção, você pode precisar tentar algumas técnicas para fazer o painel funcionar já que provavelmente há firewalls e outras medidas de segurança em vigor. Aqui estão algumas opções:

- **Use um Proxy Reverso**: Configure Nginx ou Apache para encaminhar solicitações para o painel.
- **Túnel SSH**: Se você puder fazer SSH no servidor, use `ssh -L 8080:localhost:8001
youruser@yourserver` para tunelar o painel para sua máquina local.
- **VPN**: Se seu servidor estiver atrás de uma VPN, conecte-se a ela e acesse o painel diretamente.
- **Configure o Firewall**: Abra a porta 8001 para seu IP ou rede do servidor. (ou qualquer porta que você configurou).
- **Configure Apache/Nginx**: Se você tem um servidor web na frente de sua aplicação, pode configurá-lo para um domínio ou subdomínio. Se fizer isso, defina o document root para `/path/to/your/project/vendor/flightphp/apm/dashboard`

#### Quer um painel diferente?

Você pode construir seu próprio painel se quiser! Olhe o diretório vendor/flightphp/apm/src/apm/presenter para ideias de como apresentar os dados para seu próprio painel!

## Recursos do Painel

O painel é sua HQ do APM—aqui está o que você verá:

- **Log de Solicitações**: Cada solicitação com timestamp, URL, código de resposta e tempo total. Clique em "Detalhes" para middleware, consultas e erros.
- **Solicitações Mais Lentas**: Top 5 solicitações que consomem tempo (ex: "/api/heavy" em 2.5s).
- **Rotas Mais Lentas**: Top 5 rotas por tempo médio—ótimo para identificar padrões.
- **Taxa de Erro**: Porcentagem de solicitações que falham (ex: 2.3% de 500s).
- **Percentis de Latência**: Tempos de resposta do 95º (p95) e 99º (p99)—conheça seus piores cenários.
- **Gráfico de Códigos de Resposta**: Visualize 200s, 404s, 500s ao longo do tempo.
- **Consultas/Middleware Longos**: Top 5 chamadas lentas de banco de dados e camadas de middleware.
- **Cache Hit/Miss**: Com que frequência seu cache salva o dia.

**Extras**:
- Filtre por "Última Hora", "Último Dia" ou "Última Semana".
- Alterne o modo escuro para aquelas sessões noturnas.

**Exemplo**:
Uma solicitação para `/users` pode mostrar:
- Tempo Total: 150ms
- Middleware: `AuthMiddleware->handle` (50ms)
- Consulta: `SELECT * FROM users` (80ms)
- Cache: Hit em `user_list` (5ms)

## Adicionando Eventos Personalizados

Rastreie qualquer coisa—como uma chamada de API ou processo de pagamento:

```php
use flight\apm\CustomEvent;

$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('api_call', [
    'endpoint' => 'https://api.example.com/users',
    'response_time' => 0.25,
    'status' => 200
]));
```

**Onde aparece?**
Nos detalhes da solicitação do painel em "Eventos Personalizados"—expansível com formatação JSON bonita.

**Caso de Uso**:
```php
$start = microtime(true);
$apiResponse = file_get_contents('https://api.example.com/data');
$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('external_api', [
    'url' => 'https://api.example.com/data',
    'time' => microtime(true) - $start,
    'success' => $apiResponse !== false
]));
```
Agora você verá se essa API está arrastando seu aplicativo!

## Monitoramento de Banco de Dados

Rastreie consultas PDO assim:

```php
use flight\database\SimplePdo;

$pdo = new SimplePdo('sqlite:/path/to/db.sqlite', null, null, null, [
	'trackApmQueries' => true, // required to capture queries for the APM
]);
$Apm->addPdoConnection($pdo);
```

**O que Você Obtém**:
- Texto da consulta (ex: `SELECT * FROM users WHERE id = ?`)
- Tempo de execução (ex: 0.015s)
- Contagem de linhas (ex: 42)

**Atenção**:
- **Opcional**: Pule isso se você não precisar de rastreamento de BD.
- **SimplePdo (preferido)**: Use `SimplePdo` com `trackApmQueries => true`. O `PdoWrapper` obsoleto ainda funciona (5º argumento do construtor `true`). PDO core bruto ainda não está conectado—fique ligado!
- **Aviso de Desempenho**: Registrar cada consulta em um site com muito BD pode deixar as coisas lentas. Use amostragem (`$Apm = new Apm($ApmLogger, 0.1)`) para aliviar a carga.

**Saída de Exemplo**:
- Consulta: `SELECT name FROM products WHERE price > 100`
- Tempo: 0.023s
- Linhas: 15

## Opções do Worker

Ajuste o worker ao seu gosto:

- `--timeout 300`: Para após 5 minutos—bom para testes.
- `--max_messages 500`: Limita a 500 métricas—mantém finito.
- `--batch_size 200`: Processa 200 de uma vez—equilibra velocidade e memória.
- `--daemon`: Executa sem parar—ideal para monitoramento ao vivo.

**Exemplo**:
```bash
php vendor/bin/runway apm:worker --daemon --batch_size 100 --timeout 3600
```
Executa por uma hora, processando 100 métricas por vez.

## Request ID no Aplicativo

Cada solicitação tem um ID de solicitação único para rastreamento. Você pode usar este ID em seu aplicativo para correlacionar logs e métricas. Por exemplo, você pode adicionar o request ID a uma página de erro:

```php
Flight::map('error', function($message) {
	// Get the request ID from the response header X-Flight-Request-Id
	$requestId = Flight::response()->getHeader('X-Flight-Request-Id');

	// Additionally you could fetch it from the Flight variable
	// This method won't work well in swoole or other async platforms.
	// $requestId = Flight::get('apm.request_id');
	
	echo "Error: $message (Request ID: $requestId)";
});
```

## Atualização

Se você está atualizando para uma versão mais recente do APM, há uma chance de que haja migrações de banco de dados que precisam ser executadas. Você pode fazer isso executando o seguinte comando:

```bash
php vendor/bin/runway apm:migrate
```
Isso executará quaisquer migrações necessárias para atualizar o esquema do banco de dados para a versão mais recente.

**Nota:** Se o seu banco de dados APM for grande em tamanho, essas migrações podem levar algum tempo para serem executadas. Você pode querer executar este comando durante horários de menor movimento.

### Atualizando de 0.4.3 -> 0.5.0

Se você está atualizando de 0.4.3 para 0.5.0, precisará executar o seguinte comando:

```bash
php vendor/bin/runway apm:config-migrate
```

Isso migrará sua configuração do formato antigo usando o arquivo `.runway-config.json` para o novo formato que armazena as chaves/valores no arquivo `config.php`.

## Limpando Dados Antigos

Para manter seu banco de dados organizado, você pode limpar dados antigos. Isso é especialmente útil se você estiver executando um aplicativo movimentado e quiser manter o tamanho do banco de dados gerenciável.
Você pode fazer isso executando o seguinte comando:

```bash
php vendor/bin/runway apm:purge
```
Isso removerá todos os dados com mais de 30 dias do banco de dados. Você pode ajustar o número de dias passando um valor diferente para a opção `--days`:

```bash
php vendor/bin/runway apm:purge --days 7
```
Isso removerá todos os dados com mais de 7 dias do banco de dados.

## Solução de Problemas

Travado? Tente isto:

- **Sem Dados no Painel?**
  - O worker está executando? Verifique `ps aux | grep apm:worker`.
  - Os caminhos de configuração coincidem? Verifique se os DSNs do `.runway-config.json` apontam para arquivos reais.
  - Execute `php vendor/bin/runway apm:worker` manualmente para processar métricas pendentes.

- **Erros do Worker?**
  - Dê uma olhada nos seus arquivos SQLite (ex: `sqlite3 /tmp/apm_metrics.sqlite "SELECT * FROM apm_metrics_log LIMIT 5"`).
  - Verifique os logs do PHP para rastreamentos de pilha.

- **Painel Não Inicia?**
  - Porta 8001 em uso? Use `--port 8080`.
  - PHP não encontrado? Use `--php-path /usr/bin/php`.
  - Firewall bloqueando? Abra a porta ou use `--host localhost`.

- **Muito Lento?**
  - Reduza a taxa de amostragem: `$Apm = new Apm($ApmLogger, 0.05)` (5%).
  - Reduza o tamanho do lote: `--batch_size 20`.

- **Não Está Rastreando Exceções/Erros?**
  - Se você tem [Tracy](https://tracy.nette.org/) habilitado para seu projeto, ele substituirá o tratamento de erros do Flight. Você precisará desabilitar o Tracy e então certificar-se de que `Flight::set('flight.handle_errors', true);` está definido.

- **Não Está Rastreando Consultas de Banco de Dados?**
  - Prefira `SimplePdo` com `['trackApmQueries' => true]` como o 5º argumento do construtor (array de opções).
  - Se você ainda usa o `PdoWrapper` obsoleto, passe `true` como o 5º argumento.
  - Chame `$Apm->addPdoConnection($pdo)` após criar a conexão.