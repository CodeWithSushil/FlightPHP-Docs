# Flight vs Slim

## O que é o Slim?
[Slim](https://slimframework.com) é um microframework PHP que ajuda você a escrever rapidamente aplicações web e APIs simples, porém poderosas.

Boa parte da inspiração para alguns dos recursos da v3 do Flight veio na verdade do Slim. Agrupar rotas e executar middleware em uma ordem específica são dois recursos que foram inspirados pelo Slim. O Slim v3 foi lançado com foco na simplicidade, mas houve [opiniões mistas](https://github.com/slimphp/Slim/issues/2770) em relação à v4.

## Prós em comparação ao Flight

- Slim tem uma comunidade maior de desenvolvedores, que por sua vez criam módulos úteis para ajudar você a não reinventar a roda.
- Slim segue muitas interfaces e padrões comuns na comunidade PHP, o que aumenta a interoperabilidade.
- Slim tem documentação e tutoriais decentes que podem ser usados para aprender o framework (nada comparado a Laravel ou Symfony, no entanto).
- Slim tem diversos recursos, como tutoriais no YouTube e artigos online, que podem ser usados para aprender o framework.
- Slim permite que você use quaisquer componentes que desejar para lidar com os recursos principais de roteamento, pois é compatível com PSR-7.

## Contras em comparação ao Flight

- Surpreendentemente, o Slim não é tão rápido quanto você imagina que seria para um microframework. Veja os 
  [benchmarks do TechEmpower](https://www.techempower.com/benchmarks/#hw=ph&test=fortune&section=data-r22&l=zik073-cn3) 
  para mais informações.
- Flight é voltado para um desenvolvedor que procura criar uma aplicação web leve, rápida e fácil de usar.
- Flight não tem dependências, enquanto o [Slim tem algumas dependências](https://github.com/slimphp/Slim/blob/4.x/composer.json) que você precisa instalar.
- Flight é voltado para simplicidade e facilidade de uso.
- Um dos recursos principais do Flight é que ele faz o possível para manter a compatibilidade com versões anteriores. A mudança do Slim v3 para v4 foi uma quebra de compatibilidade.
- Flight é destinado a desenvolvedores que estão se aventurando no mundo dos frameworks pela primeira vez.
- Flight também pode fazer aplicações de nível empresarial, mas não tem tantos exemplos e tutoriais quanto o Slim.
  Também exigirá mais disciplina por parte do desenvolvedor para manter as coisas organizadas e bem estruturadas.
- Flight dá ao desenvolvedor mais controle sobre a aplicação, enquanto o Slim pode introduzir alguma mágica nos bastidores.
- Flight tem [SimplePdo](/learn/simple-pdo) para acesso a banco de dados (preferido em relação ao PdoWrapper descontinuado). O Slim exige que você use uma biblioteca de terceiros.
- Flight tem um [plugin de permissões](/awesome-plugins/permissions) que pode ser usado para proteger sua aplicação. O Slim exige que você use uma biblioteca de terceiros.
- Flight tem um ORM chamado [active-record](/awesome-plugins/active-record) que pode ser usado para interagir com seu banco de dados. O Slim exige que você use uma biblioteca de terceiros.
- Flight tem uma aplicação CLI chamada [runway](/awesome-plugins/runway) que pode ser usada para executar sua aplicação a partir da linha de comando. O Slim não tem.