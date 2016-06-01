+++
date = "2016-03-07T19:06:05-03:00"
title = "Calculando Tabelas-Verdade em C -- Fórmulas fixas"
tags = ["lógica"]
categories = ["programação"]
series = ["Tabela-verdade em C"]
+++

Quando estou ensinando Lógica Aplicada à Computação, eu muitas vezes digo
aos alunos que é simples criar um programa que calcula a tabela-verdade de uma
fórmula da lógica proposicional. E o cálculo da tabela em si é realmente simples,
o problema é como o programa vai representar as fórmulas da lógica. Ou seja, para
calcular a tabela-verdade de qualquer fórmula, talvez escolhida pelo usuário, o
programa deve ser capaz de representar qualquer fórmula da lógica proposicional.
Isso não é difícil para quem já tem alguma familiaridade com processadores de
linguagens, como compiladores e interpretadores, mas pode ser muito novo para
quem ainda está no começo do curso de Computação (a disciplina de Lógica é
obrigatória no segundo período).

Neste texto eu vou evitar o problema da representação de fórmulas mostrando
como calcular a tabela-verdade para uma fórmula fixa. Para cada
fórmula que se deseje obter a tabela-verdade, é preciso mudar o código. Isso
está bem longe do ideal, mas é suficiente para mostrar o cálculo da tabela
em si, sem se preocupar com outras questões. Outros textos nesta série vão
explorar a representação de fórmulas e uma sintaxe externa para que
o usuário possa especificar para qual fórmula ele quer calcular uma
tabela-verdade.

Antes de começar, um preview. A fórmula que vamos usar como exemplo é

<div>
$$(P \rightarrow Q) \wedge (\neg Q \vee R)$$
</div>

e o código que vai calcular o valor desta fórmula é

~~~c
IMP(I[P], I[Q]) && ((!I[Q]) || I[R])
~~~

onde `IMP` é uma macro que calcula o valor da implicação. Este código
é uma tradução transparente da fórmula acima, tirando pelo fato
que não existe o operador de implicação em C, e que para calcular o
valor da fórmula usamos o valor da função interpretação de cada
variável proposicional, `I[]`, ao invés da variável em si.

Fora o cálculo da fórmula em si, as questões mais importantes são
a representação dos valores de cada variável proposicional, e como
variar esses valores de maneira organizada para gerar a tabela. A
impressão da tabela também é importante mas é simples.

Antes de falar sobre os detalhes do código, vamos definir os termos
e conceitos utilizados.

### Lógica proposicional e tabelas-verdade

A lógica proposicional lida com fórmulas que são formadas pela
combinação de variáveis proposicionais usando um conjunto de conectivos.
As variáveis proposicionais representam proposições, originalmente
criadas para modelar frases afirmativas que podem ser avaliadas como
verdadeiras ou falsas; por exemplo, "está chovendo" pode ser uma
proposição. Na verdade, proposições podem modelar várias coisas
que sempre estão em um de dois estados possíveis: verdadeiro ou
falso, 0 ou 1, corrente baixa ou corrente alta, etc.

Os conectivos geralmente usados na lógica proposicional são cinco:

* conjunção ou E-lógico, símbolo ∧
* disjunção ou OU-lógico, símbolo ∨
* negação, símbolo ¬
* implicação ou condicional, símbolo →
* bi-implicação ou bicondicional, símbolo ↔

Para determinar o valor de verdade de uma fórmula como

<div>
$$P \wedge \neg Q$$
</div>

é preciso conhecer o valor de verdade das variáveis proposicionais
que fazem parte da fórmula (no exemplo, P e Q); dado isto, as regras
dos conectivos são fixas e permitem calcular o valor da fórmula.
Formalmente, é comum definir uma função interpretação ou função
valoração que mapeia cada variável proposicional para um valor
de verdade, e a partir disto obter o valor da fórmula. Usaremos
como valores de verdade os símbolos T e F.

Uma tabela-verdade é uma maneira de listar todos os possíveis
valores de uma fórmula, de acordo com cada possível combinação
de valores de verdade para as variáveis proposicionais. Cada
linha de uma tabela-verdade lista uma possível função interpretação
para as variáveis, e o valor da fórmula para esta interpretação.

Por exemplo, para uma fórmula contendo apenas uma implicação, a
tabela-verdade é:

<div>
$$\begin{array}{cc|c}
  P & Q & P \rightarrow Q \\
  \hline
  F & F & T \\
  F & T & T \\
  T & F & F \\
  T & T & T
\end{array}$$
</div>

Essa tabela especifica completamente o comportamento do conectivo
implicação. Vamos padronizar a ordem das linhas da tabela começando
com todas as variáveis com valor F e terminando com todas as
variáveis com valor T, como na tabela anterior. Outra forma de ver
essa ordem é pensar que o valor F é um zero e um valor T é um 1, e
que seguimos a ordem dos números binários: 00, 01, 10, 11 para duas
variáveis proposicionais.

No programa em C, representamos F pelo valor 0 e T pelo valor 1,
como é usual nesta linguagem. Também usamos as constantes simbólicas
`TRUE` e `FALSE`.

Agora podemos discutir o programa que calcula a tabela-verdade,
começando pela representação das fórmulas da lógica. Para quem
quiser acompanhar a discussão com o código completo, o [programa
está disponível aqui](https://gist.github.com/tautologico/ae06d2587eece93e184b).

### Representação das variáveis

Como vimos, a fórmula será representada diretamente em código C,
o que é possível porque o programa só tem a capacidade de
calcular essa única fórmula. As variáveis proposicionais são
representadas por dois arrays, um para os nomes e
outro para os valores das variáveis. Definimos também uma constante
simbólica para guardar o número de variáveis da fórmula:

~~~c
// numero de variaveis proposicionais na formula
#define VARS              3

// representacao da formula
char nome[VARS];     // nome das variaveis
int I[VARS];         // interpretacao das variaveis
~~~

O nome das variáveis é usado apenas na impressão da tabela. O
array `I[]` representa a função interpretação que dá os valores
das variáveis proposicionais (T ou F); usei o `I` maiúsculo
por ser similar à notação de alguns livros de lógica para a
função interpretação. Os valores são representados como
valores inteiros devido à convenção de usar 0 para o valor
F e 1 para o valor T (ou V).

Como a fórmula é fixa e só usa as variáveis P, Q e R, a intenção
é que a variável 0 seja P, a variável 1 seja Q e
a variável 2 seja R. Para facilitar a indexação dos arrays
`nome[]` e `I[]`, definimos as seguintes constantes:

~~~c
// constantes para os indices das tres variaveis
#define P                 0
#define Q                 1
#define R                 2
~~~

Assim, `I[P]` representa o valor da variável P, como vemos
em alguns livros. O núcleo do código que calcula a tabela-verdade é
um loop que varia os valores das variáveis de maneira a explorar
todas as possíveis interpretações que fazem parte da tabela,
e imprimir o valor resultante da fórmula para cada interpretação.

### Loop principal

A função `main` do programa apenas chama a função `mostra_tabela()`,
que contém o loop principal do programa. Essa função começa assim:

~~~c
void mostra_tabela()
{
  int fim = FALSE;

  inicializa_formula();

  printf("Formula:\n");
  printf("H = (P -> Q) /\\ (~Q \\/ R)\n\n");

  for (int c = 0; c < VARS; c++) {
    printf(" %c |", nome[c]);
  }
  printf(" H\n");

  for (int c = 0; c < 4 * VARS + 3; c++)
    printf("-");
  printf("\n");
~~~

Esse começo inicializa os dois arrays que representam as variáveis
(chamando `inicializa_formula()`) e imprime o cabeçalho da tabela,
mostrando também a fórmula. A inicialização é simples, estabelecendo
os nomes das variáveis e os seus valores iniciais, com todas
tendo valor F (representado em C como 0 ou a constante `FALSE`).

~~~c
void inicializa_formula()
{
  nome[P] = 'P';
  nome[Q] = 'Q';
  nome[R] = 'R';

  for (int c = 0; c < VARS; c++)
    I[c] = FALSE;
}
~~~

Voltando à função `mostra_tabela()`, o loop principal
tem o seguinte formato:

~~~c
while (!fim) {
  // imprime valores atuais das variaveis

  // calcula e imprime o valor da formula

  // verifica se acabou a tabela ou passa para
  // a proxima linha
}
~~~

No código, isso fica

~~~c
  while (!fim) {
    // imprime valores atuais das variaveis
    for (int c = 0; c < VARS; c++) {
      if (I[c])
        printf(" T |");
      else
        printf(" F |");
    }

    // calcula e imprime o valor da formula
    if (valor_formula())
      printf(" T\n");
    else
      printf(" F\n");

    // verifica se acabou a tabela ou passa para
    // a proxima linha
    if (ultima_interpretacao())
      fim = TRUE;
    else
      proxima_interpretacao();
  }
~~~

A impressão dos valores das variáveis é um loop que imprime
T ou F dependendo do valor de cada uma. Para calcular o valor
da fórmula é chamada a função `valor_formula()`, cuja parte
principal já vimos antes:

~~~c
int valor_formula()
{
  return IMP(I[P], I[Q]) && ((!I[Q]) || I[R]);
}
~~~

A macro `IMP` calcula o valor do conectivo implicação usando
o operador ternário:

~~~c
#define IMP(b1, b2)       (b1 && !b2 ? FALSE : TRUE)
~~~

Como vimos anteriormente, para uma fórmula P -> Q, o conectivo
implicação da lógica proposicional
tem valor F apenas quando P tiver valor T e Q tiver valor F. O teste
verifica se esse é o caso e retorna F se for, e T caso contrário.

Voltando ao loop principal, a última parte é

~~~c
    if (ultima_interpretacao())
      fim = TRUE;
    else
      proxima_interpretacao();
~~~

Se os valores da variáveis correspondem à última interpretação na tabela,
o loop é finalizado (a variável `fim` é a condição do loop). Caso contrário,
os valores devem ser alterados para a próxima linha da tabela.

A função `ultima_interpretacao()` faz o teste que determina se os valores
atuais das variáveis correspondem à última linha da tabela:

~~~c
// retorna TRUE se a interpretacao atual eh a ultima na tabela-verdade
int ultima_interpretacao()
{
  int res = 1;

  for (int c = 0; c < VARS; c++) {
    res = res && I[c];
  }

  return res;
}
~~~

Pela ordem fixada para as linhas da tabela, a primeira linha começa
com todas as variáveis com valor F e termina com todas as variáveis
com valor T. O loop na função `ultima_interpretacao()` apenas verifica
se todos os valores são T.

Para alterar os valores das variáveis para a próxima linha, é chamada
a função `proxima_interpretacao()`

~~~c
// altera a interpretacao atual no array I[] para a proxima na
// ordem da tabela-verdade
void proxima_interpretacao()
{
  int c = VARS - 1;

  while (c >= 0 && I[c] != 0) {
    I[c--] = 0;
  }

  if (c >= 0)
    I[c] = 1;
}
~~~

Essa é a única parte um pouco mais complicada deste programa.
O propósito da função `proxima_interpretacao()` é obter a próxima função
interpretação que será mostrada na tabela, ou seja, a próxima combinação
de valores T e F para as variáveis proposicionais da fórmula. Para isso
usamos o fato que estamos representando os valores T e F com os inteiros
1 e 0, respectivamente. A sequência de interpretações segue a ordem
binária, como já discutido antes. Portanto, o que
`proxima_interpretacao()` faz é incrementar um número binário, cuidando
do vai-um quando necessário. Por exemplo, se na interpretação atual temos
P com valor F, Q com valor T e R com valor T, isso equivale à sequência
binária 011 (que pode ser lida como o número 3 em decimal). A próxima
interpretação nesse caso é 100 (4 em decimal), que é o resultado de
incrementar em um o valor 011.

Para isso, a função varre os valores atuais (no array `I[]`) de trás
para frente, começando do último (que é interpretado como o bit menos
significativo). Essa varredura continua enquanto houver dígitos 1
(que são alterados para 0) e para ao chegar no começo do array ou
quando encontra um 0 (que é alterado para 1). Por exemplo, para
100 o loop termina logo na posição mais à direita, alterando o 0
da direita para um, o que resulta em 101; por outro lado, se a
interpretação atual corresponde a 011, o loop altera os dois dígitos 1
mais à direita para 0, e para no 0 mais à esquerda, alterando-o para um,
com resultado 100 (como esperado).

Juntando tudo, a função `main` chama `mostra_tabela()` e imprime a
tabela-verdade da fórmula escolhida:

~~~text
 P | Q | R | H
---------------
 F | F | F | T
 F | F | T | T
 F | T | F | F
 F | T | T | T
 T | F | F | F
 T | F | T | F
 T | T | F | F
 T | T | T | T
~~~

Para obter a tabela de outras fórmulas, é preciso alterar o código. O
principal é mudar a função `valor_formula()`, que
calcula o valor da fórmula para a interpretação atual. Mas também é
preciso prestar atenção à constante simbólica `VARS`, que indica o número
de variáveis proposicionais da fórmula, e a inicialização dos arrays na
função `inicializa_formula()`. Com esses cuidados, é fácil mudar o programa
para imprimir a tabela-verdade de qualquer fórmula que se queira.

Mas o ideal é ter um programa que tenha a flexibilidade de imprimir a
tabela-verdade de qualquer fórmula recebida como entrada. De preferência,
o usuário deve poder especificar a fórmula em uma sintaxe que faça sentido
para quem não conhece os detalhes internos do programa; por exemplo, uma
sintaxe similar à dos livros de lógica. Essas capacidades requerem muito
mais código e novas técnicas, como será visto. A lição importante deste
texto é que o cálculo da tabela-verdade em si não é difícil.

O [código completo do programa](https://gist.github.com/tautologico/ae06d2587eece93e184b)
está disponível como um gist.

Veja outros textos da série [Tabela-verdade em C](/series/tabela-verdade-em-c).
