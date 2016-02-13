+++
date = "2011-05-16T21:00:17-03:00"
title = "P ?= NP como um problema de decisão"
tags = ["teoria da computação"]
+++

_O post abaixo foi publicado inicialmente em uma lista de discussão,
em resposta ao email de outro participante, que estava em dúvida se a
resposta do gabarito de uma questão do POSCOMP 2010 estava correta. A
questão está na imagem abaixo, com a resposta do gabarito indicada em
vermelho._

![Questão 50 POSCOMP 2010](/img/POSCOMP_2010_Q50.jpeg "POSCOMP_2010_Q50")

A questão aí é mais sutil do que parece à primeira vista. O enunciado
não fala diretamente sobre como decidir se P != NP é verdade ou não,
mas pede para considerar a hipótese P != NP como um problema de
decisão a ser resolvido por um algoritmo, ou seja, por algum
autômato. Um
[problema de decisão](http://pt.wikipedia.org/wiki/Problema_de_decis%C3%A3o)
é uma questão para a qual um algoritmo deve responder apenas uma das
duas possibilidades: ou SIM ou NÃO.

Normalmente, problemas de decisão são esquemas gerais para uma coleção
de problemas particulares que são instâncias do caso geral; por
exemplo, veja o problema da parada: decidir se uma máquina de Turing
(ou seja, um programa) específica sempre termina sua computação com
uma resposta definitiva, ou não. Esse é um problema geral. Uma
instância desse problema seria pegar uma máquina de Turing M1 (que
basicamente contém um programa específico) e perguntar: essa máquina
M1 sempre pára e chega a uma resposta? Veja que para alguns casos a
decisão é bem simples; imagine por exemplo uma máquina de Turing que
desconsidera a entrada e apenas imprime uma sequência qualquer de
símbolos na fita (digamos, a sequência “Hello, world!”). Essa máquina
de Turing obviamente sempre pára e dá uma resposta. Seria fácil fazer
uma algoritmo que, sempre que vê uma máquina de Turing que apenas
imprime na fita (ou, digamos, um programa C que só contém printfs),
responde SIM ao problema da parada, ou seja, essa máquina (ou
programa) sempre vai parar.

A dificuldade do problema da parada não está em instâncias
específicas, mas sim no problema geral: ter um algoritmo que consiga
decidir isso para qualquer instância, para qualquer máquina de Turing,
para qualquer programa de computador.

Mudando a discussão de indecibilidade para P e NP: se pensarmos no
problema SAT, o problema de satisfatibilidade de fórmulas
proposicionais, é fácil fazer um programa que responde a algumas
dessas instâncias em tempo polinomial (por exemplo, fórmulas que só
tem uma proposição, sem conectivos), mesmo que o problema geral seja
NP (porque algumas instâncias nunca poderão ser resolvidas em tempo
polinomial, se P != NP).

Agora vamos pensar em P != NP como um problema de decisão para ser
resolvido por um algoritmo. O importante é que esse problema não tem
instâncias. Ou P != NP é verdade, ou não é. É um problema único, e não
um conjunto de instâncias de problemas, como os exemplos anteriores.

Com isso, vamos observar a afirmação IV: imagine que um algoritmo é
“responda SIM” (ou seja, apenas imprimir a resposta “SIM”) e o outro é
“responda NÃO”. É claro que, para o problema P != NP, um dos dois vai
acertar. Não sabemos ainda qual dos dois é o certo, mas ou P != NP, ou
não é verdade que P != NP (e portanto P = NP). Então acho que é fácil
ver que a afirmativa IV está correta. Agora, quanto à III: desses 2
algoritmos da afirmativa IV, um dos dois está correto (embora não
saibamos qual). Mas eles são de tempo polinomial? Claro que sim, eles
apenas respondem imediatamente SIM ou NÃO, sem nenhum
processamento. Ou seja, existe um algoritmo que responde ao problema
de decisão P != NP em tempo polinomial (na verdade tempo constante,
mas todo algoritmo O(1) também é O(n) e por aí vai), e, embora nós não
saibamos que algoritmo é esse, sabemos que é um dos dois acima. Então
a alternativa III está correta também. I e II obviamente não podem
estar corretos se IV está, e na verdade não estão, exatamente porque
os algoritmos mencionados em IV são suficientes.
