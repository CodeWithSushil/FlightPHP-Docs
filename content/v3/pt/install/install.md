# Instruções de Instalação

Existem alguns pré-requisitos básicos antes de poder instalar o Flight. Nomeadamente, você precisará de:

1. [Instalar PHP no seu sistema](#installing-php)
2. [Instalar Composer](https://getcomposer.org) para a melhor experiência de desenvolvimento.

## Instalação Básica

Se você estiver usando o [Composer](https://getcomposer.org), execute o seguinte comando:

```bash
composer require flightphp/core
```

Isso colocará apenas os arquivos principais do Flight no seu sistema. Você precisará definir a estrutura do projeto, [layout](/learn/templates), [dependências](/learn/dependency-injection-container), [configurações](/learn/configuration), [autoloading](/learn/autoloading), etc. Este método garante que nenhuma outra dependência além do Flight seja instalada.

Você também pode [baixar os arquivos](https://github.com/flightphp/core/archive/master.zip) diretamente e extraí-los para o seu diretório web.

A instalação básica é perfeita para aprendizado, micro APIs e experimentos de copiar e colar. Para um layout de aplicativo completo que humanos *e* [ferramentas de codificação de IA](/learn/ai) possam seguir da mesma forma, use o esqueleto recomendado abaixo.

## Instalação Recomendada

É altamente recomendável começar com o aplicativo [flightphp/skeleton](https://github.com/flightphp/skeleton) para qualquer novo projeto. A instalação é muito fácil.

```bash
composer create-project flightphp/skeleton my-project/
cd my-project/
composer start
# banco de dados de exemplo opcional + demonstração de posts
php runway migrate
```

Essa etapa configura a estrutura do projeto, o autoloading PSR-4 do Composer, a configuração e ferramentas como [Tracy](/awesome-plugins/tracy), [Extensões do Tracy](/awesome-plugins/tracy-extensions) e [Runway](/awesome-plugins/runway). Ela também inclui o **`AGENTS.md`** na raiz (e cópias com escopo em `app/`) para que os assistentes de IA compartilhem um layout com você — veja [IA e experiência de desenvolvimento](/learn/ai).

### O que o esqueleto oferece

```text
project-root/
├── AGENTS.md              # Fonte de verdade para IA / agente
├── SECURITY.md            # Expectativas de segurança
├── .env.example           # Segredos / overlays de deploy (copiado para .env)
├── public/index.php       # Apenas entrada web
├── app/
│   ├── config/            # bootstrap, rotas, serviços, config_sample.php
│   ├── Controller/        # App\Controller\*  (pasta PascalCase!)
│   ├── Middleware/        # App\Middleware\*
│   ├── Model/             # App\Model\* (ActiveRecord)
│   ├── Utils/             # Config, Env, DatabaseFactory
│   ├── commands/          # Comandos CLI do Runway
│   ├── views/             # Templates Twig (*.twig)
│   ├── cache/
│   └── log/
├── migrations/            # Migrações SQL (.sql / .mysql.sql)
└── tests/                 # PHPUnit
```

**Namespaces seguem o caso das pastas.** O Composer mapeia `"App\\": "app/"`, então:

| Caminho no disco | Namespace |
|--------------|-----------|
| `app/Controller/HomeController.php` | `App\Controller\HomeController` |
| `app/Middleware/…` | `App\Middleware\…` |
| `app/Model/…` | `App\Model\…` |
| `app/Utils/…` | `App\Utils\…` |

No Linux, `app/controller/` **não** é o mesmo que `app/Controller/`. O autoloading diferencia maiúsculas de minúsculas — corresponda às pastas PascalCase do esqueleto. Detalhes: [Autoloading](/learn/autoloading).

**Padrões da stack (novos projetos):** views Twig, SimplePdo + ActiveRecord, Dice com injeção de `Engine` (prefira não usar `Flight::` dentro das classes do aplicativo), SQLite opcional após `php runway migrate`.

O `create-project` normalmente copia `app/config/config_sample.php` → `config.php` e `.env.example` → `.env` quando presentes. As rotas ficam em `app/config/routes.php`; serviços e DI (injeção de dependências) ficam em `app/config/services.php`.

> **Documentação ↔ esqueleto:** Estes documentos ensinam as **APIs** do Flight (muitas vezes com exemplos curtos de `Flight::`). O esqueleto define a **estrutura do aplicativo**. Ao adicionar código em `app/`, siga a árvore do esqueleto; use a documentação para nomes de métodos, opções e plugins.

## Configure seu Servidor Web

### Servidor de Desenvolvimento Embutido do PHP

Esta é de longe a maneira mais simples de começar. Você pode usar o servidor embutido para executar seu aplicativo e até mesmo usar SQLite como banco de dados (desde que o sqlite3 esteja instalado no seu sistema) sem precisar de quase nada! Basta executar o seguinte comando assim que o PHP estiver instalado:

```bash
php -S localhost:8000
# ou com o aplicativo esqueleto
composer start
```

Em seguida, abra seu navegador e acesse `http://localhost:8000`.

Se você quiser definir o diretório raiz de documentos do seu projeto como um diretório diferente (Ex: seu projeto é `~/myproject`, mas sua raiz de documentos é `~/myproject/public/`), você pode executar o seguinte comando estando no diretório `~/myproject`:

```bash
php -S localhost:8000 -t public/
# com o aplicativo esqueleto, isso já está configurado
composer start
```

Em seguida, abra seu navegador e acesse `http://localhost:8000`.

### Apache

Certifique-se de que o Apache já esteja instalado no seu sistema. Se não estiver, pesquise no Google como instalar o Apache no seu sistema.

Para o Apache, edite seu arquivo `.htaccess` com o seguinte:

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **Nota**: Se você precisar usar o Flight em um subdiretório, adicione a linha
> `RewriteBase /subdir/` logo após `RewriteEngine On`.

> **Nota**: Se você quiser proteger todos os arquivos do servidor, como um arquivo de banco de dados ou env.
> Coloque isto no seu arquivo `.htaccess`:

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

Certifique-se de que o Nginx já esteja instalado no seu sistema. Se não estiver, pesquise no Google como instalar o Nginx no seu sistema.

Para o Nginx, adicione o seguinte à sua declaração de servidor:

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## Crie seu arquivo `index.php`

Se você estiver fazendo uma instalação básica, precisará de algum código para começar.

```php
<?php

// Se você estiver usando o Composer, requisite o autoloader.
require 'vendor/autoload.php';
// se você não estiver usando o Composer, carregue o framework diretamente
// require 'flight/Flight.php';

// Em seguida, defina uma rota e atribua uma função para lidar com a solicitação.
Flight::route('/', function () {
  echo 'hello world!';
});

// Finalmente, inicie o framework.
Flight::start();
```

Com o aplicativo esqueleto, a entrada pública apenas inicializa o aplicativo. As rotas são registradas em `app/config/routes.php` (normalmente `[App\Controller\…::class, 'method']` para que o Dice possa injetar dependências). Serviços, Twig, SimplePdo e o contêiner estão conectados em `app/config/services.php`. Essa estrutura é intencional para que ferramentas de IA e humanos editem os mesmos lugares todas as vezes.

## Instalando PHP

Se você já tem `php` instalado no seu sistema, pode pular estas instruções e ir para [a seção de download](#download-the-files).

### **macOS**

#### **Instalando o PHP usando Homebrew**

1. **Instale o Homebrew** (se ainda não estiver instalado):
   - Abra o Terminal e execute:
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **Instale o PHP**:
   - Instale a versão mais recente:
     ```bash
     brew install php
     ```
   - Para instalar uma versão específica, por exemplo, PHP 8.1:
     ```bash
     brew tap shivammathur/php
     brew install shivammathur/php/php@8.1
     ```

3. **Alterne entre versões do PHP**:
   - Desvincule a versão atual e vincule a versão desejada:
     ```bash
     brew unlink php
     brew link --overwrite --force php@8.1
     ```
   - Verifique a versão instalada:
     ```bash
     php -v
     ```

### **Windows 10/11**

#### **Instalando o PHP manualmente**

1. **Baixe o PHP**:
   - Visite [PHP para Windows](https://windows.php.net/download/) e baixe a versão mais recente ou uma versão específica (ex.: 7.4, 8.0) como um arquivo zip não thread-safe.

2. **Extraia o PHP**:
   - Extraia o arquivo zip baixado para `C:\php`.

3. **Adicione o PHP ao PATH do sistema**:
   - Vá em **Propriedades do Sistema** > **Variáveis de Ambiente**.
   - Em **Variáveis do sistema**, encontre **Path** e clique em **Editar**.
   - Adicione o caminho `C:\php` (ou onde você extraiu o PHP).
   - Clique em **OK** para fechar todas as janelas.

4. **Configure o PHP**:
   - Copie `php.ini-development` para `php.ini`.
   - Edite o `php.ini` para configurar o PHP conforme necessário (ex.: definindo `extension_dir`, habilitando extensões).

5. **Verifique a instalação do PHP**:
   - Abra o Prompt de Comando e execute:
     ```cmd
     php -v
     ```

#### **Instalando Múltiplas Versões do PHP**

1. **Repita os passos acima** para cada versão, colocando cada uma em um diretório separado (ex.: `C:\php7`, `C:\php8`).

2. **Alterne entre as versões** ajustando a variável PATH do sistema para apontar para o diretório da versão desejada.

### **Ubuntu (20.04, 22.04, etc.)**

#### **Instalando o PHP usando apt**

1. **Atualize as listas de pacotes**:
   - Abra o Terminal e execute:
     ```bash
     sudo apt update
     ```

2. **Instale o PHP**:
   - Instale a versão mais recente do PHP:
     ```bash
     sudo apt install php
     ```
   - Para instalar uma versão específica, por exemplo, PHP 8.1:
     ```bash
     sudo apt install php8.1
     ```

3. **Instale módulos adicionais** (opcional):
   - Por exemplo, para instalar suporte ao MySQL:
     ```bash
     sudo apt install php8.1-mysql
     ```

4. **Alterne entre versões do PHP**:
   - Use `update-alternatives`:
     ```bash
     sudo update-alternatives --set php /usr/bin/php8.1
     ```

5. **Verifique a versão instalada**:
   - Execute:
     ```bash
     php -v
     ```

### **Rocky Linux**

#### **Instalando o PHP usando yum/dnf**

1. **Habilite o repositório EPEL**:
   - Abra o Terminal e execute:
     ```bash
     sudo dnf install epel-release
     ```

2. **Instale o repositório Remi**:
   - Execute:
     ```bash
     sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
     sudo dnf module reset php
     ```

3. **Instale o PHP**:
   - Para instalar a versão padrão:
     ```bash
     sudo dnf install php
     ```
   - Para instalar uma versão específica, por exemplo, PHP 7.4:
     ```bash
     sudo dnf module install php:remi-7.4
     ```

4. **Alterne entre versões do PHP**:
   - Use o comando de módulo `dnf`:
     ```bash
     sudo dnf module reset php
     sudo dnf module enable php:remi-8.0
     sudo dnf install php
     ```

5. **Verifique a versão instalada**:
   - Execute:
     ```bash
     php -v
     ```

### **Notas Gerais**

- Para ambientes de desenvolvimento, é importante configurar as configurações do PHP de acordo com os requisitos do seu projeto.
- Ao alternar versões do PHP, certifique-se de que todas as extensões PHP relevantes estejam instaladas para a versão específica que você pretende usar.
- Reinicie seu servidor web (Apache, Nginx, etc.) após alternar versões do PHP ou atualizar configurações para aplicar as alterações.