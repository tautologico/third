+++
date = "2011-07-16T22:07:57-03:00"
title = "Resolvendo desafios mais complicados com Prolog"
tags = ["prolog", "puzzles"]
categories = ["programação"]
+++

No [post anterior](/post/desafiosprolog) foi mostrado
como resolver desafios de lógica no estilo do Teste de Einstein em
Prolog. Entretanto, o processo de geração no esquema de geração e
teste era bastante ineficiente para problemas mais complexos. O
problema é que muitas possibilidades são geradas e descartadas quando
é feita a verificação de diferença. Por que não gerar as
possibilidades já todas diferentes, eliminando esse teste extra?

Em Prolog o predicado select(X, L, R) seleciona o elemento X na lista
L, e retira X de L formando o resto R. Ou seja, R tem todos os
elementos de L, menos o elemento selecionado X. Isso pode ser usado
para, dada a lista de possibilidades para cada atributo (por exemplo,
cores de casa), retirar uma possibilidade da lista para gerar uma
casa, depois usar o resto da lista para gerar as cores para as outras
casas. Assim, duas casas nunca serão geradas com a mesma cor.

Como exemplo, vamos usar o próprio teste de Einstein, porque ele tem
mais possibilidades e não é possível de resolver eficientemente com o
esquema do post anterior. O problema tem cinco casas, cada uma com
cinco atributos: a cor da casa, a nacionalidade do morador, a bebida
preferida do morador, o cigarro preferido e o animal de estimação. As
dicas são:

*   O Norueguês vive na primeira casa.
*   O Inglês vive na casa Vermelha.
*   O Sueco tem Cachorros como animais de estimação.
*   O Dinamarquês bebe Chá.
*   A casa Verde fica do lado esquerdo da casa Branca.
*   O homem que vive na casa Verde bebe Café.
*   O homem que fuma Pall Mall cria Pássaros.
*   O homem que vive na casa Amarela fuma Dunhill.
*   O homem que vive na casa do meio bebe Leite.
*   O homem que fuma Blends vive ao lado do que tem Gatos.
*   O homem que cria Cavalos vive ao lado do que fuma Dunhill.
*   O homem que fuma BlueMaster bebe Cerveja.
*   O Alemão fuma Prince.
*   O Norueguês vive ao lado da casa Azul.
*   O homem que fuma Blends é vizinho do que bebe Água.

Das dicas podemos tirar as possibilidades de cada atributo. Para gerar
as cores, usamos o predicado abaixo, que é só um wrapper mais
conveniente para select, passando a lista de cores disponíveis e
recebendo o resto das cores disponíveis na última posição:

~~~prolog
gera_cor(casa(C, _, _, _, _), [C], []) :- !.
gera_cor(casa(C, _, _, _, _), Cores, Resto) :-
        select(C, Cores, Resto).
~~~

Depois é só gerar os outros atributos da mesma forma:

~~~prolog
gera_nac(casa(_, N, _, _, _), [N], []) :- !.
gera_nac(casa(_, N, _, _, _), Nacs, Resto) :- select(N, Nacs, Resto).

gera_beb(casa(_, _, B, _, _), [B], []) :- !.
gera_beb(casa(_, _, B, _, _), Bebs, Resto) :- select(B, Bebs, Resto).

gera_cig(casa(_, _, _, C, _), [C], []) :- !.
gera_cig(casa(_, _, _, C, _), Cigs, Resto) :- select(C, Cigs, Resto).

gera_ani(casa(_, _, _, _, A), [A], []) :- !.
gera_ani(casa(_, _, _, _, A), Anis, Resto) :- select(A, Anis, Resto).
~~~


Com isso, podemos gerar uma casa inteira e um conjunto de casas usando
um mapeamento recursivo:

~~~prolog
gera_casa(C, atr(Cs, Ns, Bs, Cigs, As), atr(Cs2, Ns2, Bs2, Cigs2, As2)) :-
        gera_cor(C, Cs, Cs2), gera_nac(C, Ns, Ns2),
        gera_beb(C, Bs, Bs2), gera_cig(C, Cigs, Cigs2), gera_ani(C, As, As2).

gera_casas([], _) :- !.
gera_casas([C|Cs], Atribs) :-
        gera_casa(C, Atribs, Atribs2), gera_casas(Cs, Atribs2).

gera_sol([C1, C2, C3, C4, C5]) :-
        Cores = [amarela,azul,branca,verde,vermelha],
        Nacs = [alemao,dinamarques,ingles,noruegues,sueco],
        Bebs = [agua,cafe,cerveja,cha,leite],
        Cigs = [blends,bluemaster,dunhill,pallmall,prince],
        Anis = [cachorro,cavalo,gato,passaro,peixe],
        gera_casas([C1, C2, C3, C4, C5], atr(Cores, Nacs, Bebs, Cigs, Anis)).
~~~

A estrutura atr guarda as listas com todos os atributos disponíveis,
para simplificar o código.

Para resolver o teste de Einstein é preciso estabelecer quando dois
moradores são vizinhos, como é mencionado em várias
dicas. Considerando a lista de soluções S, como gerada pelo predicado
gera_sol, acima, podemos criar predicados simples que testam (ou
geram) moradores vizinhos na solução, inclusive separando vizinhos
esquerdos de vizinhos direitos, pois uma dica especifica o lado:

~~~prolog
vizinho_esq(C1, C2, [C1,C2|_]).
vizinho_esq(C1, C2, [C3|T]) :- vizinho_esq(C1, C2, T).

vizinho_dir(C1, C2, [C2,C1|_]).
vizinho_dir(C1, C2, [C3|T]) :- vizinho_dir(C1, C2, T).

vizinho(C1, C2, S) :- vizinho_esq(C1, C2, S).
vizinho(C1, C2, S) :- vizinho_dir(C1, C2, S).
~~~

Agora é só traduzir as dicas e fazer os testes para obter a
solução. Aqui mais um truque pode ser usado: embora a ordem conceitual
seja gerar as possibilidades e depois testa-las, no caso de muitas
possibilidades (como neste desafio) o processo computacional pode se
tornar ineficiente. Então usamos as propriedades do Prolog para
resolver o desafio mais rapidamente. Em um predicado Prolog “puro”
(sem cortes ou negação) podemos mudar a ordem dos objetivos em uma
conjunção sem alterar o resultado. Todos os predicados usados até
agora são puros, então podemos colocar os testes antes, para criar
restrições sobre a solução, e chamar o predicado de geração depois
para verificar as restrições e gerar o que falta. Isso feito, o
resultado para o teste de Einstein sai bem rápido em um computador
atual. Com essas considerações, o código que testa as dicas do
problema e encontra a solução é:

~~~prolog
solucao(S):-
	C1=casa(_,noruegues,_,_,_),
	C3=casa(_,_,leite,_,_),
	S=[C1,C2,C3,C4,C5], !,
    vizinho_esq(casa(verde,_,_,_,_),casa(branca,_,_,_,_), S),
    vizinho(casa(_,noruegues,_,_,_), casa(azul,_,_,_,_), S),
    vizinho(casa(_,_,_,blends,_),casa(_,_,_,_,gato), S),
    vizinho(casa(_,_,_,_,cavalo),casa(_,_,_,dunhill,_), S),
    vizinho(casa(_,_,_,blends,_),casa(_,_,agua,_,_), S),
	member(casa(vermelha,ingles,_,_,_), S),
	member(casa(_,sueco,_,_,cachorro), S),
	member(casa(_,dinamarques,cha,_,_), S),
	member(casa(verde,_,cafe,_,_), S),
	member(casa(_, _, _, pallmall, passaro), S),
	member(casa(amarela,_,_,dunhill,_), S),
	member(casa(_,_,cerveja,bluemaster,_), S),
	member(casa(_,alemao,_,prince,_), S),
    gera_sol(S).
~~~

E o resultado sai realmente rápido:
