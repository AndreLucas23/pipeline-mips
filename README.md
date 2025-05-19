# pipeline-mips
Processador MIPS desenvolvido no Logisim com uso de pipelines e contenção de hazards

# Unidade de fowarding

<p>
A unidade de fowarding é responsável por disponibilizar os dados das instruções nos estágios MEM e WB para instruções que estão no estágio EX (ou MEM no caso da instrução sw), prevenindo que as operações sejam realizadas com valores desatualizados. O circuito consiste em 7 sinais de entrada: ex/mem.WR, mem/wb.WR, ex/mem.RD, mem/wb.RD, id/ex.RS, id/ex.RT, e 3 sinais de saída FA, FB e FSW. As saídas são aplicadas em multiplexadores, dependendo do valor de saída, o sinal escolhe entre o que vem do banco de registradores (00), o dado de ex/mem (10), ou o dado de mem/wb (01), já o FSW escolhe o dado atual (0) ou o que se encontra em mem/wb (1).   
</p>
<p>
A diferença entre FA e FB consiste no registrador ao qual se escolhe, o FA faz o fowarding para o registrador RS enquanto o FB para o RT. A lógica é a mesma:
bit 1: se o sinal ex/mem.WR está aceso, o registrador de destino ex/mem.RD é diferente de 0 e ex/mem.RD é igual ao registrdor id/ex.RS (id.ex/RT), então ocorre fowarding de MEM para EX.
bit 0: se o sinal mem/wb.WR está aceso, o registrador de destino mem/wb.RD é diferente de 0 e mem/wb.RD é igual ao registrdor id/ex.RS (id.ex/RT) e o sinal ex/mem.WR não está aceso ou ex/memRD = 0 ou ex/mem.RD = id/ex.RS(id/ex.RT), então ocorre fowarding do estágio WB para EX. 
A última sentença do bit 0 explica-se paras o caso de modificações em sequência como:
  <br>
add $1, $1, $2<br>
add $1, $1, $3<br>
add $1, $1, $4<br>
para que o fowarding retorne o valor mais atualizado.
</p>
<p>
O fowarding para instrução SW é simples: se o registrador ex/mem.RT = mem/wb.RD e mem/wb.WR está aceso, então ocorre fowarding de WB para MEM. Não é preciso verificar o sinal de escrita na memória pois se ele estiver desabilitado, não há diferença se a condição é verdadeira, pois nada será escrito.   
</p>
<p>
A unidade de hazard é responsável por gerar um stall na sequência de execução caso haja uma instrução lw seguida de outra que dependa de seu valor. A lógica: se id/ex.Memtoreg está aceso e id/ex.RT = if/id.RS ou (id/ex.RT = id/ex.RT e opcode diferente do opcode do sw) então ocorre um stall. O sinal de stall zera todos os outros da unidade de controle (nop) e impede a instrução que está em if/id e pc de passar. O stall não precisa ocorrer no caso de um lw seguido de sw pois a unidade de fowarding consegue levar esse dado a tempo do WB para MEM. Para exemplificar:
<br>
lw $1, 0($sp)<br>
sw $1, 4($sp) # não ocorre stall<br>
lw $1, 0($sp)<br>
sw $2, 4($1) # ocorre stall<br>
lw $1, 0($sp)<br>
sw $1, 4($1) # ocorre stall
</p>
