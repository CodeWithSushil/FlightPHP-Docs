# IA e Experiência do Desenvolvedor com Flight

## Visão Geral

Flight facilita a otimização dos seus projetos PHP com ferramentas alimentadas por IA e fluxos de trabalho modernos para desenvolvedores. Com comandos integrados para conectar provedores de LLM (Large Language Model) e gerar instruções de codificação específicas para o projeto, Flight ajuda você e sua equipe a aproveitarem ao máximo assistentes de IA como GitHub Copilot, Cursor, Windsurf e Antigravity (Gemini).

## Entendimento

Os assistentes de codificação com IA são mais úteis quando compreendem o contexto, as convenções e os objetivos do seu projeto. Os ajudantes de IA do Flight permitem que você:
- Conecte seu projeto a provedores populares de LLM (OpenAI, Grok, Claude, etc.)
- Gere e atualize instruções específicas do projeto para ferramentas de IA, para que todos recebam ajuda consistente e relevante
- Mantenha sua equipe alinhada e produtiva, com menos tempo gasto explicando o contexto

Esses recursos estão integrados no CLI principal do Flight e no projeto inicial oficial [flightphp/skeleton](https://github.com/flightphp/skeleton).

## Uso Básico

### Configurando Credenciais de LLM

O comando `ai:init` orienta você na conexão do seu projeto a um provedor de LLM.

```bash
php runway ai:init
```

Você será solicitado a:
- Escolher seu provedor (OpenAI, Grok, Claude, etc.)
- Inserir sua chave de API
- Definir a URL base e o nome do modelo

Isso cria as credenciais necessárias para que você possa fazer solicitações futuras de LLM.

**Exemplo:**
```
Bem-vindo ao AI Init!
Qual API de LLM você deseja usar? [1] openai, [2] grok, [3] claude: 1
Digite a URL base para a API de LLM [https://api.openai.com]:
Digite sua chave de API para openai: sk-...
Digite o nome do modelo que você deseja usar (ex: gpt-4, claude-3-opus, etc) [gpt-4o]:
Credenciais salvas em .runway-creds.json
```

### Gerando Instruções de IA Específicas do Projeto

O comando `ai:generate-instructions` ajuda você a criar ou atualizar instruções para assistentes de codificação com IA, personalizadas para o seu projeto.

```bash
php runway ai:generate-instructions
```

Você responderá algumas perguntas sobre seu projeto (descrição, banco de dados, templates, segurança, tamanho da equipe, etc.). Flight usa seu provedor de LLM para gerar instruções e, em seguida, grava o mesmo conteúdo em:
- `.github/copilot-instructions.md` (para GitHub Copilot)
- `.cursor/rules/project-overview.mdc` (para Cursor)
- `.windsurfrules` (para Windsurf)
- `.gemini/GEMINI.md` (para Antigravity)
- `AGENTS.md` (na raiz do projeto, para assistentes de IA independentes de ferramentas)

**Exemplo:**
```
Por favor, descreva para que serve o seu projeto? Minha API incrível
Qual banco de dados você pretende usar? MySQL
Qual mecanismo de template HTML você pretende usar (se houver)? latte
A segurança é um elemento importante neste projeto? (s/n) s
...
Instruções de IA atualizadas com sucesso.
```

Agora, suas ferramentas de IA darão sugestões mais inteligentes e relevantes com base nas necessidades reais do seu projeto.

## Uso Avançado

- Você pode personalizar a localização dos seus arquivos de credenciais ou instruções usando opções de comando (veja `--help` para cada comando).
- Os ajudantes de IA são projetados para funcionar com qualquer provedor de LLM que suporte APIs compatíveis com OpenAI.
- Se você quiser atualizar suas instruções conforme seu projeto evolui, basta executar novamente `ai:generate-instructions` e responder aos prompts novamente.

## Veja Também

- [Flight Skeleton](https://github.com/flightphp/skeleton) – O inicializador oficial com integração de IA
- [Runway CLI](/awesome-plugins/runway) – Mais sobre a ferramenta CLI que alimenta esses comandos

## Solução de Problemas

- Se você vir "Missing .runway-creds.json", execute `php runway ai:init` primeiro.
- Certifique-se de que sua chave de API seja válida e tenha acesso ao modelo selecionado.
- Se as instruções não estiverem sendo atualizadas, verifique as permissões de arquivo no diretório do seu projeto.

## Changelog

- v3.18.4 – `ai:generate-instructions` agora também grava instruções do projeto em `AGENTS.md` na raiz do projeto.
- v3.16.0 – Adicionados comandos CLI `ai:init` e `ai:generate-instructions` para integração com IA.