+++
author = "Andrei Formiga"
date = "2016-04-19T17:31:49-03:00"
description = "description"
draft = true
tags = ["lógica"]
title = "Representação de Fórmulas da Lógica"
Type = "post"
categories = ["programação"]
series = ["Tabela-verdade em C"]
+++

Agora que já vimos [o quão fácil é calcular a tabela-verdade para uma fórmula
fixa](/post/tabverdc), vamos à parte menos fácil, que é calcular a
tabela-verdade para uma fórmula arbitrária entrada pelo usuário.

Para que o programa aceite e entenda uma fórmula qualquer entrada pelo usuário,
é preciso implementar duas etapas:

1. A fórmula deve ser traduzida do formato de entrada para uma representação
interna usada pelo programa (essa etapa é chamada de _análise sintática_);
2. A representação interna deve ser processada, o que nesse caso significa
calcular os valores da fórmula para a tabela-verdade.

Por quê traduzir do formato da entrada para uma representação interna? É natural,
já que os dois formatos têm funções diferentes e um formato que é bom para uso
humano dificilmente vai ser bom também para processamento pela máquina.

Um aspecto que é comum às duas etapas do processo é a representação interna
da fórmula, ou seja, as estruturas de dados que serão utilizadas para
representar as fórmulas no programa. Este texto se concentra principalmente
nesta representação, mas também trata da segunda etapa: uma vez tendo a
fórmula representada por uma estrutura de dados fácil de processar, podemos
aproveitar e calcular logo os valores da fórmula para obter a sua
tabela-verdade.

A representação que vamos usar para as fórmulas são as árvores sintáticas.
Essa é uma escolha comum para aplicações
que lidam com linguagens (como compiladores de linguagens de programação).

### Árvores Sintáticas

No [programa que calcula a tabela-verdade de uma fórmula fixa no código](/post/tabverdc),
esse cálculo é feito pela função

~~~c
int valor_formula()
{
  return IMP(I[P], I[Q]) && ((!I[Q]) || I[R]);
}
~~~

que calcula o valor da fórmula

<div>
$$(P \rightarrow Q) \wedge (\neg Q \vee R)$$
</div>

dado o valor das variáveis proposicionais P, Q e R (obtido através do
_array_ I[]).

Isso obviamente não vai funcionar para uma fórmula qualquer, à escolha
do usuário do programa. Precisamos criar uma função que calcula o valor
de uma fórmula passada como parâmetro. Algo como

~~~c
int valor_formula(Formula *f) { /* ... */ }
~~~

A questão da representação é justamente como deve ser o tipo `Formula`
para capturar qualquer fórmula da lógica.

Uma opção boa para isso são as árvores sintáticas, muito usadas em
compiladores e outros processadores de linguagens. (Mais especificamente,
vamos usar árvores de sintaxe _abstrata_, qualificação que será explicada
mais tarde, quando falarmos de análise sintática).

Expressões aritméticas fornecem um exemplo simples para entender as
árvores sintáticas. Geralmente escrevemos e lemos expressões como
uma sequência simples de símbolos como (2 + 3) * 4. Na verdade,
expressões possuem estrutura hierárquica e recursiva, pois podemos
ter uma sub-expressão completa como (2 + 3) dentro da expressão maior.
O uso de parênteses é uma mostra das adaptações que são necessárias
para representar a estrutura hierárquica das expressões em uma
sequência linear de símbolos.
