+++
author = "Andrei Formiga"
date = "2016-04-19T17:31:49-03:00"
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
O uso de parênteses é um exemplo das adaptações que são necessárias
para representar a estrutura hierárquica das expressões em uma
sequência linear de símbolos.

As árvores sintáticas capturam a estrutura hierárquica das expressões
pefeitamente. A árvore sintática para uma expressão é simples: os nós
internos representam operadores, e os filhos desses nós internos
são os operandos. Por exemplo, a árvore para a expressão 2 + 3 é
composta por um nó interno (a raiz), representando o operador de soma:

<center>
![Árvore para 2+3](/img/arv1.png)
</center>

Os operandos da soma são os números 2 e 3, que são filhos do nó raiz.
Se um dos operandos de um operador for uma expressão e não apenas um
número, não há problema: um dos filhos do operador na árvore será um
nó interno que também terá filhos. Por exemplo, a árvore para (2 + 3) * 4
é:
<center>
![Árvore para (2+3)*4](/img/arv2.png)
</center>

Na árvore os parênteses não são necessários porque a estrutura
da árvore representa diretamente a estrutura da expressão. A árvore
acima representa uma expressão em que a soma deve ser efetuada antes
da multiplicação, porque um dos operandos da multiplicação é a soma;
portanto, para saber que valor multiplicar por 4, é preciso antes
fazer a soma de 2 e 3. Se a expressão fosse 2 + 3 * 4, com a precedência
usual para os operadores, a multiplicação seria feita primeiro, e a árvore
seria:
<center>
![Árvore para 2+3*4](/img/arv3.png)
</center>

Expressões (fórmulas) da lógica proposicional seguem a mesma ideia, com
os conectivos funcionando como operadores. A árvore para a fórmula usada
como exemplo no post anterior é
<center>
![Árvore para fórmula](/img/arv4.png)
</center>

A questão agora é como representar essas árvores em código C.

### Implementação do tipo `Formula`

Nosso objetivo é criar um tipo `Formula` que possa representar qualquer
fórmula da lógica proposicional. Como vimos na seção anterior, esse tipo
vai poder representar as árvores sintáticas das fórmulas. Para isso
vamos criar uma estrutura encadeada onde cada nó da árvore possui ponteiros
para dois possíveis filhos. Os nós da árvore são representados pelo tipo
`Formula`, sendo que a fórmula inteira é simplesmente o nó que representa
a raiz da árvore. O tipo `Formula` contém um campo que determina o tipo do
nó, que pode ser um dos operadores ou uma das variáveis proposicionais; o
tipo do nó é representado em C pelo tipo `tipo`, uma enumeração:

~~~c
typedef enum tagTipo {
  NEG, AND, OR, IMP, BIMP, P, Q, R
} Tipo;
~~~

Além do tipo do nó, cada nó possui também ponteiros para até dois filhos,
de forma que a definição do tipo `Formula` é:

~~~c
typedef struct tagForm
{
  Tipo tipo;
  struct tagForm *dir;
  struct tagForm *esq;
} Formula;
~~~

Usamos a tag da `struct` para referenciar a própria `struct` com os ponteiros,
isso é necessário porque o `typedef` ainda não está definido; a técnica é
padrão para `struct`s recursivas em C.

Definimos uma função para ajudar a criar fórmulas:

~~~c
Formula* cria_formula(Tipo tipo, Formula *dir, Formula *esq)
{
  Formula *res = (Formula*) malloc(sizeof(Formula));

  if (res == NULL)
    return NULL;

  res->tipo = tipo;
  res->dir = dir;
  res->esq = esq;

  return res;
}
~~~

A função `cria_formula` aloca espaço para um novo nó da árvore,
e inicializa os campos do nó corretamente de acordo com as informações
passadas. Com essa função criamos funções mais simplificadas para
criar variáveis e fórmulas com conectivos. Por exemplo, para criar
uma fórmula apenas com a variável P:

~~~c
Formula *var_p()
{
  return cria_formula(P, NULL, NULL);
}
~~~

No caso de uma variável, o tipo determina qual é a variável representada
e o nó não tem filhos, usando `NULL` para os dois ponteiros do nó. Para um
conectivo unário como a negação, criamos um nó com apenas um filho (o outro
`NULL`):

~~~c
Formula* neg(Formula *e)
{
  return cria_formula(NEG, e, NULL);
}
~~~

Já para conectivos binários como a conjunção, é preciso guardar os ponteiros
para os dois filhos:

~~~c
Formula* and(Formula *d, Formula *e)
{
  return cria_formula(AND, d, e);
}
~~~

Outras funções estão disponíveis no código completo. Com essas funções
auxiliares podemos criar a fórmula
<div>
$$P \wedge Q$$
</div>
com o seguinte código:

~~~c
and(var_p(), var_q())
~~~

Que constrói uma árvore na memória com a seguinte estrutura:

![Diagrama para P AND Q](/img/diag_pandq.png)

Agora já temos a infra-estrutura necessária para descrever _quase_ qualquer
fórmula da lógica proposicional; quase porque a forma de representar as
variáveis ainda é limitada, falaremos mais sobre isso adiante. A nossa
fórmula de exemplo,

<div>
$$(P \rightarrow Q) \wedge (\neg Q \vee R)$$
</div>

é construída no programa com o seguinte código:

~~~c
Formula *f =
  and(imp(var_p(), var_q()),
      or(neg(var_q()), var_r()));

~~~

Com uma representação para as fórmulas, podemos criar uma função
`valor_formula` mais geral, que calcula o valor para qualquer fórmula
que podemos representar.

### Calculando o valor

A versão anterior da função `valor_formula` dependia da interpretação
desejada para as variáveis proposicionais (usada como uma variável global),
e a fórmula para calcular era fixa, representada diretamente no código.

Agora que temos uma representação para fórmulas, vamos fazer uma versão
do cálculo do valor que recebe a fórmula desejada como parâmetro:

~~~c
int valor_formula(Formula *f)
{
  switch(f->tipo) {

~~~

A forma de calcular o valor da fórmula depende do seu tipo, então
começamos com um `switch` nesse campo.

Para variáveis proposicionais, o valor é imediatamente obtido da
interpretação, como antes:

~~~c
  case P:
  case Q:
  case R:
      return I[indice_variavel(f->tipo)];

~~~

O único detalhe é que precisamos obter o índice da variável na
interpretação. No código, foi criado um mapeamento que coloca
a variável P com índice 0, Q com índice 1, etc. Seria melhor ter
um tratamento mais geral para as variáveis, mas (novamente) isso
é algo que vai ser comentado depois.

Se a fórmula for uma negação de outra fórmula (guardada como operando
direito da fórmula `f`), precisamos obter o valor dessa outra fórmula
e então negar o resultado:

~~~c
  case NEG:
      return !valor_formula(f->dir);

~~~

Funções de obter o valor de expressões são naturalmente recursivas, já
que para saber o valor da expressão completa precisamos obter o valor das
sub-expressões que aparecem. Os outros conectivos são similares, para
um conectivo binário como o `AND` precisamos obter o valor dos dois operandos
e então fazer o `AND` dos resultados:

~~~c
  case AND:
      return valor_formula(f->dir) &&
             valor_formula(f->esq);

~~~

A função completa é

~~~c
int valor_formula(Formula *f)
{
  switch(f->tipo) {
  case P:
  case Q:
  case R:
      return I[indice_variavel(f->tipo)];

  case NEG:
      return !valor_formula(f->dir);

  case AND:
      return valor_formula(f->dir) &&
             valor_formula(f->esq);

  case OR:
      return valor_formula(f->dir) ||
             valor_formula(f->esq);

  case IMP:
      return IMPVAL(valor_formula(f->dir),
                    valor_formula(f->esq));

  case BIMP:
      return BIMPVAL(valor_formula(f->dir),
                     valor_formula(f->esq));
  }
}
~~~

As macros `IMPVAL` e `BIMPVAL` usadas nessa função calculam o valor
de uma implicação e bi-implicação, e são fáceis de definir:

~~~c
#define IMPVAL(b1, b2)  (b1 && !b2 ? FALSE : TRUE)
#define BIMPVAL(b1, b2) (b1 == b2)

~~~

O resto da maquinaria para construir a tabela-verdade continua igual
ao programa anterior, apenas vamos mudar a função `mostra_tabela`
para receber a fórmula desejada como parâmetro:

~~~c
void mostra_tabela(Formula *f)

~~~

A fórmula `f` é usada em `mostra_tabela` quando é necessário
chamar `valor_formula`, que recebe `f` como parâmetro. A função
`main` constrói a fórmula logo antes de passá-la como parâmetro
para `mostra_tabela`:

~~~c
int main(int argc, char **argv)
{
  // (P -> Q) /\ (~Q \/ R)
  Formula *f =
    and(imp(var_p(), var_q()),
        or(neg(var_q()), var_r()));

  printf("Calculo de tabela-verdade\n\n");

  mostra_tabela(f);

  destroi_formula(f);

  return 0;
}

~~~

Como a fórmula é alocada dinamicamente com `malloc`, é preciso
liberar essa memória, o que é feito pela função `destroi_formula`.
Detalhes no programa completo.

De resto, o funcionamento do programa é o mesmo
[da versão anterior](/post/tabverdc): imprime a tabela-verdade para
uma fórmula que está no código.

### O que ganhamos?

Nesse ponto parece que após algumas voltas, paramos no mesmo lugar: a
fórmula continua fixa e determinada no código; para mudar a fórmula,
é preciso alterar o código, recompilar, e só então ver o resultado
durante a execução do programa.

Mas ter uma representação interna para qualquer fórmula cria a
possibilidade do usuário poder entrar a fórmula desejada e o
programa calcular a tabela-verdade para a fórmula entrada;
as funções `mostra_tabela` e `valor_formula` são gerais e
funcionam para qualquer fórmula que seja representada pelo
tipo `Formula`. A peça que falta é receber a entrada do usuário
e, a partir dela, construir a representação da fórmula entrada
usando o tipo `Formula`. Essa é a tarefa da análise sintática,
que será tratada no próximo texto desta série.

Um problema da representação usada são as variáveis proposicinais:
o código fixa três variáveis (P, Q e R) que as fórmulas podem usar.
Para outras variáveis, é possível mudar o código para incluir outras,
com cuidado de mudar as partes do programa que tratam das variáveis.
Mas isso depende de mudar o código e cria um problema de usabilidade:
o programa deve dizer ao usuário que ele só pode usar tais variáveis
nas suas fórmulas, o que é limitante. Isso pode ser resolvido
com técnicas de _tabelas de símbolos_, que serão assunto para outros
textos futuros da série.

O código completo está disponível.

Veja outros textos da série [Tabela-verdade em C](/series/tabela-verdade-em-c).
