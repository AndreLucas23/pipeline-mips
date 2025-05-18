<h1 style='text-align: center'>Processador MIPS com pipelines e contenção de hazards</h1>

André Lucas de Souza Lima <br>
Kayky Moreira Morais <br>
Ciência da Computação - Universidade Federal do Cariri <br>
Desenvolvido na disciplina de Arquitetura e Organização de Computadores <br>
Instruído pelo docente Ramon Santos Nepomuceno <br><br>

<h2>Do que se trata</h2>

Este projeto se trata de um processador do tipo MIPS implementado no software Logisim, em que há o uso da técnica de pipelining para acelerar a execução de instruções e contenções forwarding e stalling para impedir hazards de dados.

Além disso, serão especificadas exemplos de instruções que precisaram ser modificadas para funcionarem corretamente no ambiente de pipelining.

<h2>Como instalar o Logisim</h2>

A forma mais rápida e segura de instalar o Logisim é pelo site SourceForge, conhecido por disponibilizar downloads em uma plataforma sólida.

Ao acessar este <a href="https://sourceforge.net/projects/circuit/">link</a>, você deve ser redirecionado para a página do Logisim no SourceForge.

A partir disso, basta apenas apertar o botão principal escrito "Download" igual ao a seguir, que transferirá um executável para seu navegador e solicitará um local de armazenamento no computador.

![página do Logisim no sourceforge](./Imgs/botao-sourceforge.png)

Após isso, o Logisim já estará instalado e pronto para ser executado por meio desse arquivo.

<h2>Instruções modificadas</h2>

Primeiramente, serão abordadas certas instruções que precisaram ser alteradas para que pudessem se encaixar no novo processador sem que houvessem erros. São elas:

<h3>Instruções lógicas (AND, OR)</h3>

Para adequar as instruções AND e OR do tipo R, foram apenas inseridas duas portas lógicas AND e OR na ULA, de modo que um código com opcode 000000 (tipo R) pode vir usá-las se seu funct for 000100 (AND) ou 000101 (OR).

![portas and e or](./Imgs/and-or.png)

<h3>Instruções de desvio (BNE, JR, JAL, J)</h3>

A instrução BNE foi implementada do mesmo modo da BEQ, em que um sinal BNE é ativado da UC a partir do momento em que se identifica o opcode 001001. E então, espera-se um sinal "igual" negado vindo da ULA para garantir que o salto pode ser feito no caso dos valores vindos de RS e RT forem diferentes.

![verificacao de valores diferentes](./Imgs/porta-bne.png)

Caso sejam, o endereço calculado da branch a partir do imediato é passado para o PC, realizando assim o salto.

Abaixo, mostra-se o recebimento do imediato, que é "shiftado" duas vezes à esquerda e somado ao PC+4 para resultar no endereço alvo.

![calculo do endereço do bne](./Imgs/calculo-bne.png)

Tal endereço é passado pelo pipeline EX/MEM em direção ao estado MEM, onde é decidido, a partir da verificação anterior, se o endereço será ou não pulado.

![circuito do bne inteiro](./Imgs/circuito-bne.png)

Em relação à instrução JR, um sinal é ativo para o caso do opcode ser igual a 001011. Para a decisão do endereço alvo, há um desvio do sinal JR no estado MEM onde, junto com um OR entre ele, um sinal de JUMP e de JR, é decidido que haverá um pulo na memória de instruções.

![or para decidir sobre jump](./Imgs/or-jump.png)

O endereço sempre virá do registrador RS, que já será um número de 32 bits, então só precisará ser passado pelo pipeline ID/EX, estado EX, pipeline EX/MEM, onde finalmente irá do estado MEM até um multiplexador ligado ao PC, que decidirá entre o PC+4, endereço de JUMP, endereço de BRANCH (BEQ ou BNE) e endereço de JR.

Já para JAL, o opcode 001100 é responsável por identificar a instrução e ativar o sinal JAL vindo da UC. Assim que é ativado, ele é passado diretamente para o pipeline MEM/WB e retorna para as entradas do banco de registradores pois, caso esteja ligado, registrará o valor de PC+4 no registrador 31, como mostrado na figura a seguir:

![entradas de jal no banco de registradores](./Imgs/entradas-jal.png)

O JUMP, diferentemente, com um opcode 000110, acenderá um sinal da UC que irá chegar no estado MEM e ser inserido no mesmo OR que as instruções JR e JAL.

O cálculo do endereço, no caso do JAL e JUMP, é dado pelo "shift" de duas casas à esquerda e concatenação dos quatro bits mais significativos do PC+4 aos quatro bits mais significativos do endereço a ser calculado.

<h3>Instruções de Comparação (SLT, SLTI)</h3>

Como SLT é uma instrução de tipo R, foi apenas necessária a adição de um sinal de "menor que" na ULA, que compara o registrador RS com RT e funciona com o funct 000110 e opcode 000000.

![sinal de menor que](./Imgs/menor-que.png)

Caso tal sinal seja verdadeiro, ele é passado estendido para 32 bits ao registrador RD.

Para o caso do SLTI, foi adicionado um opcode próprio de número 001010 que registra uma operação na ULA, a qual utiliza o mesmo artifício citado durante a explicação do SLT, mas comparando o conteúdo do registrador RS com um valor imediato e, caso seja menor, transferindo um número "1" de 32 bits ao registrador RT.

<h3>Instruções de Shift (SLL, SLR)</h3>

As instruções SLL e SLR possuem a mesma função, com a única diferença que SLL desloca bits para a esquerda, enquanto SLR, para a direita.

A implmentação de ambos consiste na definição dos opcodes 001101 e 001110, respectivamente, que simplesmente ativam os sinais da UC básicos para qualquer instrução que deseja utilizar operações da ULA sem que seja de tipo R.

É o que acontece com SLL e SLR, que acendem os ALUOPs 10111 e 11000, respectivamente, os quais ativam deslocadores de bits que permitem que essas execuções os utilizem, definindo a quantidade de bits de acordo com o número encontrado no shamt.

É de extrema importância ressaltar que todas as instruções descritas acima que escrevam dados em algum registrador precisam ativar o sinal RegWrite da UC, que será sempre levado de pipeline a pipeline até o estado WB, onde haverá a escrita em si.

Lá, tal sinal dará a permissão no banco de registradores para que o registrador definido por RegDest, também vindo da UC, tenha o seu valor alterado.

<h2>O que é o pipelining?</h2>

Em um processador MIPS tradicional, as funções seguem o fluxo intuitivo de execução: a primeira se inicia, é encerrada, e então a próxima é processada, seguindo esse funcionamento até o fim.

Graficamente, parece-se com a imagem abaixo:

![representação de instruções sem pipeline](./Imgs/instrucoes-sem-pipeline.png)

Com o tempo, desenvolvedores de hardware perceberam que o tempo de execução poderia ser acelerado.

Para que isso acontecesse, uma solução foi inventada: seriam implementadas estruturas chamadas "pipelines" que, em essência, são registradores distribuidos pelo circuito que "prendem" as informações de cada instrução e, após o próximo ciclo de clock, liberam elas para o próximo pipeline,

Dessa maneira, uma instrução não precisa esperar pelo final da anterior, e sim pelo ciclo posterior, que acontece antes do final da primeira.

Com essas mudanças, o diagrama de tempo de execução fica mais próximo com a representação a seguir:

![representação de instruções com pipeline](./Imgs/instrucoes-com-pipeline.png)

Como visto, a espera total para o fim do programa é reduzida consideravelmente.

Abaixo, a imagem de um pipeline feito no Logisim:

![pipeline em logisim](./Imgs/pipeline.png)

Contudo, essa implementação gera alguns problemas a serem resolvido. Aqui, iremos tratar de um desses: os hazards de dados.

<h2>O que são os hazards de dados?</h2>

Em certas sequências de códigos a serem executados, pode ser que haja a ocasião em que um dado precise ser utilizado por uma instrução sem que ele tenha sido escrito pela anterior. 

Para exemplificar, o código abaixo possui um hazard do tipo citado.

![hazard de primeiro tipo](./Imgs/hazard-tipo1.png)

Durante a primeira linha, escreve-se o valor 5 no registrador $t1 e a segunda instrução necessita dele para inserir o mesmo valor no registrador $t2.

Porém, como a segunda instrução é executada sem que a primeira seja terminada, o valor contido no registrador não está atualizado para 5, o que revela um hazard.

Além do tipo acima, há uma segunda ocorrência que se desenrola quando uma instrução tenta buscar um dado que ainda não está pronto por estar sendo preparado por outra.

O hazard mencionado pode ser visto na próxima figura:

![hazard de segundo tipo](./Imgs/hazard-tipo2.png)

Aqui, a primeira linha guarda, no registrador $t0, o valor armazenado no endereço contido em $s0 e a instrução seguinte precisa dele para salvar um número em $t1.

Todavia, a informação ainda não está pronta pois a segunda linha utiliza o valor em um estado anterior ao da primeira.

Ambas formas de hazard devem ser corrigidas, respectivamente, por forwarding e stalling, técnicas específicas que serão destrinchadas a seguir.
