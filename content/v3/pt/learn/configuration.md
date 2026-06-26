# Configuração

## Visão Geral 

Flight fornece uma maneira simples de configurar vários aspectos do framework para atender às necessidades da sua aplicação. Alguns são definidos por padrão, mas você pode substituí-los conforme necessário. Você também pode definir suas próprias variáveis para serem usadas em toda a sua aplicação.

## Entendendo

Você pode personalizar certos comportamentos do Flight definindo valores de configuração
através do método `set`.

```php
Flight::set('flight.log_errors', true);
```

No arquivo `app/config/config.php`, você pode ver todas as variáveis de configuração padrão disponíveis para você.

## Uso Básico

### Opções de Configuração do Flight

A seguir está uma lista de todas as configurações de configuração disponíveis:

- **flight.base_url** `?string` - Substitui a URL base da requisição se o Flight estiver sendo executado em um subdiretório. (padrão: null)
- **flight.case_sensitive** `bool` - Correspondência sensível a maiúsculas e minúsculas para URLs. (padrão: false)
- **flight.handle_errors** `bool` - Permite que o Flight lide com todos os erros internamente. (padrão: true)
  - Se você quiser que o Flight lide com erros em vez do comportamento padrão do PHP, isso precisa ser true.
  - Se você tiver o [Tracy](/awesome-plugins/tracy) instalado, você quer definir isso como false para que o Tracy possa lidar com os erros.
  - Se você tiver o plugin [APM](/awesome-plugins/apm) instalado, você quer definir isso como true para que o APM possa registrar os erros.
- **flight.log_errors** `bool` - Registra erros no arquivo de log de erros do servidor web. (padrão: false)
  - Se você tiver o [Tracy](/awesome-plugins/tracy) instalado, o Tracy registrará erros com base nas configurações do Tracy, não nesta configuração.
- **flight.debug** `bool` - Exibe informações detalhadas de erro (mensagem de exceção, código e stack trace) no navegador quando um erro ocorre. (padrão: false)
  - **Nunca habilite isso em produção** — isso vaza detalhes internos da aplicação. Use apenas para desenvolvimento local ou staging.
  - Quando `false`, um `500 Internal Server Error` genérico é mostrado em vez disso. Combine com `flight.log_errors` para capturar erros no lado do servidor.
- **flight.allow_method_override** `bool` - Permite que o método HTTP seja substituído via o cabeçalho de requisição `X-HTTP-Method-Override` ou um campo `_method` no corpo do POST. (padrão: true)
  - **Definir isso como `false` é recomendado** para aplicações que não precisam de spoofing de método baseado em formulário HTML, pois impede que clientes forjem requisições `DELETE` ou `PUT` através de um formulário POST padrão.
  - Veja [Segurança](/learn/security#flight-configuration-hardening) para mais detalhes.
- **flight.views.path** `string` - Diretório contendo arquivos de template de visualização. (padrão: ./views)
- **flight.views.extension** `string` - Extensão do arquivo de template de visualização. (padrão: .php)
- **flight.content_length** `bool` - Define o cabeçalho `Content-Length`. (padrão: true)
  - Se você estiver usando o [Tracy](/awesome-plugins/tracy), isso precisa ser definido como false para que o Tracy possa renderizar corretamente.
- **flight.v2.output_buffering** `bool` - Usa buffer de saída legado. Veja [migrando para v3](migrating-to-v3). (padrão: false)

### Configuração do Loader

Há adicionalmente outra configuração para o loader. Isso permitirá que você 
carregue classes automaticamente com `_` no nome da classe.

```php
// Habilita o carregamento de classes com underscores
// Padronizado como true
Loader::$v2ClassLoading = false;
```

### Variáveis

Flight permite que você salve variáveis para que possam ser usadas em qualquer lugar da sua aplicação.

```php
// Salva sua variável
Flight::set('id', 123);

// Em outro lugar da sua aplicação
$id = Flight::get('id');
```
Para verificar se uma variável foi definida você pode fazer:

```php
if (Flight::has('id')) {
  // Faça algo
}
```

Você pode limpar uma variável fazendo:

```php
// Limpa a variável id
Flight::clear('id');

// Limpa todas as variáveis
Flight::clear();
```

> **Nota:** Só porque você pode definir uma variável não significa que você deve. Use este recurso com moderação. O motivo é que qualquer coisa armazenada aqui se torna uma variável global. Variáveis globais são ruins porque podem ser alteradas de qualquer lugar da sua aplicação, tornando difícil rastrear bugs. Além disso, isso pode complicar coisas como [testes unitários](/guides/unit-testing).

### Erros e Exceções

Todos os erros e exceções são capturados pelo Flight e passados para o método `error`.
se `flight.handle_errors` estiver definido como true.

O comportamento padrão é enviar uma resposta genérica `HTTP 500 Internal Server Error`
com algumas informações de erro.

Você pode [substituir](/learn/extending) este comportamento para suas próprias necessidades:

```php
Flight::map('error', function (Throwable $error) {
  // Lida com o erro
  echo $error->getTraceAsString();
});
```

Por padrão, erros não são registrados no servidor web. Você pode habilitar isso alterando a configuração:

```php
Flight::set('flight.log_errors', true);
```

#### 404 Não Encontrado

Quando uma URL não pode ser encontrada, o Flight chama o método `notFound`. O comportamento
padrão é enviar uma resposta `HTTP 404 Not Found` com uma mensagem simples.

Você pode [substituir](/learn/extending) este comportamento para suas próprias necessidades:

```php
Flight::map('notFound', function () {
  // Lida com não encontrado
});
```

## Veja Também
- [Estendendo Flight](/learn/extending) - Como estender e personalizar a funcionalidade principal do Flight.
- [Testes Unitários](/guides/unit-testing) - Como escrever testes unitários para sua aplicação Flight.
- [Tracy](/awesome-plugins/tracy) - Um plugin para tratamento avançado de erros e depuração.
- [Extensões do Tracy](/awesome-plugins/tracy_extensions) - Extensões para integrar o Tracy com o Flight.
- [APM](/awesome-plugins/apm) - Um plugin para monitoramento de desempenho de aplicações e rastreamento de erros.

## Solução de Problemas
- Se você está tendo problemas para encontrar todos os valores da sua configuração, você pode fazer `var_dump(Flight::get());`

## Changelog
- v3.18.1 - Adicionadas as opções de configuração `flight.debug` e `flight.allow_method_override`.
- v3.5.0 - Adicionada configuração para `flight.v2.output_buffering` para suportar o comportamento de buffer de saída legado.
- v2.0 - Configurações principais adicionadas.