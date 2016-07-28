+++
author = "Andrei Formiga"
date = "2013-07-10T20:29:17-03:00"
description = "Como calcular os coeficientes na regressão linear."
draft = false
keywords = ["regressão linear"]
tags = ["matemática", "aprendizado de máquina"]
title = "Regressão Linear Simples"
topics = ["Aprendizado de máquina"]
type = "post"

+++

A maioria dos exemplos conhecidos de aplicações de aprendizado de
máquina são sistemas que realizam a tarefa de
classificação. Classificação consiste em determinar a que classe um
objeto pertence, dados os valores de um conjunto de atributos do
objeto. Normalmente cada objeto pertence a apenas uma classe entre um
conjunto fixo de possibilidades, o caso mais comum sendo o da
classificação binária, na qual o objeto pode pertencer a uma de duas
classes possíveis. Por exemplo, filtragem de spam: dado o conteúdo de
uma mensagem de email (conjunto de palavras) decidir se esta mensagem
é ou não é spam. Outros exemplos são detectar se uma operação com
cartão de crédito é fraudulenta ou não, e detectar se um conjunto de
pixels é um rosto ou não. Matematicamente, classificar é aplicar uma
função que dados os valores dos atributos como entrada produz como
resposta uma saída discreta. Treinar um classificador é faze-lo
aprender a função de classificação, a partir de um conjunto de dados
cuja classe de cada objeto é conhecida.

Uma outra tarefa importante de aprendizado é a regressão. A diferença
entre classificação e regressão é que nesta última a saída do sistema
(da função aprendida) é contínua. Por exemplo, prever o preço que um
objeto terá quando vendido em um mercado, ou prever a pontuação média
no ENEM dos alunos de uma escola, baseado na pontuação dos mesmos
alunos em um simulado aplicado pela escola.

Na regressão linear a função que deve ser aprendida é uma relação
linear entre entrada e saída. Modelos lineares são simples e mais
tratáveis matematicamente, mas bastante úteis. Modelos lineares também
servem como base para entender mecanismos mais complicados como redes
neurais. A regressão linear é simples quando o modelo estabelece que a
saída depende apenas de um atributo de entrada. É uma função linear de
uma única variável, ou seja, f(x) = kx + c, onde k e c são
coeficientes constantes. A questão é como determinar k e c. A
abordagem que usaremos aqui é conhecida como método dos mínimos
quadrados, e neste método a função f(x) é aprendida a partir de um
conjunto de dados cuja saída é conhecida; no contexto do aprendizado
de máquina esse conjunto de dados normalmente é chamado de conjunto de
treinamento. A ideia é calcular os coeficientes de maneira a minimizar
uma medida de erro que é a diferença quadrática entre a saída real
(que conhecemos para o conjunto de treinamento) e a saída prevista
pelo modelo. Aqui vamos mostrar como calcular os coeficientes para a
regressão simples.

### Encontrando os Coeficientes

Seja a função de regressão f(x) = kx + c que modela a relação entre
uma variável independente (x) e uma variável dependente (y), e seja um
conjunto de n pontos de dados conhecidos:
<div>
$$D = \{ (x_1, y_1), (x_2, y_2), \dots, (x_n, y_n)\}$$
</div>

Queremos encontrar os coeficientes k e c para o modelo que minimizem a
medida de erro quadrático (RSS ou _Residual Sum of Squares_):
<div>
$$RSS = \sum_{i=1}^n (y_i - f(x_i))^2$$
</div>

Para isso, vamos calcular as derivadas parciais do erro (RSS) em
relação a k e c e igualar a zero para obter o ponto de mínimo. Podemos
desenvolver a expressão para o RSS da seguinte forma:
<center>
![RSS](/img/rss.svg)
</center>

Assim, podemos calcular as derivadas parciais:
<center>
![Derivada de RSS em c](/img/drssdc1.svg)
</center>

Para simplificar as expressões, vamos calcular as somas das colunas e
chamar tais somas de X e Y:
<div>
$$X = \sum\_{i=1}^n x_i, \quad Y = \sum\_{i=1}^n y$$
</div>

Note que X e Y podem ser calculados usando apenas os pontos do
conjunto de dados D. Igualando a derivada parcial em relação a c a
zero e isolando c, temos
<center>
![Derivada de RSS em c](/img/drssdc2.svg)
</center>

Com isso é possível calcular o valor de c se soubermos o valor de
k. Para obter o valor de k calculamos a derivada parcial do RSS em
relação a k e igualamos a expressão da derivada a zero, a partir da
Eq.(1):
<center>
![Derivada de RSS em k](/img/drssdk.svg)
</center>

Agora igualando a zero e substituindo a expressão para c encontrada na
Eq.(2), obtemos:
<center>
![Derivada de RSS em k](/img/drssdk2.svg)
</center>

Note que a expressão para o valor de k na Eq.(3) agora depende apenas
dos valores dos pontos de dados em D. Assim, é possível calcular o
valor de k pela Equação (3), e com esse valor é possível calcular o
valor de c pela Equação (2).

Com essas fórmulas é bastante fácil implementar um programa simples
que calcula k e c, e a partir deles prever o resultado para novos
pontos de dados. O vídeo a seguir mostra como fazer esses cálculos
para um conjunto de dados de entrada, usando a linguagem de
programação [Julia](http://julialang.org) e o sistema de visualização
[gnuplot](http://gnuplot.info/).

{{< youtube 3x3vMhPVygQ >}}
