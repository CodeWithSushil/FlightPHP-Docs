# IA e Experiência do Desenvolvedor com Flight

## Visão geral

O Flight é projetado para trabalhar *com* ferramentas de codificação de IA—não contra elas. Uma API pequena e previsível, um layout de aplicação claro no [esqueleto oficial](https://github.com/flightphp/skeleton) e arquivos de instrução específicos do projeto fazem com que assistentes como GitHub Copilot, Cursor, Windsurf, Claude Code e Gemini possam seguir os mesmos padrões que você escreveria manualmente.

Com os comandos integrados do Runway para conectar a provedores de LLM e gerar instruções de projeto, o Flight ajuda você e sua equipe a obter ajuda consistente e relevante sem colar o mesmo contexto em cada chat.

## Compreensão

Assistentes de codificação de IA são mais úteis quando entendem o contexto, as convenções e os objetivos do seu projeto. Os auxiliares de IA do Flight permitem:

- Conectar seu projeto a provedores populares de LLM (OpenAI, Grok, Claude, etc.)
- Gerar e atualizar instruções específicas do projeto para que todos recebam a mesma orientação
- Manter o código escrito à mão e o gerado por IA em um único layout (especialmente com o esqueleto)

Esses recursos vêm com o CLI principal do Flight (via [Runway](/awesome-plugins/runway)) e já estão configurados no iniciador oficial [flightphp/skeleton](https://github.com/flightphp/skeleton).

### O que o esqueleto oferece para IA

O iniciador oficial trata **`AGENTS.md` como a fonte da verdade** para ferramentas de IA:

| Arquivo | Função |
|------|------|
| **`AGENTS.md`** (raiz do projeto) | Regras globais, fluxo de inicialização, namespaces, DI, "o que não fazer" |
| **`AGENTS.md`** com escopo em `app/`, `migrations/`, `tests/`, etc. | Dicas leves e específicas da pasta ao trabalhar nessa árvore |
| **`SECURITY.md`** | Segredos, cabeçalhos, XSS/SQL, denúncia—a segurança permanece deliberada e separada |

**Não** há um arquivo de estilo separado para Copilot / Cursor / Gemini / Windsurf no esqueleto. Aponte seu assistente para o `AGENTS.md` raiz (e deixe-o seguir os links para os arquivos com escopo). Os humanos podem ignorar esses arquivos por completo e usar o [README](https://github.com/flightphp/skeleton); o layout é o mesmo de qualquer forma.

> **A documentação ensina APIs; o esqueleto ensina layout.** Exemplos curtos de `Flight::` nestes documentos são ótimos para aprender. Em uma aplicação esqueleto, prefira classes `App\…`, injeção de construtor e `$this->app` em vez da fachada estática dentro dos controladores. Consulte [Instalação](/install) e [Autoload](/learn/autoloading).

## Uso básico

### Configurando credenciais de LLM

O comando `ai:init` orienta você na conexão do seu projeto a um provedor de LLM.

```bash
php runway ai:init
```

Você será solicitado a:

- Escolher seu provedor (OpenAI, Grok, Claude, etc.)
- Inserir sua chave de API
- Definir a URL base e o nome do modelo

Isso cria as credenciais usadas para solicitações posteriores de LLM (por exemplo, para gerar instruções).

**Exemplo:**
```
Bem-vindo ao AI Init!
Qual API de LLM você deseja usar? [1] openai, [2] grok, [3] claude: 1
Digite a URL base para a API de LLM [https://api.openai.com]:
Digite sua chave de API para openai: sk-...
Digite o nome do modelo que você deseja usar (ex.: gpt-4, claude-3-opus, etc.) [gpt-4o]:
Credenciais salvas em .runway-creds.json
```

### Gerando instruções de IA específicas para o projeto

O comando `ai:generate-instructions` cria ou atualiza instruções para assistentes de codificação de IA, adaptadas ao *seu* projeto.

```bash
php runway ai:generate-instructions
```

Você responderá a algumas perguntas (descrição, banco de dados, templates, segurança, tamanho da equipe, etc.). O Flight usa seu provedor de LLM para gerar instruções e as escreve principalmente em:

- **`AGENTS.md`** na raiz do projeto (independente de ferramenta; o que o esqueleto oficial e a maioria dos agentes modernos esperam)

Dependendo da versão do CLI e das opções, o comando também pode escrever cópias específicas para fluxos de trabalho mais antigos (por exemplo, arquivos de regras para Copilot, Cursor, Windsurf ou Gemini). Para **novos projetos a partir do esqueleto**, trate **`AGENTS.md`** (além de quaisquer arquivos `AGENTS.md` com escopo que você mantenha em `app/`) como a fonte única da verdade—não mantenha cinco arquivos de instrução divergentes manualmente.

**Exemplo:**
```
Descreva para que serve seu projeto? Minha API incrível
Qual banco de dados você planeja usar? MySQL
Qual mecanismo de templates HTML você planeja usar (se houver)? twig
A segurança é um elemento importante deste projeto? (s/n) s
...
Instruções de IA atualizadas com sucesso.
```

Agora as ferramentas de IA podem sugerir código que corresponda à sua pilha e ao seu layout reais—não a um tutorial genérico de PHP.

## Uso avançado

- Personalize credenciais ou caminhos de saída com opções de comando (veja `--help` em cada comando).
- Os auxiliares funcionam com qualquer provedor de LLM que fale uma API compatível com OpenAI.
- Execute novamente `ai:generate-instructions` conforme o projeto evolui para manter os agentes sincronizados.
- No esqueleto, mantenha a política de segurança em **`SECURITY.md`** e o layout de codificação em **`AGENTS.md`** para que nenhum documento vire uma mistura de tudo.
- Prefira [docs.flightphp.com](https://docs.flightphp.com) e o servidor MCP do Flight quando os agentes precisarem de detalhes da API; verifique métodos inventados em `vendor/flightphp/core`.

## Veja também

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Iniciador oficial com `AGENTS.md`, Twig, SimplePdo e Dice configurados para uma estrutura amigável à IA
- [Instalação](/install) – Layout recomendado com `create-project`
- [Autoload](/learn/autoloading) – As **maiúsculas/minúsculas** das pastas correspondem aos namespaces (`App\Controller` ↔ `app/Controller/`)
- [CLI Runway](/awesome-plugins/runway) – CLI que alimenta os comandos `ai:*` e de scaffolding
- [Segurança](/learn/security) – Padrões seguros que agentes (e humanos) não devem enfraquecer

## Solução de problemas

- Se aparecer "Missing .runway-creds.json", execute `php runway ai:init` primeiro.
- Certifique-se de que sua chave de API seja válida e tenha acesso ao modelo selecionado.
- Se as instruções não estiverem sendo atualizadas, verifique as permissões de arquivo no diretório do seu projeto.
- Se um agente inventar APIs do Flight ou o layout de pastas errado, aponte-o para o **`AGENTS.md`** raiz e para este site de documentação; o layout do esqueleto vence para código em `app/`.

## Changelog

- v3.18.4 – `ai:generate-instructions` grava as instruções do projeto em `AGENTS.md` na raiz do projeto.
- v3.16.0 – Adicionados os comandos de CLI `ai:init` e `ai:generate-instructions` para integração com IA.