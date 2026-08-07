# Flight vs Slim

## ¿Qué es Slim?
[Slim](https://slimframework.com) es un microframework de PHP que te ayuda a escribir rápidamente aplicaciones web y APIs simples pero potentes.

Gran parte de la inspiración para algunas de las características de la v3 de Flight provino realmente de Slim. Agrupar rutas y ejecutar middleware en un orden específico son dos características inspiradas en Slim. Slim v3 se lanzó orientado a la simplicidad, pero ha habido [reseñas mixtas](https://github.com/slimphp/Slim/issues/2770) con respecto a la v4.

## Ventajas en comparación con Flight

- Slim tiene una comunidad más grande de desarrolladores, que a su vez crean módulos útiles para ayudarte a no reinventar la rueda.
- Slim sigue muchas interfaces y estándares comunes en la comunidad de PHP, lo que aumenta la interoperabilidad.
- Slim tiene documentación y tutoriales decentes que se pueden usar para aprender el framework (aunque nada comparado con Laravel o Symfony).
- Slim tiene varios recursos como tutoriales de YouTube y artículos en línea que se pueden usar para aprender el framework.
- Slim te permite usar los componentes que quieras para manejar las características principales de enrutamiento, ya que es compatible con PSR-7.

## Desventajas en comparación con Flight

- Sorprendentemente, Slim no es tan rápido como cabría esperar para un microframework. Consulta los [benchmarks de TechEmpower](https://www.techempower.com/benchmarks/#hw=ph&test=fortune&section=data-r22&l=zik073-cn3) para más información.
- Flight está orientado a un desarrollador que busca construir una aplicación web ligera, rápida y fácil de usar.
- Flight no tiene dependencias, mientras que [Slim tiene algunas dependencias](https://github.com/slimphp/Slim/blob/4.x/composer.json) que debes instalar.
- Flight está orientado a la simplicidad y la facilidad de uso.
- Una de las características principales de Flight es que hace todo lo posible por mantener la compatibilidad hacia atrás. Slim v3 a v4 fue un cambio radical.
- Flight está pensado para desarrolladores que se aventuran por primera vez en el mundo de los frameworks.
- Flight también puede manejar aplicaciones a nivel empresarial, pero no tiene tantos ejemplos y tutoriales como Slim. También requerirá más disciplina por parte del desarrollador para mantener las cosas organizadas y bien estructuradas.
- Flight le da al desarrollador más control sobre la aplicación, mientras que Slim puede introducir algo de magia detrás de escena.
- Flight tiene [SimplePdo](/learn/simple-pdo) para el acceso a bases de datos (preferido sobre el obsoleto PdoWrapper). Slim requiere que uses una biblioteca de terceros.
- Flight tiene un [plugin de permisos](/awesome-plugins/permissions) que se puede usar para asegurar tu aplicación. Slim requiere que uses una biblioteca de terceros.
- Flight tiene un ORM llamado [active-record](/awesome-plugins/active-record) que se puede usar para interactuar con tu base de datos. Slim requiere que uses una biblioteca de terceros.
- Flight tiene una aplicación CLI llamada [runway](/awesome-plugins/runway) que se puede usar para ejecutar tu aplicación desde la línea de comandos. Slim no la tiene.