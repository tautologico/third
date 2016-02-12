+++
date = "2011-06-28T10:58:55-03:00"
title = "Resolvendo desafios de lógica com Prolog"
tags = ["prolog", "puzzles"]
categories = ["programação"]
+++

Dado que em Prolog é fácil criar estruturas (termos compostos) e fazer
buscas sujeitas a certas restrições, é fácil resolver os tais desafios
de lógica usando a linguagem. O tipo de desafio de lógica em questão é
similar aquele ["teste de Einstein"](http://rachacuca.com.br/teste-de-einstein/)
que algumas correntes de email dizem que só meia-dúzia de pessoas no mundo conseguem resolver
(o que é uma grande balela). Várias informações (mas não todas) são dadas
sobre um conjunto de indivíduos, e é preciso deduzir as informações
que faltam para chegar na solução. Resolver um problema desses em
Prolog é uma forma de explorar várias características de linguagem e
alguns padrões de uso.

A [Coquetel](http://coquetel.uol.com.br/) publica uma linha de revistas
de Desafios de Lógica desse
tipo, usando uma tabela para ajudar a cruzar as informações e resolver
o problema. Esses problemas são um pouco diferentes porque algumas
informações não estão nas dicas, mas apenas nas listas de atributos
nas tabelas, e porque às vezes é preciso raciocinar por exclusão
(assumindo que não existem duas pessoas com o mesmo atributo). Para a
solução (em Prolog) de um puzzle simples no estilo do teste de
Einstein, veja [esse post acolá](http://braindumpinstant.blogspot.com/2011/04/prolog-1.html).
Aqui vou mostrar a solução para um
problema de uma revista Desafios de Lógica (número 98). Sem colocar a
tabela, aqui estão as informações dadas (espero que a Coquetel não me
processe):

> Desafio: Primeiro de Abril
>
> Quatro pessoas que gostam de pregar peças decidiram tornar o primeiro de abril inesquecível,
> ou seja, com muitas brincadeiras. Cada um pregou uma peça numa vítima diferente usando
> um objeto inofensivo.
>
> Nomes: Ana, Ester, Pablo, Rodolfo
>
> Sobrenomes: Fontes, Levis, Matoso, Salgado
>
> Brincadeiras: Almofada de barulho, Aranha falsa, Foto alterada, Mosca falsa
>
> Vítimas: Irmão, Mãe, Pai, Tia
>
> Dicas:
>
> Ana deu risadas enquanto colocava uma aranha falsa na comida de sua vítima.
> A pessoa de sobrenome Salgado (que não é Ana) pregou uma peça em seu irmão.
> A pessoa de sobrenome Matoso colocou uma almofada de barulho na cadeira de sua vítima.
> Rodolfo pregou uma peça em sua tia, mas não foi ele que usou a almofada de barulho.
> A brincadeira feita por Levis incluía uma mosca falsa. A vítima de Ester foi seu pai.

Para garantir que todas as possibilidades são examinadas e são
testadas posteriormente, incluindo as partes de dedução por exclusão,
decidi usar o padrão "gerar e testar" (*generate-and-test*) que é comum
em Prolog: gerar todas as possibilidades e ir restringindo (testando)
até achar a solução.

### Geração

Começando com a geração: o procedimento a seguir gera todas as
configurações possíveis, se for usado backtracking. Para representar
as informações de uma pessoa, usei uma estrutura
__p(Nome, Sobrenome, Brincadeira, Vítima)__.

~~~prolog
/* p(Nome, Sobrenome, Brincadeira, Vitima) */
gera(p(N, S, B, V)) :-
    member(N, [ana, ester, pablo, rodolfo]),
    member(S, [fontes, levis, matoso, salgado]),
    member(B, [almofada, aranha, foto, mosca]),
    member(V, [irmao, mae, pai, tia]).
~~~

O procedimento gera as possibilidades corretas usando o predicado
pré-definido member, garantindo que o nome gerado será um entre
[ana, ester, pablo, rodolfo], e por aí vai. Podemos testar a geração
fazendo uma consulta como

~~~prolog
?- gera(P).
~~~

O próximo passo é gerar não só um indivíduo, mas uma solução completa,
que são quatro indivíduos. Se simplesmente usarmos

~~~prolog
?- gera(P1), gera(P2), gera(P3), gera(P4).
~~~

serão gerados muitos grupos com indivíduos repetidos. Precisamos
garantir a geração de quatro indivíduos diferentes em todos os
atributos. Não funciona se tentarmos usar P1 \= P2 e por aí vai,
porque o Prolog vai dizer que duas estruturas que diferem em apenas um
dos termos são diferentes. Por exemplo,

~~~prolog
?- p(ana, fontes, almofada, irmao) \=
     p(ana, fontes, almofada, mosca).
true.
~~~

Precisamos diferenciar todos os atributos. Além disso, é preciso ter
quatro indivíduos diferentes dois a dois, pois se P1 é diferente de P2
e P2 é diferente de P3, é possível que P1 seja igual a P3, e por aí
vai. Assim, definimos

~~~prolog
dif(p(N1, S1, B1, V1), p(N2, S2, B2, V2)) :-
    N1 \= N2, S1 \= S2, B1 \= B2, V1 \= V2.

todas_dif(P1, P2, P3, P4) :-
    dif(P1, P2), dif(P1, P3), dif(P1, P4),
    dif(P2, P3), dif(P2, P4), dif(P3, P4).
~~~

Agora, se testarmos uma consulta do tipo

~~~prolog
?- gera(P1), gera(P2), gera(P3), gera(P4),
   todas_dif(P1, P2, P3, P4).
~~~

Só veremos soluções em que as quatro pessoas são diferentes em todos
os atributos. Falta testar as possibilidades geradas com as dicas
dadas no problema para encontrar a solução.

### Testes

A solução consiste de quatro indivíduos, todos diferentes (usando o
critério de diferença discutido), que se encaixem nas dicas dadas pelo
problema. A ideia é capturada no procedimento que resolve o problema:

~~~prolog
solucao(S) :-
     S = [P1, P2, P3, P4],
     gera(P1), gera(P2), gera(P3), gera(P4),
     todas_dif(P1, P2, P3, P4),
     member(p(ana, _, aranha, _), S),
     member(p(N, salgado, _, irmao), S), N \= ana,
     member(p(_, matoso, almofada, _), S),
     member(p(rodolfo, _, B, tia), S), B \= almofada,
     member(p(_, levis, mosca, _), S),
     member(p(ester, _, _, pai), S).
~~~

O primeiro objetivo da solução apenas diz que S (a solução) é uma
lista de tamanho 4, formada por P1, P2, P3 e P4. Na segunda e terceira
linha do corpo da cláusula, temos o padrão de geração discutido
antes. Depois, cada linha é um teste que representa uma dica do
problema. O primeiro teste diz que Ana usou a Aranha como brincadeira,
mas não conhecemos seu sobrenome nem qual foi sua vítima; esse
indivíduo faz parte da solução (usando o predicado pré-definido
member). A segunda diz que a pessoa de sobrenome Salgado pregou uma
peça no irmão; não conhecemos o nome dessa pessoa, mas sabemos que não
é Ana, como expresso pela desigualdade seguinte. As outras linhas
traduzem as outras dicas, sem maiores novidades.

Carregando o código inteiro em um sistema Prolog como o
[SWI-Prolog](http://www.swi-prolog.org/), e
fazendo a consulta

~~~prolog
?- solucao(S).
~~~

A resposta deverá aparecer após algum tempo. Dependendo do computador,
isso pode demorar alguns minutos, o que é ruim. Além disso, são
retornadas soluções equivalentes, em que apenas a ordem dos indivíduos
muda. Isso tudo demonstra que o sistema Prolog está buscando em um
espaço de estados muito maior do que necessário. Podemos fazer isso de
maneira mais eficiente.

### Geração mais eficiente

Não afeta a resolução dizer que o primeiro indivíduo na solução tem
nome Ana, o segundo Ester, o terceiro Pablo e o quarto Rodolfo. Isso
restringe o espaço de busca sem eliminar a solução. Se gerarmos as
possibilidades já prendendo esses atributos, a busca pela solução se
torna muito mais rápida. Definimos um novo predicado para gerar as
possibilidades mais eficientemente:

~~~prolog
gera_ef(P1, P2, P3, P4) :-
    P1 = p(ana, _, _, _),
    P2 = p(ester, _, _, _),
    P3 = p(pablo, _, _, _),
    P4 = p(rodolfo, _, _, _),
    gera(P1), gera(P2), gera(P3), gera(P4),
    todas_dif(P1, P2, P3, P4).
~~~

Depois é só usar esse predicado na busca pela solução. Além disso, só
precisamos de uma solução, então vamos incluir um corte no final para
evitar cálculos inúteis. O resultado é

~~~prolog
solucao2(S) :-
    S = [P1, P2, P3, P4],
    gera_ef(P1, P2, P3, P4),
    member(p(ana, _, aranha, _), S),
    member(p(N, salgado, _, irmao), S), N \= ana,
    member(p(_, matoso, almofada, _), S),
    member(p(rodolfo, _, B, tia), S), B \= almofada,
    member(p(_, levis, mosca, _), S),
    member(p(ester, _, _, pai), S),
    !.
~~~

E agora uma consulta do tipo

~~~prolog
?- solucao2(S).
~~~

Vai achar a solução muito mais rápido. O código completo pode ser
visto [aqui](https://gist.github.com/1051306).
