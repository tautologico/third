+++
date = "2011-04-18T22:37:13-03:00"
title = "Um pouco de lamb(a)da"
tags = ["teoria da computação", "lambda-cálculo"]
+++

Hoje vi um
[post no blog da Caelum](http://blog.caelum.com.br/comecando-com-o-calculo-lambda-e-a-programacao-funcional-de-verdade/comment-page-1/)
sobre o lambda-cálculo (ou cálculo lambda) e, aproveitando que estou
ensinando programação funcional atualmente, achei interessante
escrever algumas ideias com a intenção de complementar o que está lá.

O lambda cálculo é um formalismo lógico usado hoje em dia
principalmente na teoria das linguagens de programação. Algumas
aplicações do lambda cálculo nessa área incluem o projeto e análise de
linguagens de programação e sistemas de tipos e a semântica das
linguagens de programação. Curiosamente,
[alguns linguistas também utilizam o cálculo](http://www.ling.gu.se/~lager/kurser/MathLing/kursplan.html)
para estudar a semântica das linguagens naturais. Além disso, o
cálculo também é usado em ramos da lógica pura e na teoria das funções
recursivas (que pode ser considerada parte da teoria da computação). O
cálculo é mais famoso pela grande influência que teve na definição do
paradigma de programação funcional.

Originalmente, Alonzo Church criou o lambda-cálculo como uma
ferramenta para estudar os fundamentos da matemática, coisa que estava
na moda no começo do século XX. A ideia de Church era usar a noção de
“processo” ou “transformação” (função) como essencial para fundamentar
a matemática, ao invés da noção de conjunto de Cantor. O lambda
cálculo não deu muito certo para isso na época, mas acabou sendo
importante em outra questão do tempo: a busca pela definição formal do
que vem a ser um
[procedimento efetivo](http://en.wikipedia.org/wiki/Effective_method). Em
termos atuais, diríamos que essa busca tentava definir formalmente o
que é “computação”.

(A ideia de usar o conceito de transformação como central na
matemática retornou na segunda metade do século XX através da Teoria
das Categorias, mas isso é outra história.)

Com relação ao cálculo em si, sua versão original (chamada hoje de
lambda-cálculo não-tipado) é muito simples. Essencialmente o cálculo é
formado por expressões-lambda e duas operações de transformação. As
expressões-lambda são sequências de símbolos formadas em uma sintaxe
específica, como expressões da lógica proposicional ou de uma
linguagem de programação. As operações transformam uma ou mais
expressões já existentes em uma nova expressão.

A sintaxe das expressões-lambda é determinada pelas duas operações:
abstração e aplicação (sendo que a aplicação envolve uma operação de
substituição chamada conversão-β). Uma expressão-lambda pode ser uma
variável, uma abstração de uma expressão, ou uma aplicação de duas
expressões:

* Variáveis: x, y, z, um conjunto qualquer de nomes de variáveis. Aqui
    usaremos letras minúsculas perto do final do alfabeto.
* Abstrações: dada uma expressão-lambda E, podemos formar uma
    abstração de E usando λ + variável + ‘.’ + E. Por exemplo: λx.x
* Aplicações: dadas duas expressões-lambda E e F, a expressão da
    aplicação é formada pela justaposição de uma ao lado da outra: E F

A sintaxe é apenas isso. Formalmente, é preciso definir essa sintaxe
mais cuidadosamente para garantir que uma expressão-lambda sempre pode
ser analisada de maneira não-ambígua, mas para os propósitos deste
post isso é suficiente. Alguns exemplos de expressões-lambda:

* x (Apenas uma variável)
* λx.x
* λx.y
* (λx.x x)(λx.x x)
* λm.λn.λa.λb. m (n b a) (n a b)

Agora vamos ao significado dessas coisas: a ideia é que abstração
signifique a criação de uma função, e aplicação signfique o uso ou
chamada dessa função em cima de um parâmetro. A conversão-β é a regra
de substituição que diz como a aplicação deve funcionar:

* Dada a aplicação (λx.E) F, o resultado é a expressão E, mas
    substituindo todas a ocorrências de x por F (o argumento da
    função).

Por exemplo: (λx.x) y tem como resultado a expressão y (E = x, F = y);
(λx.x)(λx.y) tem como resultado (λx.y) (E = x, F = (λx.y)). E é só
isso, substituição textual. No lambda-cálculo puro só podemos criar
expressões como definido acima. Mas vamos incluir algumas definições
adicionais como a função infixa de adição e números naturais como 2 e
3 e podemos ter expressões como (λx.x + 2) 3, que se for avaliada pela
conversão-β tem como resultado 3 + 2, exatamente o que esperamos da
aplicação de funções. (Indo mais longe no estudo do cálculo, vemos que
é possível definir os próprios números naturais e a operação de adição
no lambda-cálculo puro, mas não vamos passear por aí neste post; vide
a entrada na wikipedia pra
[numerais de Church](http://en.wikipedia.org/wiki/Church_numeral)).

A definição do lambda-calculus vista acima não inclui funções de dois
parâmetros, mas isso não é realmente necessário. Uma função de dois
argumentos pode ser criada simplesmente fazendo duas abstrações em
sequência: uma função usando a adição poderia ser definida como
λx.λy.x+y. Para aplicar essa função também temos que usar duas
aplicações: ((λx.λy.x+y) 3) 2 se transforma em (λy.3+y) 2, que se
transforma em 3+2\. Esse processo de considerar uma função de dois
argumentos como uma sequência de duas funções é o tal
[currying](http://pt.wikipedia.org/wiki/Currying) muito usado em
linguagens funcionais.

Após essa conversa toda, chegamos ao ponto em que dá para explicar em
mais detalhes a definição de valores-verdade no cálculo puro. Podemos
definir os valores _true_ e _false_ como:

* true = λx.λy.x
* false = λx.λy.y

A questão é: **por que** definir true e false dessa forma? A definição
é, em boa medida, arbitrária. Outras definições para os
valores-verdade são possíveis, mas essas acima são simples e
funcionam. Para ver que funcionam, basta pensar em como definir uma
expressão condicional (if-then-else) no cálculo. O requerimento básico
é:

* **if** E **then** F **else** G deve ser igual a F se E tiver valor
    true, e G se E tiver valor false

Para implementar isso no cálculo, dadas as definições de true e false
acima, basta usar aplicação diretamente. Dada uma expressão-lambda E
com resultado igual a um valor-verdade,

* if E then F else G = (E F) G

É fácil ver que isso funciona:

* (true F) G = ((λx.λy.x) F) G = (λy.F) G = F
* (false F) G = ((λx.λy.y) F) G = (λy.y) G = G

As outras definições (de pares, and, or, not, e mesmo as codificações
de números e aritmética no cálculo puro) seguem esse padrão. As
definições em si são arbitrárias, mas o importante é que elas
funcionem em um contexto maior.

O cálculo tem outras coisas muito interessantes além disso. O cálculo
não-tipado visto acima, que é basicamente um modelo de substituição, é
muito usado para estudar sistemas de macros em linguagens da família
Lisp, por exemplo. Uma outra dimensão interessante é ir para os
cálculos tipados e ver como os sistemas de tipos das linguagens de
programação podem ser formalizados (existe
[um livro inteiro sobre isso](http://www.cis.upenn.edu/~bcpierce/tapl/)). Os
combinadores da
[lógica combinatória](http://en.wikipedia.org/wiki/Combinatory_logic)
de Haskell Curry, um sistema formal muito relacionado ao
lambda-cálculo, também apresentam algumas ideias dignas de
investigação, como programação
[point-free](http://www.haskell.org/haskellwiki/Pointfree) (ou
tácita), a definição de
[bibliotecas de combinadores](http://en.wikipedia.org/wiki/Combinator_library),
e as técnicas para
[implementação de linguagens funcionais _lazy_](http://research.microsoft.com/en-us/um/people/simonpj/papers/slpj-book-1987/).

E, se você estudar e entender tudo isso, talvez você consiga entender
a linguagem [Scala](http://www.scala-lang.org) na sua totalidade :)
