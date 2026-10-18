# Persistência Linux com GSocket: um estudo de Red Team

## Resumo

Este artigo explica como o GSocket pode fazer parte de uma situação de persistência em Linux. O texto usa como base o material do Red-Book da Infiltr8. A ideia principal é que duas coisas diferentes trabalhem juntas: um recurso do próprio sistema que pode iniciar um programa de novo depois de um login ou da inicialização do computador, e o GSocket pode ajudar esse programa a se comunicar com outro computador pela rede GSRN. Isso pode facilitar a comunicação quando os computadores não têm um endereço fixo ou quando conexões diretas são difíceis. Mesmo assim, ainda é preciso ter acesso ao computador, iniciar o programa e conseguir se conectar à rede. O artigo também compara scripts, serviços systemd e acesso por SSH. Por fim, apresenta uma forma segura de um time Red Team testar se uma organização consegue perceber esse tipo de atividade.

**Palavras-chave:** Red Team; Linux; persistência; GSocket; GSRN; systemd; SSH; MITRE ATT&CK.

## 1. Introdução

Um time de Red Team faz testes autorizados para entender como uma organização se sairia diante de algumas ações que um invasor poderia tentar. Nesse tipo de teste, não basta descobrir se um programa consegue iniciar. Também é importante saber se a equipe de segurança percebe as mudanças, entende o que aconteceu e responde a tempo.

O site da Infiltr8 apresenta o GSocket como uma ferramenta de rede que pode conectar computadores por meio da Global Socket Relay Network, ou GSRN. Um relay é um intermediário que ajuda os computadores a se encontrarem. Segundo a fonte, os computadores usam um segredo compartilhado, e a comunicação é criptografada. Assim, eles não precisam depender de um endereço IP público fixo para se localizar.

O GSocket, sozinho, não faz um programa iniciar novamente depois que o computador é reiniciado. Essa parte depende de recursos do Linux, como scripts de login, tarefas agendadas ou serviços do systemd. Portanto, o risco aparece quando um mecanismo de inicialização automática é ligado a um programa que se comunica pela rede.

## 2. Como o GSocket funciona neste cenário

### 2.1 Comunicação pela GSRN

Na situação descrita pelo site, os dois computadores se conectam à GSRN. O relay encaminha a comunicação entre eles, e um segredo compartilhado ajuda os computadores a se reconhecerem. Como o tráfego é criptografado, quem observa a rede pode ter dificuldade para ver o conteúdo da comunicação.

Isso não quer dizer que a ferramenta consiga passar por qualquer bloqueio. O computador ainda precisa conseguir se comunicar com a rede necessária. Além disso, mesmo que o conteúdo esteja criptografado, o sistema pode registrar informações importantes: qual programa iniciou, qual usuário estava usando o computador, de onde o programa foi executado e se algum arquivo de inicialização foi alterado.

### 2.2 Hipótese do teste

O cenário começa supondo que alguém já conseguiu executar um programa no computador. O GSocket não é, por si só, uma forma de obter o primeiro acesso. A pergunta do teste é: **a equipe de segurança percebe quando um programa é preparado para iniciar novamente e tenta se comunicar com outro computador?**

Para responder, o time pode verificar:

- O sistema registra quando um arquivo de inicialização é alterado?
- É possível identificar qual usuário e qual programa fizeram a mudança?
- Os registros de rede mostram qual programa iniciou uma conexão?
- A equipe percebe se um programa volta a iniciar depois de uma reinicialização?
- Existe um plano para investigar e remover mudanças suspeitas sem perder informações importantes?

O objetivo é avaliar os controles de segurança em computadores autorizados, não manter acesso escondido ou entrar em sistemas fora do escopo do teste.

## 3. Etapas da situação estudada

O cenário pode ser entendido em cinco etapas. Elas ajudam a organizar o teste e a análise, mas não são instruções para instalar uma ferramenta.

**1. Acesso ao computador.** Para que a situação aconteça, o programa precisa já ter sido colocado ou executado no computador. Em um teste, a equipe deve anotar de onde veio o arquivo, qual usuário estava envolvido e quais programas estavam ativos naquele momento.

**2. Escolha de quando o programa inicia.** Um recurso do Linux pode iniciar um programa quando alguém entra no sistema, em um horário programado ou quando o computador liga. O recurso escolhido define quando o programa roda e com quais permissões.

**3. Tentativa de comunicação.** Depois de iniciar, o programa pode tentar se comunicar usando a GSRN. A equipe de segurança deve observar tanto a conexão quanto o programa que a iniciou.

**4. Repetição da atividade.** Se o programa parar, o recurso de inicialização pode iniciá-lo outra vez, dependendo de como foi configurado. Comparar o horário da mudança com os horários de execução e de conexão ajuda a entender a sequência.

**5. Limpeza do teste.** Quando o exercício terminar, a equipe deve retirar os arquivos e as mudanças temporárias, cancelar os segredos de teste e confirmar que nada continua iniciando. Essa verificação faz parte do trabalho.

## 4. Três formas de manter acesso

O site apresenta três opções. Scripts e serviços systemd podem iniciar um programa GSocket. SSH é diferente: ele pode manter acesso por meio de contas e chaves de acesso remoto, sem usar a GSRN.

### 4.1 Scripts e tarefas agendadas

O site cita arquivos usados quando um usuário entra no sistema, como `.profile` e `.bashrc`, além de tarefas agendadas. Esses recursos podem iniciar um programa em um momento específico, como no login ou em um horário definido.

Para o Red Team, o ponto importante é verificar se a organização percebe uma mudança nesses arquivos ou tarefas. A equipe deve saber quais entradas são normais, quem as alterou e com qual objetivo. O programa também pode rodar com as permissões do usuário relacionado, então é importante observar qual conta está envolvida.

### 4.2 Serviços do systemd

O systemd é uma parte do Linux que controla serviços, isto é, programas que podem funcionar em segundo plano. Um serviço pode iniciar quando o computador liga e pode continuar ativo mesmo sem uma pessoa estar usando uma sessão.

Em um teste, vale conferir se a equipe consegue identificar serviços novos ou alterados, descobrir qual programa eles iniciam e verificar qual usuário está associado. O nome do serviço não é suficiente para saber se ele é confiável. Também é preciso olhar o caminho do programa, quem é o responsável e se a mudança foi aprovada.

### 4.3 Acesso persistente por SSH

SSH é uma forma comum de administrar computadores à distância. O site relacionado também explica como chaves de acesso SSH podem ser usadas de forma indevida para permitir novos acessos. Essa técnica é separada do GSocket: não usa a GSRN e não depende dela.

Mesmo sendo diferente, SSH pode ter um objetivo parecido: permitir que alguém volte a acessar o computador depois. Por isso, um teste de SSH deve verificar quem pode usar cada chave, quem aprovou o acesso e se as tentativas de conexão ficam registradas. O time deve analisar essa técnica separadamente do GSocket para saber qual controle funcionou ou falhou.

| Forma | Quando pode agir | O que a equipe deve conferir |
| --- | --- | --- |
| Script de login | Quando alguém entra no sistema | Mudanças nos arquivos de perfil e qual usuário fez a alteração |
| Tarefa agendada | Em um horário ou evento definido | Mudanças na tarefa, conta usada e frequência de execução |
| Serviço systemd | Quando o sistema inicia ou o serviço é reiniciado | Serviço, programa iniciado, usuário e registros do systemd |
| Chave SSH | Quando alguém tenta se conectar por SSH | Chaves autorizadas, mudanças e registros de autenticação |

Esses recursos também são usados por administradores e programas legítimos. Por isso, uma mudança deve ser investigada antes de ser removida.

## 5. Como testar com segurança

O teste deve ser autorizado e limitado a computadores definidos previamente. Antes de começar, a equipe precisa combinar o período do teste, quem será avisado, quais mudanças são permitidas e como interromper a atividade se algo sair do plano.

**Primeiro, registrar como o computador está.** Anote os arquivos de login, tarefas, serviços, chaves SSH, programas em execução e conexões de rede que já existem. Isso ajuda a comparar o estado antes e depois.

**Depois, escolher uma coisa para testar.** Cada rodada deve avaliar um mecanismo por vez. Por exemplo, pode-se verificar se uma alteração controlada em um arquivo de login é percebida. Em outra rodada, testa-se o monitoramento de serviços systemd ou de chaves SSH. Assim, fica mais fácil descobrir onde está a falha.

**Usar uma atividade sem perigo.** O teste pode usar um programa simples e aprovado que apenas registre uma mensagem de identificação. Ele não deve abrir um terminal remoto, executar comandos recebidos nem acessar dados. Se a equipe também precisar testar a comunicação de rede, deve usar equipamentos e credenciais temporários sob seu controle.

**Acompanhar o que acontece.** Registre quando a mudança foi feita, quando o programa iniciou e se algum alerta apareceu. Também anote quem recebeu o alerta e o que essa pessoa fez. Isso mostra se os registros ajudam a entender a situação, e não apenas se existe uma notificação.

**Remover tudo ao terminar.** Apague o programa e as mudanças de teste, cancele os segredos temporários e confira se o computador voltou ao estado inicial. A equipe responsável deve confirmar a limpeza com registros.

## 6. Como avaliar o resultado

O teste pode ser avaliado com perguntas simples:

- **Registro:** o sistema guardou informações sobre a mudança, a execução e a conexão?
- **Identificação:** foi possível descobrir qual usuário e programa estavam envolvidos?
- **Alerta:** a equipe recebeu um aviso? Quanto tempo levou?
- **Investigação:** os analistas conseguiram juntar os registros e entender a ordem dos acontecimentos?
- **Resposta:** a equipe seguiu o plano sem causar impacto desnecessário?
- **Recuperação:** todos os arquivos e acessos temporários foram removidos?

Um único alerta não prova que houve um ataque. Uma conexão criptografada ou um serviço desconhecido também pode ter uma explicação legítima. A conclusão deve levar em conta várias evidências juntas, como quem fez a mudança, qual programa iniciou e que conexão aconteceu depois.

## 7. O que as organizações podem fazer

As organizações podem manter uma lista dos scripts de login, tarefas, serviços e chaves SSH que são esperados. Mudanças fora do processo normal devem ser registradas e investigadas. Também é importante guardar informações sobre quem iniciou cada programa, qual era o programa anterior e quando a conexão de rede aconteceu.

No caso do SSH, cada chave deve ter um responsável conhecido e ser retirada quando não for mais necessária. Serviços e tarefas devem ser comparados com uma lista aprovada. Os registros devem ser enviados para um local central, para que não dependam apenas do computador que está sendo investigado.

Se houver suspeita de acesso indevido, a equipe deve guardar as evidências antes de remover arquivos. Em seguida, pode limitar a comunicação do computador, cancelar chaves ou segredos expostos, investigar como o acesso começou e procurar outras mudanças. Se não for possível confiar no estado do sistema, restaurá-lo a partir de uma cópia segura pode ser melhor do que remover arquivos isolados.

## 8. Limites e cuidados

Este artigo não apresenta comandos ou arquivos de configuração para instalar persistência ou criar acesso remoto. Os testes descritos devem acontecer apenas em computadores autorizados e com um plano de limpeza. Usar um relay externo ou mudar recursos de inicialização exige permissão clara, credenciais temporárias e limites de tempo.

O comportamento pode mudar conforme a versão do programa, a configuração do computador e as regras da rede. Por isso, o relatório do teste deve registrar essas condições. Também não se deve concluir que uma conexão é maliciosa apenas porque passou por um relay ou estava criptografada.

## 9. Conclusão

O GSocket pode ajudar dois computadores a se comunicarem por meio de um relay, mas não faz um programa iniciar sozinho depois que o computador reinicia. Para isso, seria necessário algum recurso do Linux, como um script de login, uma tarefa agendada ou um serviço systemd. O acesso por SSH é outra possibilidade e precisa ser analisado separadamente.

Para um time de Red Team, o principal é testar se a organização consegue perceber as mudanças, descobrir qual programa iniciou a conexão e responder corretamente. Um bom teste não precisa criar um acesso remoto funcional: pode usar uma ação segura e ainda mostrar se os controles de segurança estão funcionando.

## Referências

1. Infiltr8, [GSocket for Persistence](https://infiltr8.io/redbook/redteam/persistence/linux/gsocket-for-persistence/). Fonte sobre GSocket, GSRN e os mecanismos de inicialização citados.
2. Infiltr8, [SSH for Persistence](https://infiltr8.io/redbook/redteam/persistence/linux/ssh/). Fonte complementar sobre riscos relacionados ao acesso por SSH.
3. Projeto GSocket, [repositório oficial](https://github.com/hackerschoice/gsocket).
4. MITRE ATT&CK, [Persistence (TA0003)](https://attack.mitre.org/tactics/TA0003/), [Cron (T1053.003)](https://attack.mitre.org/techniques/T1053/003/), [Systemd Service (T1543.002)](https://attack.mitre.org/techniques/T1543/002/) e [SSH Authorized Keys (T1098.004)](https://attack.mitre.org/techniques/T1098/004/).
