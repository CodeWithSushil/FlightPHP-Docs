# Flight vs Fat-Free

## O que é o Fat-Free?
[Fat-Free](https://fatfreeframework.com) (carinhosamente conhecido como **F3**) é um micro-framework PHP poderoso e fácil de usar, projetado para ajudá-lo a construir aplicações web dinâmicas e robustas - rápido!

O Flight se compara ao Fat-Free de várias maneiras e provavelmente é o primo mais próximo em termos de recursos e simplicidade. O Fat-Free tem muitos recursos que o Flight não tem, mas também tem muitos recursos que o Flight tem. O Fat-Free está começando a mostrar sua idade e não é mais tão popular quanto já foi.

As atualizações estão se tornando menos frequentes e a comunidade não está tão ativa quanto antes. O código é simples o suficiente, mas às vezes a falta de disciplina na sintaxe pode dificultar a leitura e o entendimento. Ele funciona com PHP 8.3, mas o código em si ainda parece que vive em PHP 5.3.

## Prós em comparação ao Flight

- O Fat-Free tem algumas estrelas a mais no GitHub do que o Flight.
- O Fat-Free tem uma documentação razoável, mas deixa a desejar em algumas áreas com clareza.
- O Fat-Free tem alguns recursos escassos, como tutoriais no YouTube e artigos online que podem ser usados para aprender o framework.
- O Fat-Free tem [alguns plugins úteis](https://fatfreeframework.com/3.8/api-reference) integrados que às vezes são úteis.
- O Fat-Free tem um ORM integrado chamado Mapper que pode ser usado para interagir com seu banco de dados. O Flight tem [active-record](/awesome-plugins/active-record).
- O Fat-Free tem Sessões, Cache e localização integrados. O Flight exige que você use bibliotecas de terceiros, mas isso é coberto na [documentação](/awesome-plugins).
- O Fat-Free tem um pequeno grupo de [plugins criados pela comunidade](https://fatfreeframework.com/3.8/development#Community) que podem ser usados para estender o framework. O Flight tem alguns cobertos na página de [documentação](/awesome-plugins) e de [exemplos](/examples).
- O Fat-Free, assim como o Flight, não tem dependências.
- O Fat-Free, assim como o Flight, é voltado para dar ao desenvolvedor controle sobre sua aplicação e uma experiência de desenvolvimento simples.
- O Fat-Free mantém compatibilidade reversa como o Flight faz (parcialmente porque as atualizações estão se tornando [menos frequentes](https://github.com/bcosca/fatfree/releases)).
- O Fat-Free, assim como o Flight, é destinado a desenvolvedores que estão se aventurando no mundo dos frameworks pela primeira vez.
- O Fat-Free tem um mecanismo de template integrado que é mais robusto que o mecanismo de template do Flight. O Flight recomenda [Latte](/awesome-plugins/latte) para conseguir isso.
- O Fat-Free tem um comando CLI exclusivo do tipo "rota" onde você pode construir aplicativos CLI dentro do próprio Fat-Free e tratá-lo muito como uma requisição `GET`. O Flight consegue isso com [runway](/awesome-plugins/runway).

## Contras em comparação ao Flight

- O Fat-Free tem alguns testes de implementação e até possui sua própria classe de [teste](https://fatfreeframework.com/3.8/test) que é muito básica. No entanto,
  não é 100% testado por unidade como o Flight é. 
- Você tem que usar um mecanismo de busca como o Google para realmente pesquisar no site da documentação.
- O Flight tem modo escuro no site da documentação. (deixa o microfone cair)
- O Fat-Free tem alguns módulos que estão lamentavelmente sem manutenção.
- O Flight tem [SimplePdo](/learn/simple-pdo) para acesso a banco de dados, que é um pouco mais simples que a classe `DB\SQL` integrada do Fat-Free (e preferido em relação ao PdoWrapper descontinuado).
- O Flight tem um [plugin de permissões](/awesome-plugins/permissions) que pode ser usado para proteger sua aplicação. O Fat Free exige que você use 
  uma biblioteca de terceiros.
- O Flight tem um ORM chamado [active-record](/awesome-plugins/active-record) que parece mais um ORM do que o Mapper do Fat-Free.
  O benefício adicional do `active-record` é que você pode definir relacionamentos entre registros para junções automáticas, enquanto o Mapper do Fat-Free
  exige que você crie [views SQL](https://fatfreeframework.com/3.8/databases#ProsandCons).
- Surpreendentemente, o Fat-Free não tem um namespace raiz. O Flight tem namespace de ponta a ponta para não colidir com seu próprio código.
  a classe `Cache` é a maior ofensora aqui.
- O Fat-Free não tem middleware. Em vez disso, existem hooks `beforeroute` e `afterroute` que podem ser usados para filtrar requisições e respostas nos controladores.
- O Fat-Free não pode agrupar rotas.
- O Fat-Free tem um manipulador de contêiner de injeção de dependência, mas a documentação é incrivelmente escassa sobre como usá-lo.
- A depuração pode ficar um pouco complicada, pois basicamente tudo é armazenado no que é chamado de [`HIVE`](https://fatfreeframework.com/3.8/quick-reference)